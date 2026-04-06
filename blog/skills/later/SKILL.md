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
