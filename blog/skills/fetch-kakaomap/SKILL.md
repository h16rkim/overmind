---
name: fetch-kakaomap
description: 카카오맵에서 원하는 장소의 정보를 추출
---

# 카카오맵 장소 조사 (Kakao Map Place Research)

Claude for Chrome MCP 도구를 활용하여 카카오맵에서 장소 정보를 조사합니다.


## Instructions

**중요: 모든 응답은 한국어로 작성합니다.**

당신은 카카오맵을 활용하여 장소 정보를 조사하는 전문가입니다. Claude for Chrome MCP 도구를 사용하여 카카오맵에서 장소를 검색하고 상세 정보를 수집합니다.

## 사전 준비

1. **브라우저 탭 확인**
   ```
   mcp__claude-in-chrome__tabs_context_mcp 도구로 현재 탭 상태 확인
   탭이 없으면 mcp__claude-in-chrome__tabs_create_mcp로 새 탭 생성
   ```

2. **카카오맵 접속**
   ```
   mcp__claude-in-chrome__navigate 도구로 https://map.kakao.com 접속
   ```

## 조사 프로세스

### Step 1: 장소 검색

1. **검색창 찾기**
   ```
   mcp__claude-in-chrome__find 도구 사용
   query: "search input" 또는 "검색"
   ```

2. **검색어 입력**
   ```
   mcp__claude-in-chrome__form_input 도구 사용
   ref: [검색창 ref_id]
   value: "[지역명] [장소명]" (예: "통영 타베루")
   ```

3. **검색 실행**
   ```
   mcp__claude-in-chrome__computer 도구 사용
   action: "key"
   text: "Enter"
   ```

4. **검색 결과 대기**
   ```
   mcp__claude-in-chrome__computer 도구 사용
   action: "wait"
   duration: 2
   ```

### Step 2: 검색 결과에서 장소 선택

1. **검색 결과 확인**
   ```
   mcp__claude-in-chrome__read_page 도구로 검색 결과 목록 확인
   또는
   mcp__claude-in-chrome__computer 도구로 스크린샷 촬영
   action: "screenshot"
   ```

2. **원하는 장소 클릭**
   ```
   mcp__claude-in-chrome__find 도구로 장소명 찾기
   query: "[장소명]"

   또는 좌표로 클릭
   mcp__claude-in-chrome__computer 도구 사용
   action: "left_click"
   coordinate: [x, y]
   ```

3. **상세 페이지 로딩 대기**
   ```
   mcp__claude-in-chrome__computer 도구 사용
   action: "wait"
   duration: 2
   ```

### Step 3: 상세 정보 수집

1. **URL에서 Place ID 추출**
   ```
   현재 URL 확인하여 Place ID 추출
   형식: https://place.map.kakao.com/[PLACE_ID]
   ```

2. **상세 페이지 내용 읽기**
   ```
   mcp__claude-in-chrome__get_page_text 도구 사용
   또는
   mcp__claude-in-chrome__read_page 도구 사용
   ```

3. **수집할 정보**
   | 항목 | 설명 | 위치 |
   |------|------|------|
   | 상세 링크 | `https://place.map.kakao.com/[PLACE_ID]` | URL |
   | 평점 | ⭐ 별점 (5점 만점) | 장소명 근처 |
   | 리뷰 수 | 리뷰 개수 | 평점 옆 |
   | 영업시간 | 요일별 영업시간 | 기본정보 섹션 |
   | 휴무일 | 정기 휴무일 | 영업시간 내 또는 별도 표시 |
   | 주소 | 정확한 주소 | 기본정보 섹션 |
   | 전화번호 | 연락처 | 기본정보 섹션 |
   | 대표 메뉴/가격 | 식당의 경우 | 메뉴 섹션 |
   | 위도/경도 | 좌표 정보 | 상세 페이지 텍스트 또는 공유 링크 |

