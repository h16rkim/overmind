---
name: place
description: 블로그의 Place API를 사용하여 가보고 싶은 장소를 추가하거나 검색합니다
---

# Place 관리 (장소 추가/검색)

블로그 Place API를 호출하여 가보고 싶은 장소를 추가하거나 검색합니다.

## Instructions

**모든 응답은 한국어로 작성합니다.**

당신은 사용자와 대화하며 가보고 싶은 장소 정보를 수집하고, 블로그 Place API를 통해 장소를 저장하거나 검색하는 역할을 합니다.

### 환경 설정

- **API Base URL**: `https://h16rk.im`
- **인증 헤더**: `X-Blog-Api-Key` — 값은 환경변수 `BLOG_API_KEY`에서 가져옴
- **API 호출 도구**: `WebFetch` 도구 사용

### PlaceType (장소 타입)

| 값 | 설명 |
|-----|------|
| `cafe` | 카페 |
| `restaurent` | 레스토랑 |
| `bar` | 바/술집 |
| `culture` | 문화생활 (미술관, 박물관, 영화관 등) |
| `etc` | 기타 |

## 장소 검색 프로세스

사용자가 장소를 검색하고 싶어할 때:

1. 검색 조건 파악 (키워드, 타입 등)
2. API 호출:
   ```
   WebFetch 도구 사용
   url: https://h16rk.im/client/api/v1/places?search={검색어}&type={타입}&page={페이지}
   method: GET
   headers: { "X-Blog-Api-Key": "${BLOG_API_KEY}" }
   ```
3. 결과를 테이블 형태로 정리하여 사용자에게 보여줌

### 검색 결과 출력 형식

```markdown
| 이름 | 타입 | 주소 | 등록일 |
|------|------|------|--------|
| 장소명1 | cafe | 서울시 ... | 20260405 |
| 장소명2 | restaurent | 부산시 ... | 20260404 |
```

## 장소 추가 프로세스

사용자가 장소를 추가하고 싶어할 때:

### Step 1: 정보 수집

사용자에게 다음 정보를 대화로 수집합니다:

| 필드 | 필수 | 설명 | 예시 |
|------|------|------|------|
| name | O | 장소 이름 | "블루보틀 성수" |
| type | X | 장소 타입 (cafe, restaurent) | "cafe" |
| content | O | 장소에 대한 설명/메모 | "커피가 맛있는 곳" |
| address | X | 주소 | "서울시 성동구 ..." |
| link | X | 관련 URL | "https://..." |
| memo | X | 추가 메모 | "주말에 줄이 길다" |
| latitude | X | 위도 (33.0~39.0) | 37.5443 |
| longitude | X | 경도 (124.0~132.0) | 127.0557 |

**대화 예시:**
- 사용자: "블루보틀 성수 추가해 줘"
- → name: "블루보틀 성수"가 파악됨. content가 없으므로 물어봄
- 에이전트: "블루보틀 성수에 대한 설명을 알려주세요. 타입은 cafe/restaurent 중 어떤 것인가요?"
- 사용자: "카페야. 커피가 진짜 맛있는 곳이야"
- → type: "cafe", content: "커피가 진짜 맛있는 곳" 파악 완료

**최소한의 질문으로 정보를 수집합니다.** 사용자의 메시지에서 유추 가능한 정보는 직접 채웁니다. name과 content만 필수이며, 나머지는 없으면 null로 보냅니다.

### Step 2: 확인 및 저장

수집한 정보를 사용자에게 보여주고 확인을 받습니다:

```
다음 장소를 추가할까요?
- 이름: 블루보틀 성수
- 타입: cafe
- 설명: 커피가 진짜 맛있는 곳
```

사용자가 확인하면 API 호출:

```
WebFetch 도구 사용
url: https://h16rk.im/client/api/v1/places
method: POST
headers: {
  "Content-Type": "application/json",
  "X-Blog-Api-Key": "${BLOG_API_KEY}"
}
body: {
  "name": "블루보틀 성수",
  "type": "cafe",
  "content": "커피가 진짜 맛있는 곳",
  "address": null,
  "link": null,
  "memo": null,
  "latitude": null,
  "longitude": null
}
```

### Step 3: 결과 전달

API 응답 성공 시:
```
장소가 추가되었습니다!
- 이름: 블루보틀 성수
- ID: 1
- 등록일: 2026-04-05
```

## API 레퍼런스

### 장소 목록 조회
```
GET https://h16rk.im/client/api/v1/places
Query: page (default: 1), size (default: 20), type (optional), search (optional)
Header: X-Blog-Api-Key: ${BLOG_API_KEY}

Response: { "success": true, "statusCode": 200, "data": { "places": [...], "page": 1, "hasNext": false, "hasPrev": false } }
```

### 장소 상세 조회
```
GET https://h16rk.im/client/api/v1/places/{id}
Header: X-Blog-Api-Key: ${BLOG_API_KEY}

Response: { "success": true, "statusCode": 200, "data": { "id": 1, "name": "...", "type": "cafe", "content": "...", ... } }
```

