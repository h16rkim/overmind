---
name: playwright
description: Playwright MCP를 사용하여 봇 차단(User-Agent 검사·접속 차단)이 적용된 웹사이트도 크롤링한다. 네이버 스마트스토어·쿠팡·11번가처럼 기본 헤드리스 브라우저가 차단되는 사이트의 상품·검색·랭킹 데이터를 수집할 때 사용한다.
---

# Playwright 봇 차단 우회 크롤링

`@executeautomation/playwright-mcp-server` MCP 도구를 사용해 봇 차단이 걸린 사이트를 헤드리스로 크롤링한다. 이 스킬은 다음 두 가지 흔한 실패를 해결한다.

1. **브라우저 바이너리 버전 불일치** — MCP 서버가 기대하는 Chromium 버전이 로컬에 없을 때 발생하는 `Executable doesn't exist` 오류.
2. **User-Agent 기반 차단** — Playwright 기본 UA(HeadlessChrome 등)로 접근 시 네이버 같은 사이트가 "현재 서비스 접속이 불가합니다" 에러 페이지를 반환하는 문제.

## 사전 준비: 도구 로드

Playwright MCP 도구는 deferred tool이므로 호출 전 `ToolSearch`로 스키마를 로드해야 한다. 일반적으로 다음을 함께 로드한다.

```
ToolSearch query:
  select:mcp__plugin_mcps_playwright__playwright_navigate,
         mcp__plugin_mcps_playwright__playwright_custom_user_agent,
         mcp__plugin_mcps_playwright__playwright_evaluate,
         mcp__plugin_mcps_playwright__playwright_get_visible_text,
         mcp__plugin_mcps_playwright__playwright_close
```

필요에 따라 `playwright_get_visible_html`, `playwright_screenshot`, `playwright_click`, `playwright_fill` 등도 같이 로드한다.

## 표준 크롤링 절차

### Step 1. User-Agent를 실제 브라우저 값으로 교체

**`navigate` 호출보다 먼저** `playwright_custom_user_agent`를 호출해야 한다. 페이지가 이미 로드된 뒤에 UA를 바꾸면 차단된 응답이 그대로 남아있다.

```
mcp__plugin_mcps_playwright__playwright_custom_user_agent
  userAgent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/131.0.0.0 Safari/537.36"
```

OS에 맞춰 macOS·Windows·Linux UA를 선택한다. 차단이 매우 엄격한 사이트는 최신 Chrome 정식 빌드 UA가 가장 잘 통한다.

### Step 2. 페이지 이동

```
mcp__plugin_mcps_playwright__playwright_navigate
  url: "<대상 URL>"
  headless: true
  width: 1440
  height: 900
  waitUntil: "networkidle"   # SPA·지연 로딩이 많은 사이트에 유리
```

`waitUntil: "networkidle"`은 초기 데이터가 비동기 요청으로 채워지는 사이트(스마트스토어 등)에서 필수에 가깝다. 정적 페이지면 `"domcontentloaded"`만 줘도 충분하다.

### Step 3. 차단 여부 검증

페이지가 정상인지 먼저 확인한다. **본문이 비어있거나 상품 카드가 0개면 봇 차단을 의심**한다.

```
mcp__plugin_mcps_playwright__playwright_evaluate
  script:
    JSON.stringify({
      title: document.title,
      url: location.href,
      bodyStart: document.body.innerText.substring(0, 300),
      linkCount: document.querySelectorAll('a').length
    })
```

