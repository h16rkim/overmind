---
name: fetch-googlemap
description: 구글맵에서 원하는 장소의 정보를 추출
---

# 구글맵 장소 조사 (Google Maps Place Research)

Claude for Chrome MCP 도구를 활용하여 구글맵에서 장소 정보를 조사합니다.


## Instructions

**중요: 모든 응답은 한국어로 작성합니다.**

당신은 구글맵을 활용하여 장소 정보를 조사하는 전문가입니다. Claude for Chrome MCP 도구를 사용하여 구글맵에서 장소를 검색하고 상세 정보를 수집합니다.

## 사전 준비

1. **브라우저 탭 확인**
   ```
   mcp__claude-in-chrome__tabs_context_mcp 도구로 현재 탭 상태 확인
   탭이 없으면 mcp__claude-in-chrome__tabs_create_mcp로 새 탭 생성
   ```

2. **구글맵 접속**
   ```
   mcp__claude-in-chrome__navigate 도구로 https://www.google.com/maps 접속
   ```

## 조사 프로세스

### Step 1: 장소 검색

1. **검색창 찾기**
   ```
   mcp__claude-in-chrome__find 도구 사용
   query: "search" 또는 "검색"
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
   duration: 3
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

3. **상세 패널 로딩 대기**
   ```
   mcp__claude-in-chrome__computer 도구 사용
   action: "wait"
   duration: 3
   ```

### Step 3: 상세 정보 수집

1. **URL에서 Place 정보 추출**
   ```
   현재 URL 확인
   형식: https://www.google.com/maps/place/[장소명]/@[lat],[lng],[zoom]z/data=...
   ```

2. **상세 패널 내용 읽기**
   ```
   mcp__claude-in-chrome__get_page_text 도구 사용
   또는
   mcp__claude-in-chrome__read_page 도구 사용
   ```

3. **수집할 정보**
   | 항목 | 설명 | 위치 |
   |------|------|------|
   | 구글맵 링크 | 현재 페이지 URL | URL 바 |
   | 평점 | ⭐ 별점 (5점 만점) | 장소명 아래 |
   | 리뷰 수 | 리뷰 개수 | 평점 옆 괄호 안 |
   | 영업시간 | 요일별 영업시간 | 상세 정보 섹션 |
   | 휴무일 | 정기 휴무일 | 영업시간 내 |
   | 주소 | 정확한 주소 | 상세 정보 섹션 |
   | 전화번호 | 연락처 | 상세 정보 섹션 |
   | 카테고리 | 업종/장소 유형 | 평점 위 또는 장소명 아래 |
   | 웹사이트 | 공식 웹사이트 URL | 상세 정보 섹션 |
   | 위도/경도 | 좌표 정보 | URL에서 추출 |

4. **위도/경도 추출 (URL에서)**

   구글맵 장소 상세 페이지의 URL에는 위도/경도가 포함되어 있는 경우가 많습니다.

   **URL 패턴:**
   ```
   https://www.google.com/maps/place/[장소명]/@[latitude],[longitude],[zoom]z/data=...
   ```

   **예시:**
   ```
   https://www.google.com/maps/place/타베루/@34.8567890,128.4234567,17z/data=...
   → latitude: 34.8567890, longitude: 128.4234567
   ```

   **방법 1: URL 직접 확인**
   ```
   mcp__claude-in-chrome__javascript_tool 도구 사용
   script:
     const url = window.location.href;
     const match = url.match(/@(-?\d+\.\d+),(-?\d+\.\d+)/);
     match ? `latitude: ${match[1]}, longitude: ${match[2]}` : 'not found';
   ```

   **방법 2: 단축 URL(`maps.app.goo.gl`)에서 좌표 추출**

   구글맵 공유 링크는 `https://maps.app.goo.gl/xxx` 형태의 단축 URL인 경우가 많습니다.
   단축 URL을 리다이렉트하면 `@lat,lng` 패턴이 포함된 전체 URL을 얻을 수 있습니다.
   ```
   Bash 도구로 curl 리다이렉트 헤더 확인:
   curl -sI -L "https://maps.app.goo.gl/xxx" | grep -i "location"
   ```
   Location 헤더에서 `@lat,lng` 패턴을 추출합니다.

   일괄 처리 시 Python 스크립트 사용:
   ```python
   import subprocess, re
   url = "https://maps.app.goo.gl/xxx"
   r = subprocess.run(["curl", "-sI", "-L", url], capture_output=True, text=True, timeout=15)
   locations = re.findall(r'[Ll]ocation:\s*(.*)', r.stdout)
   final_url = locations[-1].strip() if locations else ""
   match = re.search(r'@(-?\d+\.\d+),(-?\d+\.\d+)', final_url)
   if match:
       latitude, longitude = float(match.group(1)), float(match.group(2))
   ```

   **방법 3: URL에 좌표가 없는 경우**
   - 지도에서 장소 마커를 우클릭하면 좌표가 표시되는 경우가 있음
   - 또는 페이지 텍스트에서 좌표 형태의 숫자를 검색

   **주의사항:**
   - 구글맵 URL의 `@` 뒤 숫자가 `latitude, longitude` 순서
   - 한국의 좌표 범위: 위도 33~39, 경도 124~132
   - 대만의 좌표 범위: 위도 22~26, 경도 120~122
   - URL에 좌표가 없는 경우 null로 처리

