# 계산기 웹앱 — 기획 스펙 문서

## 프로젝트 개요

빌드 도구 없이 `index.html` 단일 파일로 동작하는 바닐라 HTML/CSS/JS 계산기 웹앱.
브라우저에서 파일을 직접 열어 실행하며, 별도 서버·설치·빌드 과정이 없다.

---

## 구현된 기능 목록

### 1. 기본 계산기
- 사칙연산 (+, -, ×, ÷)
- 부호 전환 (+/-), 퍼센트 (%)
- 연속 연산자 입력 시 중간 계산 자동 처리
- 0 나누기 오류 처리
- 키보드 단축키 지원 (숫자, 연산자, Enter, Escape, Backspace)

### 2. 계산 히스토리
- 최근 5개 결과 자동 저장 (최신순)
- 히스토리 항목 클릭 시 해당 결과값 불러오기
- 히스토리 전체 지우기 버튼
- 가장 최근 항목 파란 테두리 강조 표시

### 3. 사용자 로그인 / 회원가입
- localStorage 기반 (서버 없음)
- 비밀번호 SHA-256 해시 저장 (Web Crypto API, 평문 저장 금지)
- 로그인/회원가입 탭 전환
- 로그인 유지 (페이지 새로고침 후에도 세션 복원)
- 사용자별 히스토리·테마 설정 분리 저장
- 로그아웃 시 상태 완전 초기화

### 4. 테마 커스터마이징
- ⚙️ 설정 버튼 → 우측 슬라이드인 패널
- 프리셋 7종:

  | 아이콘 | 이름 | 계열 |
  |--------|------|------|
  | ☀️ | 라이트 | 흰/회색 |
  | 🌙 | 다크 | 네이비 |
  | 🌸 | 체리블라썸 | 분홍 |
  | 🌊 | 오션 | 하늘/청록 |
  | 🌿 | 포레스트 | 연초록 |
  | 🌅 | 선셋 | 따뜻한 노랑 |
  | 💜 | 미드나이트 | 딥퍼플 |

- 커스텀 색상 피커 2개: 배경색 / 어센트색 (실시간 반영)
- 테마 설정은 사용자별로 localStorage에 저장·복원

### 5. iOS 스타일 버튼 디자인
- 둥근 모서리 (`border-radius: 50px`)
- 연산자·등호 버튼 주황색 (`#FF9F0A`)
- 클릭 시 어두워지는 효과 (`filter: brightness(0.72)`)
- Web Audio API 기반 "뾱" 효과음 (외부 파일 없음)

---

## 기술 스택

| 항목 | 내용 |
|------|------|
| 언어 | HTML / CSS / JavaScript (바닐라) |
| 빌드 | 없음 |
| 저장소 | localStorage |
| 암호화 | Web Crypto API (SHA-256) |
| 사운드 | Web Audio API (오실레이터 합성) |
| 테마 | CSS 변수 (`--var`) JS 동적 적용 |

---

## localStorage 구조

| 키 | 형식 | 설명 |
|----|------|------|
| `calc_users` | `[{id, username, pwHash}]` | 전체 사용자 목록 |
| `calc_currentUser` | `{id, username}` | 현재 로그인 세션 |
| `calc_histories` | `{userId: [{expr, result}]}` | 사용자별 히스토리 |
| `calc_themes` | `{userId: {preset, customBg, customAccent}}` | 사용자별 테마 |

---

## JS 주요 상태 변수

| 변수 | 설명 |
|------|------|
| `current` | 현재 디스플레이 숫자 문자열 |
| `previous` | 첫 번째 피연산자 |
| `operator` | 선택된 연산자 (`+` `-` `*` `/`) |
| `shouldReset` | 다음 입력 시 current 덮어쓸지 여부 |
| `calcHistory` | 히스토리 배열 (최대 5개) |
| `currentUser` | 로그인한 사용자 `{id, username}` |
| `currentPreset` | 현재 적용 중인 테마 ID |
| `currentCustomBg` | 커스텀 배경색 (null이면 프리셋 값 사용) |
| `currentCustomAccent` | 커스텀 어센트색 |

---

## JS 주요 함수

| 함수 | 역할 |
|------|------|
| `calculate(silent)` | 연산 수행. silent=true면 히스토리 미저장 |
| `setOperator(op)` | 연산자 선택, 미완료 연산 있으면 먼저 처리 |
| `addHistory` / `renderHistory` | 히스토리 배열 관리 및 DOM 렌더링 |
| `submitAuth()` | 로그인/회원가입 처리 |
| `loginSuccess(user)` | 로그인 후 UI 전환 및 데이터 복원 |
| `logout()` | 세션 제거, 상태·테마 초기화 |
| `applyTheme(presetId, bg, accent)` | CSS 변수 일괄 적용 |
| `selectPreset(id)` | 프리셋 선택 및 저장 |
| `applyCustomColors(bg, accent)` | 색상 피커 실시간 반영 |
| `openSettings` / `closeSettings` | 설정 패널 열기/닫기 |
| `playClickSound()` | Web Audio API 효과음 재생 |
| `init()` | 앱 시작 시 세션 복원 또는 로그인 화면 표시 |

---

## 파일 구조

```
my-first-app/
└── index.html     ← 모든 코드 (CSS + HTML + JS) 인라인 포함
```

---

## 알려진 개선 가능 항목 (미구현)

- `=` 연속 입력 시 마지막 연산 반복
- 숫자 최대 길이 제한
- 천 단위 구분 표시 (1,000,000)
- 모바일 반응형 레이아웃
- 히스토리 innerHTML XSS 방어 (textContent로 교체)
