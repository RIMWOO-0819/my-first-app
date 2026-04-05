# /release-check — 배포 전 점검

이 계산기 웹앱(`index.html`)의 배포 준비 상태를 점검한다.
`plan-spec.md`와 `HANDOFF.md`를 참조해 현재 구현 상태와 스펙을 대조한다.

---

## 진행 방식

점검을 세 개의 병렬 트랙으로 나눠 동시에 수행한다.

**Agent A — 기능 동작 검증**
**Agent B — UI 검증**
**Agent C — 데이터 / 보안 검증**

각 트랙을 병렬로 실행하고, 완료 후 결과를 통합해 최종 보고서를 작성한다.

---

## Agent A: 기능 동작 검증

`index.html`의 JS 로직(`681~1280줄` 범위)을 읽고 아래 항목을 코드 레벨에서 점검한다.

### 체크리스트

#### 기본 계산기
- [ ] 사칙연산 (+, -, ×, ÷) 각각 구현 여부 (`calculate()`)
- [ ] 0 나누기 시 "오류" 처리 구현 여부
- [ ] 연속 연산자 입력 시 중간 계산 자동 처리 (`setOperator()`)
- [ ] `+/-`, `%` 버튼 구현 여부 (`toggleSign()`, `percent()`)
- [ ] 키보드 단축키 이벤트리스너 구현 여부 (`keydown`)
- [ ] Backspace 처리 구현 여부

#### 히스토리
- [ ] 최근 5개 제한 로직 구현 여부 (`addHistory()`)
- [ ] 히스토리 항목 클릭 시 결과값 불러오기 구현 여부
- [ ] 히스토리 전체 지우기 구현 여부 (`clearHistory()`)
- [ ] 히스토리 localStorage 저장/복원 구현 여부

#### 인증
- [ ] 로그인/회원가입 분기 처리 구현 여부 (`submitAuth()`)
- [ ] 비밀번호 SHA-256 해시 처리 구현 여부 (`hashPassword()`)
- [ ] 페이지 새로고침 후 세션 복원 구현 여부 (`init()`)
- [ ] 로그아웃 시 상태 완전 초기화 구현 여부 (`logout()`)

#### 테마 / 글꼴
- [ ] 테마 프리셋 7종 정의 여부 (`PRESETS`)
- [ ] 커스텀 색상 피커 구현 여부 (`applyCustomColors()`)
- [ ] 글꼴 변환 5종 정의 여부 (`FONTS`)
- [ ] 글꼴 Google Fonts 동적 로드 구현 여부 (`loadGoogleFont()`)
- [ ] 테마·글꼴 사용자별 저장/복원 구현 여부

**미구현 항목 확인**: `HANDOFF.md`의 "미구현/알려진 개선 가능 항목" 목록 중 현재도 미구현인 항목을 코드에서 확인해 그대로 보고한다.

---

## Agent B: UI 검증

`index.html`의 CSS(`7~620줄`)와 HTML(`620~710줄`)을 읽고 아래 항목을 점검한다.

### 체크리스트

#### 레이아웃
- [ ] 계산기 고정 폭(`320px`) 설정 여부
- [ ] 히스토리 패널 폭(`220px`) 설정 여부
- [ ] 모바일 반응형 CSS(`@media`) 존재 여부 — 없으면 "미구현" 으로 명시
- [ ] `body` flex 중앙 정렬 구현 여부

#### 버튼 스타일
- [ ] iOS 스타일 `border-radius: 50px` 적용 여부
- [ ] 연산자/등호 버튼 주황색(`#FF9F0A`) 적용 여부
- [ ] 클릭 시 `filter: brightness(0.72)` 피드백 구현 여부
- [ ] `0` 버튼 두 칸(`.btn-zero`) 처리 여부

#### 설정 패널
- [ ] 우측 슬라이드인 애니메이션 CSS 구현 여부
- [ ] 오버레이 배경 클릭 시 닫히는 구조 HTML 확인
- [ ] 테마 프리셋 그리드 렌더링 요소 존재 여부 (`#presetGrid`)
- [ ] 글꼴 그리드 요소 존재 여부 (`#fontGrid`)
- [ ] 커스텀 색상 피커 2개 존재 여부 (`#colorBg`, `#colorAccent`)

#### 인증 오버레이
- [ ] 로그인/회원가입 탭 전환 구조 존재 여부
- [ ] 에러 메시지 표시 요소 존재 여부 (`#authError`)

#### CSS 변수
- [ ] `:root`에 테마 CSS 변수 전체 정의 여부
- [ ] `transition` 속성으로 테마 전환 애니메이션 여부

