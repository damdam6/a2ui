# renderers/lit/a2ui_explorer/src/local-gallery.css.ts

## 개요

`LocalGallery` Lit 컴포넌트에서 사용하는 모든 스타일을 `CSSResult` 형태로 내보내는 모듈이다. Lit의 `css` 태그드 템플릿을 사용하여 작성되었으며, 컴포넌트의 레이아웃 구조(헤더, 네비게이션 패널, 갤러리 패널, 인스펙터 패널), 색상 테마, 인터랙션 상태를 모두 정의한다.

## 의존성

### 외부 패키지
- `lit` — `css` 태그드 템플릿 리터럴 함수

### 저장소 내부 모듈
없음.

## Exports

- `appStyles` (`CSSResult`) — `LocalGallery` 컴포넌트의 전체 스타일 정의

## 상세 명세

### `appStyles` 상수 (`CSSResult`)

`css` 태그드 템플릿으로 작성된 스타일 객체. Lit 컴포넌트의 `static styles`에 배열 요소로 포함되어 Shadow DOM 내부에 적용된다.

정의된 주요 CSS 규칙과 그 의미:

**`:host`** — 컴포넌트 루트. `flex`, `column`, `100vh × 100vw`, `overflow: hidden`. 배경색 `#0f172a`(짙은 남색), 텍스트 색 `#f1f5f9`, `font-family: system-ui, sans-serif`.

**`header`** — 상단 헤더. padding `16px 24px`, 반투명 배경 `rgba(15, 23, 42, 0.8)`, 하단 경계선 `rgba(148, 163, 184, 0.1)`, `flex` + `space-between` + `align-items: center`, `flex-shrink: 0`.

**`h1`** — margin 0, `font-size: 1.5rem`.

**`p.subtitle`** — 색상 `#94a3b8`, margin `4px 0 0 0`, `font-size: 0.9rem`.

**`main`** — `flex: 1`, `display: flex`, `overflow: hidden`.

**`.nav-pane`** — 좌측 네비게이션 패널. 너비 `250px`, 배경 `#1e293b`, 우측 경계선, `flex column`, `overflow-y: auto`.

**`.nav-item`** — 네비게이션 항목. padding `16px`, `cursor: pointer`, 하단 경계선, `transition: background 0.2s`. `:hover` 시 `rgba(255,255,255,0.05)` 배경. `.active` 시 `rgba(56,189,248,0.1)` 배경과 좌측 `4px solid #38bdf8` 강조선.

**`.nav-title`** — margin `0 0 4px 0`, `font-size: 0.95rem`, `font-weight: 500`.

**`.nav-desc`** — `font-size: 0.8rem`, 색상 `#94a3b8`, `white-space: nowrap`, `overflow: hidden`, `text-overflow: ellipsis`.

**`.gallery-pane`** — 중앙 갤러리 영역. `flex: 1`, `flex-direction: column`, 배경 `#0f172a`, `overflow: hidden`.

**`.preview-header`** — 갤러리 상단 제어 영역. padding `16px`, 배경 `#1e293b`, 하단 경계선, `flex column`, `align-items: stretch`, `gap: 12px`. 중첩된 `h2`는 margin 0.

**`.agent-controls`** — 에이전트 제어 컨테이너. `display: flex`, `gap: 8px`, `justify-content: space-between`. 중첩된 `fieldset`은 `border-radius: 8px`, 경계선 색 `rgba(148,163,184,0.2)`.

**`.theme-controls`** — 테마 제어 섹션. `font-size: 0.9rem`, 색상 `#94a3b8`, `flex column`, `align-items: flex-start`, `gap: 4px`.

**`.color-input-group`** — 색상 입력 그룹. `display: flex`, `gap: 8px`, `align-items: center`.

**`.color-input`** — 색상 피커 input. `border: none`, padding 0, `width: 24px`, `height: 24px`, `cursor: pointer`, `background: none`.

**`.message-controls`** — 메시지 제어 섹션. `display: flex`, `gap: 8px`, `align-items: center`, `font-size: 0.9rem`, 색상 `#94a3b8`.

**`button`** — 버튼 기본 스타일. 배경 `#38bdf8`(하늘색), 텍스트 `#0f172a`, `border: none`, padding `6px 12px`, `border-radius: 4px`, `font-weight: 600`, `cursor: pointer`. `:hover` 시 `#7dd3fc`. `:disabled` 시 배경 `#475569`, 텍스트 `#94a3b8`, `cursor: not-allowed`.

**`.preview-content`** — 미리보기 콘텐츠 영역. `flex: 1`, padding `24px`, `overflow-y: auto`, `justify-content: center`.

**`.surface-container`** — surface 래퍼. `width: 100%`, `max-width: 600px`, 배경 `rgba(255,255,255,0.05)`, 경계선 `rgba(148,163,184,0.2)`, `border-radius: 8px`, padding `24px`.

**`.inspector-pane`** — 우측 인스펙터 패널. 너비 `400px`, `flex column`, 좌측 경계선 `rgba(148,163,184,0.1)`, 배경 `#020617`.

**`.inspector-section`** — 인스펙터 내 섹션. `flex: 1`, `flex column`, 하단 경계선, `overflow: hidden`.

**`.inspector-header`** — 섹션 헤더. padding `12px 16px`, 배경 `#1e293b`, `font-weight: bold`, `font-size: 0.8rem`, `text-transform: uppercase`, 색상 `#94a3b8`.

**`.inspector-body`** — 섹션 본문. `flex: 1`, `overflow-y: auto`, padding `16px`, `font-family: 'JetBrains Mono', monospace`, `font-size: 0.8rem`, `white-space: pre-wrap`.

**`.log-list`** — 로그 목록. `display: flex`, `flex-direction: column-reverse`, `gap: 8px` (최신 항목이 위로 표시됨).

**`.log-entry`** — 개별 로그 항목. padding `8px`, 배경 `rgba(255,255,255,0.03)`, `border-radius: 4px`, 좌측 경계선 `2px solid #38bdf8`.

## 동작 흐름

이 파일은 순수 스타일 선언 모듈이며 실행 시점의 동작 흐름이 없다. `appStyles`가 import되면 Lit 컴포넌트의 `static styles` 배열에 포함되어 Shadow DOM에 주입된다.
