# 글꼴 변환 — 기획 스펙

## 목적
계산기 전체 글꼴을 사용자가 선택할 수 있도록 하여 개인화 경험을 강화한다.

## 제공 글꼴 5종

| ID | 이름 | 분위기 | Google Fonts |
|----|------|--------|--------------|
| `default` | 기본 | 시스템 폰트 (현재) | 없음 |
| `roboto-mono` | Roboto Mono | 모노스페이스, 개발자 감성 | ✅ |
| `nunito` | Nunito | 둥글고 부드러운 느낌 | ✅ |
| `orbitron` | Orbitron | 디지털/미래적 | ✅ |
| `noto-sans-kr` | Noto Sans KR | 한국어 지원 깔끔한 산세리프 | ✅ |

## 동작 명세

- 설정 패널(⚙️)에 글꼴 선택 섹션 추가 (테마 섹션 하단)
- 글꼴 버튼 5개 나열, 현재 선택된 글꼴에 강조 표시
- 글꼴 선택 시 → `<body>` 또는 `.calculator` 전체에 font-family 즉시 적용
- 로그인 상태면 사용자별 localStorage에 저장
- 로그인 후 해당 사용자의 저장된 글꼴 복원
- 로그아웃 시 기본 글꼴로 초기화
- Google Fonts는 `<link>` 태그로 `<head>`에 동적 추가 (중복 방지)

## UI 변경 사항

- `settings-panel` 내부 하단에 `.font-section` 추가
- `.font-btn` 클래스: 글꼴 이름을 해당 글꼴로 표시하는 버튼
- `.font-btn.active`: 현재 선택 강조 (테마 프리셋과 동일한 스타일)

## 데이터 구조

### 새 상태 변수
- `currentFont` — 현재 적용 중인 글꼴 ID (기본값: `'default'`)

### localStorage 변경
- `calc_themes` 구조 확장:
  ```
  {userId: {preset, customBg, customAccent, font}}
  ```
  기존 키에 `font` 필드 추가 (하위 호환: 없으면 `'default'` 사용)

## 영향받는 기존 코드

| 함수 | 위치 | 변경 내용 |
|------|------|----------|
| `applyTheme()` | ~832줄 | font 파라미터 추가 또는 별도 `applyFont()` 함수 신설 |
| `selectPreset()` | ~880줄 | 글꼴 저장 로직 포함하도록 확장 |
| `loadUserData()` | ~780줄 | font 복원 로직 추가 |
| `logout()` | ~760줄 | 글꼴 초기화 추가 |
| `loginSuccess()` | ~730줄 | 글꼴 복원 호출 추가 |
| `renderPresetGrid()` | ~950줄 | 건드리지 않음 |

## 작업 분해

- [ ] A. UI — 설정 패널에 글꼴 섹션 HTML/CSS 추가
- [ ] B. 로직 — `applyFont()`, `selectFont()` 함수 구현 + Google Fonts 동적 로드
- [ ] C. 연동 — loadUserData / logout / loginSuccess에 글꼴 저장·복원 연결