### 데스크톱 / 모바일 점검 안내

브라우저 개발자 도구에서 직접 확인할 항목:

```
데스크톱 (1280px 이상):
- 히스토리 패널과 계산기가 나란히 표시되는지
- 설정 패널이 우측에서 슬라이드인되는지
- 계산기 카드 그림자가 정상 표시되는지

모바일 (375px — iPhone SE 기준):
- 계산기가 화면을 벗어나지 않는지
- 버튼이 터치하기 충분한 크기인지 (최소 44px)
- 히스토리 패널 처리 방식 확인 (현재 고정 폭이라 모바일에서 깨질 수 있음)
```

---

## Agent C: 데이터 / 보안 검증

`index.html`의 JS를 읽고 아래 항목을 점검한다.

### localStorage 구조 점검

코드에서 실제로 사용하는 localStorage 키와 스펙을 대조한다.

| 키 | 스펙 형식 | 코드 내 사용 함수 | 일치 여부 |
|----|----------|-----------------|----------|
| `calc_users` | `[{id, username, pwHash}]` | `getUsers()` / `saveUsers()` | 점검 |
| `calc_currentUser` | `{id, username}` | `loginSuccess()` / `logout()` | 점검 |
| `calc_histories` | `{userId: [{expr, result}]}` | `getHistories()` / `saveHistories()` | 점검 |
| `calc_themes` | `{userId: {preset, customBg, customAccent, font}}` | `getThemes()` / `saveThemes()` | 점검 |

**하위 호환 처리 확인**: `loadUserData()`에서 기존 문자열 형식 테마 데이터 처리 코드 존재 여부.

브라우저에서 직접 확인하는 방법:
```
개발자 도구 → Application → Local Storage → 파일 경로 선택
→ calc_users, calc_currentUser, calc_histories, calc_themes 키 확인
```

### 보안 체크

#### XSS 취약점
- [ ] `renderHistory()` 내 `innerHTML` 사용 여부 확인
  - 사용 중이면: 취약점 존재 — `textContent`로 교체 권고
  - 히스토리 데이터는 사용자가 직접 입력하지 않으므로 실제 위험도는 낮으나, 방어적 코딩 권장
- [ ] `renderPresetGrid()` 내 `innerHTML` 사용 여부 확인
  - 프리셋은 코드 내 상수이므로 안전
- [ ] `renderFontGrid()` 내 `innerHTML` 사용 여부 확인

#### 인증 보안
- [ ] 비밀번호 평문 저장 여부 — `pwHash`만 저장하는지 확인
- [ ] `hashPassword()`가 `crypto.subtle.digest('SHA-256', ...)`를 사용하는지 확인
- [ ] 입력값 trim 처리 여부 (`username.trim()`)
- [ ] 비밀번호 최소 길이 검증(4자) 구현 여부

#### 입력 검증
- [ ] 빈 아이디/비밀번호 제출 방지 구현 여부
- [ ] 중복 아이디 회원가입 방지 구현 여부
- [ ] 숫자 입력 최대 길이 제한 여부 — **현재 미구현**으로 명시

---

## 최종 보고서 형식

세 Agent의 결과를 통합해 아래 형식으로 출력한다.

```
# 배포 전 점검 결과 — [날짜]

## 요약
- 전체 체크 항목: N개
- 통과: N개 / 실패: N개 / 미구현(알려진): N개

## 기능 동작 검증 결과 (Agent A)
[통과/실패 목록]

## UI 검증 결과 (Agent B)
[통과/실패 목록]
[브라우저 직접 확인 필요 항목]

## 데이터/보안 검증 결과 (Agent C)
[통과/실패 목록]

## 배포 전 수정 우선순위

### 🔴 즉시 수정 (배포 불가)
- [실제 버그, 크래시, 보안 취약점]

### 🟡 권고 수정 (배포 가능하나 개선 필요)
- [UX 문제, 방어적 코딩 누락]

### 🟢 알려진 미구현 (향후 개선)
- HANDOFF.md의 미구현 항목 그대로 나열

## 콘솔 에러 확인 방법
브라우저에서 index.html을 열고:
1. F12 → Console 탭 → 빨간 에러 메시지 확인
2. F12 → Console 탭에서 직접 실행:
   - localStorage 전체 확인: `JSON.stringify(localStorage, null, 2)`
   - 현재 사용자 확인: `localStorage.getItem('calc_currentUser')`
   - 테마 설정 확인: `localStorage.getItem('calc_themes')`
3. Network 탭 → Google Fonts 로드 성공 여부 확인 (글꼴 기능 사용 시)
```