`document.title`이 "에러" / "Access Denied" / "Forbidden" 등을 포함하거나 `bodyStart`에 "접속이 불가합니다", "비정상적인 접근" 같은 문구가 보이면 차단된 것이다. 해결 방법은 [트러블슈팅](#트러블슈팅) 참고.

### Step 4. 데이터 추출

`playwright_evaluate`로 DOM을 직접 쿼리해 구조화된 JSON을 만든다. 일반 `get_visible_text`는 노이즈가 많으므로 셀렉터 기반 추출이 더 안정적이다.

**리스트형 페이지 추출 패턴**

```javascript
const links = Array.from(document.querySelectorAll('a[href*="/products/"]'));
const map = new Map();
for (const a of links) {
  const m = a.href.match(/\/products\/(\d+)/);
  if (!m) continue;
  const id = m[1];
  if (map.has(id)) continue;
  // 상품 카드 컨테이너 찾기: li > article > class 매칭
  const container =
    a.closest('li') ||
    a.closest('article') ||
    a.closest('[class*="item"]') ||
    a.closest('[class*="card"]') ||
    a.parentElement?.parentElement;
  const text = container
    ? container.innerText.trim().replace(/\s+/g, ' ').substring(0, 300)
    : '';
  map.set(id, { id, href: a.href.split('?')[0], text });
}
JSON.stringify(Array.from(map.values()).slice(0, 10), null, 2)
```

핵심 아이디어:
- `closest()`로 카드 단위 컨테이너를 잡으면 한 상품의 정보(제목·가격·평점·리뷰 수 등)가 한 덩어리로 묶인다.
- `Map`으로 상품 ID를 키로 사용해 중복을 제거한다(같은 카드 내부에 링크가 여러 개 있는 경우 흔함).
- 결과를 `JSON.stringify`로 반환하면 호출자가 파싱하기 편하다.

### Step 5. 정리

```
mcp__plugin_mcps_playwright__playwright_close
```

브라우저를 닫지 않으면 다음 세션에서 이전 UA 설정이 남거나 리소스가 누수된다.

## 트러블슈팅

### A. `Executable doesn't exist at .../chromium-XXXX/...`

MCP 서버가 특정 Playwright 빌드 버전(예: `1200`)의 Chromium을 요구하는데 로컬엔 다른 버전만 설치돼 있을 때 발생한다.

**해결: 정확히 그 버전을 다운로드하는 Playwright npm 버전을 찾아 설치**

```bash
mkdir -p /tmp/pw-install && cd /tmp/pw-install
npm init -y >/dev/null
# 메시지에 나온 build vNNNN과 매칭되는 playwright npm 버전을 시도
npm install playwright@1.57 --no-save
npx playwright install chromium
```

빌드 번호 → npm 버전 매핑은 출력 메시지의 `playwright build vNNNN`으로 확인한다(예: 1200 → playwright 1.57, 1194 → 1.56, 1193 → 1.55, 1148 → 1.49). 한 번에 맞지 않으면 인접 버전을 1~2번 더 시도한다.

**하지 말 것**: `chromium-1223 → chromium-1200` 같은 심볼릭 링크로 버전 검증을 우회하지 말 것. 안전 검증을 회피하는 동작으로 거부될 수 있고, ABI가 깨질 위험도 있다.

### B. 네이버 류 차단: "현재 서비스 접속이 불가합니다"

증상: `document.title`이 `[에러] 에러페이지 - 시스템오류`, 본문에 "동시에 접속하는 이용자 수가 많거나..." 안내.

원인은 거의 항상 **UA가 HeadlessChrome으로 노출**되어 있어서다. Step 1을 누락했거나, `playwright_close` 없이 다른 사이트로 이동해 UA 설정이 풀린 경우다.

해결:
1. `playwright_close`로 브라우저 종료
2. `playwright_custom_user_agent`로 데스크톱 Chrome UA 다시 설정
3. `playwright_navigate`로 재진입

그래도 차단되면:
- `width`/`height`를 모바일이 아닌 데스크톱 해상도(1440x900 이상)로 둔다.
- `waitUntil: "networkidle"`로 변경한다.
- Referer가 필요한 사이트는 검색 결과 페이지를 거쳐서 들어간다.

### C. 데이터가 SPA로 늦게 채워짐

`evaluate`가 빈 배열을 반환하면 데이터가 아직 렌더되지 않았을 가능성이 크다.

```
mcp__plugin_mcps_playwright__playwright_evaluate
  script:
    new Promise(resolve => setTimeout(() => resolve('waited'), 3000))
```

3초 정도 대기 후 다시 `evaluate`로 추출한다. 그래도 비면 `getElementsByClassName` 대신 `querySelectorAll`로 더 넓은 셀렉터를 사용하고, `document.body.innerText`를 일부 잘라 출력해 실제 DOM에 어떤 텍스트가 있는지 확인한다.

### D. 무한 스크롤·페이지네이션

리스트가 스크롤로 더 로드되는 경우:

```javascript
window.scrollTo(0, document.body.scrollHeight);
```

를 `evaluate`로 실행하고, 위 C의 대기 패턴으로 1~2초 기다린 뒤 다시 추출한다. 페이지네이션이면 URL의 페이지 파라미터(`?page=2`)를 바꿔 `navigate`를 다시 호출하는 쪽이 안정적이다.

## 호출 가능한 주요 도구 목록

| 용도 | 도구 |
|---|---|
| UA 설정 | `mcp__plugin_mcps_playwright__playwright_custom_user_agent` |
| 페이지 이동 | `mcp__plugin_mcps_playwright__playwright_navigate` |
| JS 실행/데이터 추출 | `mcp__plugin_mcps_playwright__playwright_evaluate` |
| 본문 텍스트 | `mcp__plugin_mcps_playwright__playwright_get_visible_text` |
| HTML 덤프 | `mcp__plugin_mcps_playwright__playwright_get_visible_html` |
| 스크린샷 | `mcp__plugin_mcps_playwright__playwright_screenshot` |
| 입력/클릭 | `mcp__plugin_mcps_playwright__playwright_fill`, `playwright_click` |
| 종료 | `mcp__plugin_mcps_playwright__playwright_close` |