4. **위도/경도 추출**

   카카오맵 상세 페이지에서 좌표를 추출하는 방법입니다. 방법 1(OG 메타태그)이 가장 정확하므로 우선적으로 사용합니다.

   **방법 1 (권장): OG 메타태그의 staticmap URL에서 추출**

   카카오맵 장소 페이지 HTML의 `<meta>` 태그에 좌표가 포함되어 있습니다.
   ```
   Bash 도구로 curl을 사용하여 추출:
   curl -sL "https://place.map.kakao.com/[PLACE_ID]" | grep -o 'staticmap[^"]*'
   ```
   결과 예시: `staticmap/og?type=place&srs=wgs84&...&m=128.4234567%2C34.8567890`
   - `m=` 파라미터의 값이 `경도(longitude)%2C위도(latitude)` 순서
   - `%2C`는 쉼표(`,`)의 URL 인코딩

   여러 장소를 일괄 처리할 때는 Python 스크립트로 반복 호출:
   ```python
   import subprocess, re
   place_id = "133408066"
   r = subprocess.run(["curl", "-sL", f"https://place.map.kakao.com/{place_id}"],
                       capture_output=True, text=True, timeout=10)
   match = re.search(r'staticmap.*?m=([\d.]+)%2C([\d.]+)', r.stdout)
   if match:
       longitude, latitude = float(match.group(1)), float(match.group(2))
   ```

   **방법 2: 페이지 텍스트에서 좌표 찾기**
   ```
   mcp__claude-in-chrome__get_page_text 도구로 페이지 텍스트를 읽고,
   "127." 또는 "128." 또는 "126." 으로 시작하는 좌표 형태의 텍스트를 찾습니다.

   한국의 좌표 범위:
   - 경도(longitude): 124.0 ~ 132.0 (주로 126.XXX ~ 129.XXX)
   - 위도(latitude): 33.0 ~ 39.0 (주로 33.XXX ~ 38.XXX)

   예시 텍스트: "128.4234567, 34.8567890" 또는 "128.4234567 34.8567890"
   → longitude: 128.4234567, latitude: 34.8567890
   ```

   **방법 3: JavaScript로 좌표 추출**
   ```
   mcp__claude-in-chrome__javascript_tool 도구 사용
   script:
     const text = document.body.innerText;
     const match = text.match(/(12[4-9]\.\d+|13[0-2]\.\d+)[,\s]+(3[3-9]\.\d+)/);
     match ? `longitude: ${match[1]}, latitude: ${match[2]}` : 'not found';
   ```

   **주의사항:**
   - 일부 장소 페이지(SPA 전용)에서는 OG 메타태그에 staticmap이 포함되지 않을 수 있음 → 방법 2, 3으로 대체
   - `12X.XXX`로 시작하는 숫자가 경도(longitude), `3X.XXX`로 시작하는 숫자가 위도(latitude)
   - 좌표 순서가 "경도, 위도" 또는 "위도, 경도"일 수 있으므로 값의 범위로 구분
   - 좌표를 찾지 못한 경우 null로 처리

### Step 4: "더보기" 클릭 (필요시)

영업시간이나 메뉴 정보가 접혀있는 경우:

```
mcp__claude-in-chrome__find 도구로 "더보기" 또는 "펼치기" 버튼 찾기
query: "더보기" 또는 "영업시간"

mcp__claude-in-chrome__computer 도구로 클릭
action: "left_click"
ref: [버튼 ref_id]
```

### Step 5: 다음 장소 검색

1. **검색창으로 돌아가기**
   ```
   mcp__claude-in-chrome__navigate 도구로 https://map.kakao.com 재접속
   또는
   검색창 찾아서 새 검색어 입력
   ```

2. **Step 1~4 반복**

## 출력 형식

