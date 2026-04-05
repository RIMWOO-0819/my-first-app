# 프로젝트 핸드오프 문서

작성일: 2026-04-05

---

## 프로젝트 개요

`index.html` 단일 파일로 동작하는 바닐라 HTML/CSS/JS 계산기 웹앱.
빌드 도구, 서버, 설치 과정 없음. 브라우저에서 파일을 직접 열어 실행.

---

## 현재 구현 상태

### 완료된 기능

| 기능 | 상태 |
|------|------|
| 사칙연산 (+, -, ×, ÷) | ✅ 완료 |
| 부호 전환 (+/-), 퍼센트 (%) | ✅ 완료 |
| 연속 연산자 자동 중간 계산 | ✅ 완료 |
| 0 나누기 오류 처리 ("오류" 표시) | ✅ 완료 |
| 키보드 단축키 (숫자/연산자/Enter/Esc/Backspace) | ✅ 완료 |
| 계산 히스토리 (최근 5개, 최신순) | ✅ 완료 |
| 히스토리 항목 클릭으로 결과값 불러오기 | ✅ 완료 |
| 히스토리 전체 지우기 버튼 | ✅ 완료 |
| 히스토리 최신 항목 파란 테두리 강조 | ✅ 완료 |
| 로그인 / 회원가입 (localStorage 기반) | ✅ 완료 |
| 비밀번호 SHA-256 해시 저장 (Web Crypto API) | ✅ 완료 |
| 로그인 탭 ↔ 회원가입 탭 전환 | ✅ 완료 |
| 페이지 새로고침 후 세션 복원 | ✅ 완료 |
| 사용자별 히스토리 분리 저장 | ✅ 완료 |
| 사용자별 테마 설정 분리 저장 | ✅ 완료 |
| 로그아웃 시 상태 완전 초기화 | ✅ 완료 |
| 테마 프리셋 7종 (라이트/다크/체리블라썸/오션/포레스트/선셋/미드나이트) | ✅ 완료 |
| 커스텀 배경색 / 어센트색 피커 (실시간 반영) | ✅ 완료 |
| 설정 패널 우측 슬라이드인 애니메이션 | ✅ 완료 |
| iOS 스타일 둥근 버튼 + 클릭 피드백 | ✅ 완료 |
| Web Audio API 클릭 효과음 ("뾱") | ✅ 완료 |
| 글꼴 변환 (5종 프리셋, 사용자별 저장) | ✅ 완료 |

---

## 파일 구조

```
my-first-app/
├── index.html      ← 모든 코드 (CSS + HTML + JS) 인라인 포함 (~1220줄)
├── plan-spec.md    ← 기획 스펙 문서
├── CLAUDE.md       ← Claude Code 지시 파일
└── HANDOFF.md      ← 이 문서
```

---

## 코드 구조 (index.html 내부)

### CSS (7~587줄)
- CSS 변수로 테마 전환 (`--bg-page`, `--bg-btn-op` 등)
- 라이트 테마가 `:root` 기본값으로 정의됨
- 주요 컴포넌트: `.calculator`, `.history-panel`, `.auth-overlay`, `.settings-panel`

### HTML (588~680줄)
- 인증 오버레이 → 로그인/회원가입 카드
- 상단 사용자 바 (로그아웃 버튼 포함)
- 히스토리 패널 + 계산기 본체
- 설정 패널 (우측 슬라이드인)

### JS (681~1221줄)

#### 인증 & 사용자 관리 (681~830줄)
- `hashPassword(pw)` — Web Crypto API SHA-256
- `submitAuth()` — 로그인/회원가입 처리 (async)
- `loginSuccess(user)` — 로그인 후 UI 전환 및 데이터 복원
- `logout()` — 세션 제거, 상태·테마 초기화
- `loadUserData(userId)` — 히스토리·테마 복원
- `init()` — 앱 시작 시 세션 복원

#### 테마 시스템 (832~1006줄)
- `PRESETS` 객체 — 7종 테마 CSS 변수 정의
- `applyTheme(presetId, customBg, customAccent)` — CSS 변수 일괄 적용
- `selectPreset(id)` — 프리셋 선택 및 저장
- `applyCustomColors(bg, accent)` — 색상 피커 실시간 반영
- `renderPresetGrid()` — 설정 패널 프리셋 그리드 렌더링

#### 계산기 로직 (1008~1188줄)
- 상태: `current`, `previous`, `operator`, `shouldReset`, `calcHistory`
- `inputNum(num)` / `inputDot()` — 숫자 입력
- `setOperator(op)` — 연산자 선택 (미완료 연산 자동 처리)
- `calculate(silent)` — 연산 수행 (`silent=true`면 히스토리 미저장)
- `clearAll()` / `toggleSign()` / `percent()` — 유틸리티
- `addHistory` / `renderHistory` / `clearHistory` — 히스토리 관리
- `keydown` 이벤트리스너 — 키보드 단축키 통합 처리

#### 글꼴 시스템 (~1010~1070줄)
- `FONTS` 객체 — 5종 글꼴 정의 (family, Google Fonts 키)
- `loadGoogleFont(fontKey)` — Google Fonts `<link>` 동적 삽입 (중복 방지)
- `applyFont(fontId)` — body.fontFamily 즉시 변경, 버튼 active 갱신
- `selectFont(fontId)` — 글꼴 선택 및 localStorage 저장
- `renderFontGrid()` — 설정 패널 글꼴 버튼 그리드 렌더링

#### 효과음 (~1240~1270줄)
- `playClickSound()` — Web Audio API 오실레이터 합성
- `.buttons` mousedown 이벤트 위임으로 모든 버튼에 적용

---

## localStorage 구조

| 키 | 값 형식 | 설명 |
|----|---------|------|
| `calc_users` | `[{id, username, pwHash}]` | 전체 사용자 목록 |
| `calc_currentUser` | `{id, username}` | 현재 로그인 세션 |
| `calc_histories` | `{userId: [{expr, result}]}` | 사용자별 히스토리 |
| `calc_themes` | `{userId: {preset, customBg, customAccent, font}}` | 사용자별 테마·글꼴 |

---

## 미구현 / 알려진 개선 가능 항목

- `=` 연속 입력 시 마지막 연산 반복
- 천 단위 구분 표시 (1,000,000)
- 모바일 반응형 레이아웃 (현재 고정 폭 320px)

---

## 실행 방법

```
index.html 파일을 브라우저에서 직접 열기
```

별도 설치, 빌드, 서버 불필요.
