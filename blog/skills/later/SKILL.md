---
name: later
description: 블로그의 Later API를 사용하여 나중에 할 일을 추가하거나 검색합니다
---

# Later 관리 (할 일 추가/검색/완료)

블로그 Later API를 호출하여 나중에 할 일을 추가하거나 검색, 완료 처리합니다.

## Instructions

**모든 응답은 한국어로 작성합니다.**

당신은 사용자와 대화하며 나중에 할 일 정보를 수집하고, 블로그 Later API를 통해 저장하거나 검색하는 역할을 합니다.

### 환경 설정

- **API Base URL**: `https://h16rk.im`
- **인증 헤더**: `X-Blog-Api-Key` — 값은 환경변수 `BLOG_API_KEY`에서 가져옴
- **API 호출 도구**: `WebFetch` 도구 사용

### LaterType (할 일 타입)

| 값 | 설명 |
|-----|------|
| `work` | 업무 관련 |
| `chore` | 잡일/집안일 |
| `cart` | 구매할 것 |
| `read` | 읽을 것 |
| `etc` | 기타 |

## 할 일 검색 프로세스

사용자가 할 일을 검색하고 싶어할 때:

1. 검색 조건 파악 (키워드, 타입, archive 여부 등)
2. API 호출:
   ```
   WebFetch 도구 사용
   url: https://h16rk.im/client/api/v1/laters?search={검색어}&type={타입}&archive={true|false}&page={페이지}
   method: GET
   headers: { "X-Blog-Api-Key": "${BLOG_API_KEY}" }
   ```
3. 결과를 테이블 형태로 정리하여 사용자에게 보여줌

### 검색 결과 출력 형식

```markdown
| 제목 | 타입 | 마감일 | 완료 |
|------|------|--------|------|
| 할일1 | work | 20260410 1430 | - |
| 할일2 | read | - | 20260405 |
```

## 할 일 추가 프로세스

사용자가 할 일을 추가하고 싶어할 때:

### Step 1: 정보 수집

사용자에게 다음 정보를 대화로 수집합니다:

| 필드 | 필수 | 설명 | 예시 |
|------|------|------|------|
| title | O | 할 일 제목 | "블로그 리팩토링" |
| type | X | 할 일 타입 (work, chore, cart, read, etc) | "work" |
| body | X | 상세 내용 (마크다운) | "## 목표\n- API 정리\n- 테스트 추가" |
| deadline | X | 마감일 (ISO 형식) | "2026-04-10T14:30" |
| link | X | 관련 URL | "https://..." |

**대화 예시:**
- 사용자: "블로그 리팩토링 추가해 줘"
- → title: "블로그 리팩토링"이 파악됨. type을 물어봄
- 에이전트: "타입은 어떤 것으로 할까요? (work/chore/cart/read/etc) 마감일이 있나요?"
- 사용자: "work이고, 다음 주 금요일까지"
- → type: "work", deadline 계산

**최소한의 질문으로 정보를 수집합니다.** 사용자의 메시지에서 유추 가능한 정보는 직접 채웁니다. title만 필수이며, 나머지는 없으면 null로 보냅니다.

### Step 1.5: 링크 내용 요약 (link가 있는 경우)

사용자가 link를 제공한 경우, 저장 전에 해당 링크의 내용을 읽어 요약합니다.

#### 링크 유형별 도구 선택

링크 URL 패턴을 분석하여 가장 적합한 도구를 사용합니다:

| 링크 패턴 | 사용 도구 | 비고 |
|-----------|----------|------|
| `*.slack.com/*` | `mcp__claude_ai_Slack__slack_read_channel` 또는 `mcp__claude_ai_Slack__slack_read_thread` | 채널/스레드 URL 파싱하여 적절한 도구 선택 |
| `*.atlassian.net/wiki/*` | `mcp__claude_ai_Atlassian__getConfluencePage` | Confluence 페이지 ID 추출하여 조회 |
| `*.atlassian.net/browse/*` 또는 `*.atlassian.net/jira/*` | `mcp__claude_ai_Atlassian__getJiraIssue` | Jira 이슈 키 추출하여 조회 |
| GitHub issue URL | `mcp__plugin_mcps_fetch__fetch_github_issue` | GitHub 이슈 내용 조회 |
| GitHub PR URL | `mcp__plugin_mcps_fetch__fetch_github_pull_request` | GitHub PR 내용 조회 |
| 그 외 일반 웹 URL | `mcp__plugin_mcps_fetch__fetch` 또는 `WebFetch` | 일반 웹페이지 내용 조회 |

#### 요약 프로세스

1. 위 표를 참고하여 링크 유형에 맞는 도구로 내용을 가져옴
2. 가져온 내용을 **3~5문장으로 요약**
3. 요약 내용을 body 필드에 포함시킴
   - 기존 body가 있는 경우: 기존 body 끝에 `\n\n---\n\n**링크 요약:**\n{요약 내용}` 형태로 추가
   - 기존 body가 없는 경우: `**링크 요약:**\n{요약 내용}` 을 body로 설정
4. title이 아직 없는 경우, 링크 내용을 바탕으로 적절한 title을 제안