### 단일 장소 결과
```markdown
## [장소명] 조사 결과

| 항목 | 내용 |
|------|------|
| 카카오맵 링크 | https://place.map.kakao.com/[PLACE_ID] |
| 평점 | ⭐ [평점] ([리뷰 수]개 리뷰) |
| 주소 | [주소] |
| 영업시간 | [영업시간] |
| 휴무일 | ⚠️ [휴무일] |
| 대표메뉴 | [메뉴명] ([가격]) |
| 위도 | [latitude] (예: 34.8567890) |
| 경도 | [longitude] (예: 128.4234567) |
| 특징 | [특징/설명] |
```

### 여러 장소 결과 테이블
```markdown
| 장소명 | 평점 | 리뷰 | 주소 | 영업시간 | 휴무일 | 위도 | 경도 | 링크 |
|--------|------|------|------|----------|--------|------|------|------|
| [**장소1**](링크) | 4.5 | 30 | 주소1 | 10:00~22:00 | ⚠️ 월요일 | 34.856 | 128.423 | [링크](url) |
| [**장소2**](링크) | 4.2 | 50 | 주소2 | 09:00~18:00 | 연중무휴 | 34.857 | 128.424 | [링크](url) |
```

### 마크다운 형식 규칙
- **링크**: `[**장소명**](https://place.map.kakao.com/[PLACE_ID])`
- **아이콘**:
  - 🎁 지인 추천
  - ⭐ 숙소/Airbnb 추천
  - ⚠️ 휴무일 주의
  - 📍 위치 정보
- **휴무일 강조**: `⚠️ **[요일] 휴무**`

## 트러블슈팅

### 검색 결과가 없는 경우
1. 검색어 변경 시도 (예: "통영 타베루" → "타베루 통영")
2. 장소 유형 추가 (예: "타베루 음식점")
3. 정확한 주소로 검색

### 상세 페이지 로딩 실패
1. `wait` 시간 늘리기 (2초 → 3초)
2. 페이지 새로고침 후 재시도
3. 직접 URL 접속: `https://map.kakao.com`에서 재검색

### 정보를 찾을 수 없는 경우
1. 스크롤하여 추가 정보 확인
   ```
   mcp__claude-in-chrome__computer
   action: "scroll"
   scroll_direction: "down"
   coordinate: [화면 중앙 좌표]
   ```
2. 탭 전환 (홈, 메뉴, 사진, 리뷰 등)
3. "더보기" 버튼 클릭

### Place ID를 찾을 수 없는 경우
1. URL 직접 확인
2. 검색 결과 목록에서 "상세보기" 또는 장소명 클릭
3. 지도에서 마커 클릭 후 정보창의 "자세히 보기" 클릭

## 예시 세션

```
## 조사 결과

| 장소명 | 평점 | 리뷰 | 영업시간 | 휴무일 |
|--------|------|------|----------|--------|
| [**타베루**](https://place.map.kakao.com/1799732285) 🎁 | 4.5 | 30 | 12:00~22:00 | ⚠️ **화요일** |
| [**전혁림미술관**](https://place.map.kakao.com/10719736) 🎁 | 4.5 | - | 수~일 10:00~17:00 | ⚠️ **월,화** |
| [**레몬샵**](https://place.map.kakao.com/2098618398) 🎁 | 5.0 | 1 | ⚠️ **토,일만** 11:00~17:00 | 월~금 |

### 휴무일 분석
- 타베루: 화요일 휴무
- 전혁림미술관: 월,화 휴무 → Day 1(금)~Day 3(일) 방문 가능
- 레몬샵: 주말만 영업 → Day 2(토), Day 3(일)만 방문 가능
```

더 많은 결과물 예시로는, 다음의 파일을 참고하세요 [examples.md](examples.md).


## 고급 기능

### 여러 장소 병렬 조사
여러 장소를 조사할 때는 한 번에 하나씩 순차적으로 진행합니다.
각 장소 조사 완료 후 결과를 누적하여 최종 테이블로 정리합니다.