### Step 4: "더보기" 클릭 (필요시)

영업시간이나 상세 정보가 접혀있는 경우:

```
mcp__claude-in-chrome__find 도구로 "더보기" 또는 영업시간 드롭다운 찾기
query: "영업시간" 또는 "hours"

mcp__claude-in-chrome__computer 도구로 클릭
action: "left_click"
ref: [버튼 ref_id]
```

### Step 5: 다음 장소 검색

1. **검색창으로 돌아가기**
   ```
   mcp__claude-in-chrome__navigate 도구로 https://www.google.com/maps 재접속
   또는
   검색창 찾아서 기존 텍스트 지우고 새 검색어 입력
   ```

2. **Step 1~4 반복**

## 출력 형식

### 단일 장소 결과
```markdown
## [장소명] 조사 결과

| 항목 | 내용 |
|------|------|
| 구글맵 링크 | [URL] |
| 평점 | ⭐ [평점] ([리뷰 수]개 리뷰) |
| 카테고리 | [카테고리] |
| 주소 | [주소] |
| 영업시간 | [영업시간] |
| 휴무일 | ⚠️ [휴무일] |
| 전화번호 | [전화번호] |
| 웹사이트 | [URL] |
| 위도 | [latitude] (예: 34.8567890) |
| 경도 | [longitude] (예: 128.4234567) |
| 특징 | [특징/설명] |
```

### 여러 장소 결과 테이블
```markdown
| 장소명 | 평점 | 리뷰 | 주소 | 영업시간 | 휴무일 | 위도 | 경도 | 링크 |
|--------|------|------|------|----------|--------|------|------|------|
| [**장소1**](링크) | 4.5 | 30 | 주소1 | 10:00~22:00 | ⚠️ 월요일 | 34.856 | 128.423 | [구글맵](url) |
| [**장소2**](링크) | 4.2 | 50 | 주소2 | 09:00~18:00 | 연중무휴 | 34.857 | 128.424 | [구글맵](url) |
```

### 마크다운 형식 규칙
- **링크**: `[**장소명**](구글맵 URL)`
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
3. 영어로 검색 시도 (예: "Taberu Tongyeong")
4. 정확한 주소로 검색

### 상세 패널이 열리지 않는 경우
1. `wait` 시간 늘리기 (3초 → 5초)
2. 검색 결과 목록에서 장소명을 직접 클릭
3. 페이지 새로고침 후 재시도

### 정보를 찾을 수 없는 경우
1. 상세 패널을 스크롤하여 추가 정보 확인
   ```
   mcp__claude-in-chrome__computer
   action: "scroll"
   scroll_direction: "down"
   coordinate: [패널 중앙 좌표]
   ```
2. "리뷰", "정보", "사진" 등 탭 전환
3. "더보기" 버튼 클릭

### 구글맵 언어가 영어인 경우
- 검색어는 한국어로 입력해도 결과가 잘 나옴
- 영업시간 등 정보는 영어로 표시될 수 있으나, 결과는 한국어로 정리

## 예시 세션

```
## 조사 결과

| 장소명 | 평점 | 리뷰 | 영업시간 | 휴무일 |
|--------|------|------|----------|--------|
| [**타베루**](https://www.google.com/maps/place/...) 🎁 | 4.5 | 120 | 12:00~22:00 | ⚠️ **화요일** |
| [**전혁림미술관**](https://www.google.com/maps/place/...) 🎁 | 4.3 | 85 | 수~일 10:00~17:00 | ⚠️ **월,화** |

### 휴무일 분석
- 타베루: 화요일 휴무
- 전혁림미술관: 월,화 휴무 → Day 1(금)~Day 3(일) 방문 가능
```

## 고급 기능

### 여러 장소 병렬 조사
여러 장소를 조사할 때는 한 번에 하나씩 순차적으로 진행합니다.
각 장소 조사 완료 후 결과를 누적하여 최종 테이블로 정리합니다.

### 카카오맵과 비교 조사
구글맵과 카카오맵의 평점/리뷰 수가 다를 수 있습니다.
필요시 `fetch-kakaomap` 스킬과 함께 사용하여 두 플랫폼의 정보를 비교할 수 있습니다.