**주의사항:**
- 요약은 핵심 내용 위주로 간결하게 작성
- **fetch에 실패하거나 내용을 읽을 수 없는 경우, 요약을 포함시키지 않고 그대로 later를 생성** (사용자에게 별도 확인 불필요)

### Step 2: 확인 및 저장

수집한 정보를 사용자에게 보여주고 확인을 받습니다:

```
다음 할 일을 추가할까요?
- 제목: 블로그 리팩토링
- 타입: work
- 마감일: 2026-04-10 14:30
```

사용자가 확인하면 API 호출:

```
WebFetch 도구 사용
url: https://h16rk.im/client/api/v1/laters
method: POST
headers: {
  "Content-Type": "application/json",
  "X-Blog-Api-Key": "${BLOG_API_KEY}"
}
body: {
  "title": "블로그 리팩토링",
  "type": "work",
  "body": null,
  "deadline": "2026-04-10T14:30",
  "link": null
}
```

### Step 3: 결과 전달

API 응답 성공 시:
```
할 일이 추가되었습니다!
- 제목: 블로그 리팩토링
- ID: 1
- 등록일: 2026-04-06
```

## 할 일 완료 프로세스

사용자가 할 일을 완료 처리하고 싶어할 때:

1. 완료할 항목 파악 (ID 또는 제목으로 검색)
2. API 호출:
   ```
   WebFetch 도구 사용
   url: https://h16rk.im/client/api/v1/laters/{id}/complete
   method: PATCH
   headers: { "X-Blog-Api-Key": "${BLOG_API_KEY}" }
   ```
3. 결과 전달

## 할 일 수정 프로세스

기존에 등록된 할 일의 정보를 수정할 때:

1. GET으로 기존 데이터 조회
2. 변경할 필드만 수정하여 PUT으로 전송

```
WebFetch 도구 사용
url: https://h16rk.im/client/api/v1/laters/{id}
method: PUT
headers: {
  "Content-Type": "application/json",
  "X-Blog-Api-Key": "${BLOG_API_KEY}"
}
body: {
  "title": "...",
  "type": "...",
  "body": "...",
  "deadline": "2026-04-10T14:30",
  "link": "..."
}
```

**주의:** PUT 요청 시 모든 필드를 포함해야 합니다. 먼저 GET으로 기존 데이터를 조회한 후, 변경할 필드만 수정하여 전송합니다.

## API 레퍼런스

### 할 일 목록 조회
```
GET https://h16rk.im/client/api/v1/laters
Query: page (default: 1), size (default: 20), type (optional), search (optional), archive (default: false)
Header: X-Blog-Api-Key: ${BLOG_API_KEY}

Response: { "success": true, "statusCode": 200, "data": { "laters": [...], "page": 1, "hasNext": false, "hasPrev": false } }
```

### 할 일 상세 조회
```
GET https://h16rk.im/client/api/v1/laters/{id}
Header: X-Blog-Api-Key: ${BLOG_API_KEY}

Response: { "success": true, "statusCode": 200, "data": { "id": 1, "title": "...", "type": "work", "body": "...", "deadline": "...", "completedAt": null, ... } }
```

### 할 일 추가
```
POST https://h16rk.im/client/api/v1/laters
Header: Content-Type: application/json, X-Blog-Api-Key: ${BLOG_API_KEY}
Body: { "title": "...", "type": "work", "body": "...", "deadline": "2026-04-10T14:30", "link": "..." }

Response: { "success": true, "statusCode": 200, "data": { "id": 1, ... } }
```

### 할 일 수정
```
PUT https://h16rk.im/client/api/v1/laters/{id}
Header: Content-Type: application/json, X-Blog-Api-Key: ${BLOG_API_KEY}
Body: { "title": "...", "type": "...", "body": "...", "deadline": "...", "link": "..." }

Response: { "success": true, "statusCode": 200, "data": { "id": 1, ... } }
```

### 할 일 완료
```
PATCH https://h16rk.im/client/api/v1/laters/{id}/complete
Header: X-Blog-Api-Key: ${BLOG_API_KEY}

Response: { "success": true, "statusCode": 200, "data": { "id": 1, "completedAt": "...", ... } }
```

### 할 일 삭제
```
DELETE https://h16rk.im/client/api/v1/laters/{id}
Header: X-Blog-Api-Key: ${BLOG_API_KEY}

Response: { "success": true, "statusCode": 200, "data": null }
```

### 타입 목록 조회
```
GET https://h16rk.im/client/api/v1/laters/types
Header: X-Blog-Api-Key: ${BLOG_API_KEY}

Response: { "success": true, "statusCode": 200, "data": [{ "value": "work", "name": "WORK" }, ...] }
```

## 트러블슈팅

### API 호출 실패 (401/403)
- `BLOG_API_KEY` 환경변수가 설정되어 있는지 확인
- 헤더 이름이 정확히 `X-Blog-Api-Key`인지 확인

### 서버 연결 실패
- 블로그 서버(`https://h16rk.im`)가 실행 중인지 확인
- 로컬 개발 시에는 `http://localhost:8080`으로 변경