### 장소 추가
```
POST https://h16rk.im/client/api/v1/places
Header: Content-Type: application/json, X-Blog-Api-Key: ${BLOG_API_KEY}
Body: { "name": "...", "type": "cafe|restaurent", "content": "...", "address": "...", "link": "...", "memo": "...", "latitude": 37.5443, "longitude": 127.0557 }

Response: { "success": true, "statusCode": 200, "data": { "id": 1, ... } }
```

## 장소 수정 프로세스

기존에 등록된 장소의 정보를 수정할 때:

```
Bash 도구 사용
curl -s -X PUT "https://h16rk.im/client/api/v1/places/{id}" \
  -H 'Content-Type: application/json' \
  -H "X-Blog-Api-Key: ${BLOG_API_KEY}" \
  -d '{ "name": "...", "type": "...", "content": "...", "address": "...", "link": "...", "memo": "...", "latitude": 37.5443, "longitude": 127.0557 }'
```

**주의:** PUT 요청 시 모든 필드를 포함해야 합니다. 먼저 GET으로 기존 데이터를 조회한 후, 변경할 필드만 수정하여 전송합니다.

**위도/경도 포함:** 카카오맵/구글맵 조사(`fetch-kakaomap`, `fetch-googlemap` 스킬)에서 위도/경도를 파악한 경우, 장소 추가/수정 시 `latitude`와 `longitude` 필드에 반드시 포함합니다. 값을 모르는 경우 null로 보냅니다.

## 블로그 게시글에서 장소 일괄 추가

사용자가 블로그 게시글 URL을 제공하며 장소를 추가해달라고 할 때:

### Step 1: 게시글 읽기
- `mcp__claude-in-chrome__navigate`로 게시글 URL 접속
- `mcp__claude-in-chrome__get_page_text`로 본문 내용 읽기
- 음식점, 카페, 서점, 문화공간 등 장소 목록 추출

### Step 2: 카카오맵 링크 추출
- `mcp__claude-in-chrome__javascript_tool`로 게시글 내 카카오맵 링크 수집:
  ```javascript
  const links = document.querySelectorAll('a[href*="place.map.kakao.com"]');
  const map = {};
  links.forEach(link => {
    const name = link.textContent.trim();
    if (name && !map[name]) {
      map[name] = link.href.match(/\/(\d+)$/)?.[1] || '';
    }
  });
  Object.entries(map).map(([k,v]) => k + ':' + v).join('\n');
  ```

### Step 3: Bash로 일괄 등록
- 장소가 많을 경우, `Bash` 도구에서 `curl` 반복문으로 일괄 API 호출
- 카카오맵 링크가 있으면 `link` 필드에 `https://place.map.kakao.com/{placeId}` 형식으로 포함
- 장소 타입 분류: 음식점→`restaurent`, 카페/서점→`cafe`, 미술관/영화관/박물관 등→`culture`, 공원/산책로 등→`etc`

### Step 4: 링크 업데이트 (기존 장소)
- 이미 등록된 장소에 카카오맵 링크를 추가하려면:
  1. GET으로 기존 데이터 조회
  2. link 필드만 변경하여 PUT으로 업데이트
  ```bash
  update_link() {
    local id=$1 link=$2
    local data=$(curl -s "$BASE/$id" -H "X-Blog-Api-Key: $API_KEY")
    local name=$(echo "$data" | jq -r '.data.name')
    local type=$(echo "$data" | jq -r '.data.type')
    local content=$(echo "$data" | jq -r '.data.content')
    local address=$(echo "$data" | jq -r '.data.address')
    local memo=$(echo "$data" | jq -r '.data.memo')
    local lat=$(echo "$data" | jq '.data.latitude')
    local lng=$(echo "$data" | jq '.data.longitude')
    curl -s -X PUT "$BASE/$id" \
      -H 'Content-Type: application/json' \
      -H "X-Blog-Api-Key: $API_KEY" \
      -d "$(jq -n --arg name "$name" --arg type "$type" --arg content "$content" --arg address "$address" --arg link "$link" --arg memo "$memo" --argjson lat "$lat" --argjson lng "$lng" '{name:$name,type:$type,content:$content,address:$address,link:$link,memo:$memo,latitude:$lat,longitude:$lng}')"
  }
  ```

## API 레퍼런스 (추가)

### 장소 수정
```
PUT https://h16rk.im/client/api/v1/places/{id}
Header: Content-Type: application/json, X-Blog-Api-Key: ${BLOG_API_KEY}
Body: { "name": "...", "type": "...", "content": "...", "address": "...", "link": "...", "memo": "...", "latitude": 37.5443, "longitude": 127.0557 }

Response: { "success": true, "statusCode": 200, "data": { "id": 1, ... } }
```

## 트러블슈팅

### API 호출 실패 (401/403)
- `BLOG_API_KEY` 환경변수가 설정되어 있는지 확인
- 헤더 이름이 정확히 `X-Blog-Api-Key`인지 확인

### 서버 연결 실패
- 블로그 서버(`https://h16rk.im`)가 실행 중인지 확인
- 로컬 개발 시에는 `http://localhost:8080`으로 변경
