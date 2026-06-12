# renderers/react/tests/v0_8/unit/components/Image.test.tsx

## 개요

A2UI 명세를 따르는 `Image` 컴포넌트의 단위 테스트 모음이다. 필수 속성 `url`과 `usageHint`, 선택 속성 `fit`의 처리를 검증한다. `img` 요소 렌더링, `src` 어트리뷰트 설정, 빈 URL 처리, `usageHint`별 테마 클래스 적용, `fit` 모드별 `--object-fit` CSS 변수 설정, DOM 구조를 포괄적으로 테스트한다.

## 의존성

### 외부 패키지
- `vitest` — `describe`, `it`, `expect`
- `@testing-library/react` — `render`
- `react` — `React`

### 저장소 내부 모듈
- [`../../utils`](../../utils/index.ts.md) — `TestWrapper`, `TestRenderer`, `createSimpleMessages`

## Exports

이 파일은 아무것도 export하지 않는다.

## 상세 명세 (테스트 케이스)

### describe: `Image Component`

#### describe: `Basic Rendering`

- **`should render an img element`**
  - 검증: `url: {literalString: 'https://example.com/image.jpg'}`, `usageHint: 'mediumFeature'`로 렌더링하면 `img` 요소가 DOM에 존재한다.

- **`should render with wrapper div`**
  - 검증: `.a2ui-image` 클래스를 가진 래퍼 요소가 DOM에 존재한다.

- **`should set src attribute from url`**
  - 검증: `url: {literalString: 'https://example.com/photo.png'}` → `img.src === 'https://example.com/photo.png'`.

- **`should have empty alt attribute`**
  - 검증: `img.alt === ''`. Image 컴포넌트는 항상 빈 alt를 사용한다.

- **`should render different src for different url inputs`**
  - 검증: `'https://example.com/first.jpg'`와 `'https://example.com/second.jpg'`를 각각 독립 렌더링하면 두 img의 `src`가 서로 다르고 각자의 URL과 일치한다.

- **`should return null for empty url`**
  - 검증: `url: {literalString: ''}` → `img` 요소가 DOM에 **없다**(`not.toBeInTheDocument()`). 빈 URL은 null 렌더링으로 처리된다.

#### describe: `Usage Hints`

대상 힌트 배열: `['icon', 'avatar', 'smallFeature', 'mediumFeature', 'largeFeature', 'header']`.

- **`should render with usageHint="${hint}"` (파라미터화)**
  - 검증: 각 `usageHint` 값에 대해 `img` 요소가 정상 렌더링된다.
  - `usageHints.forEach`로 6개의 개별 테스트를 동적 생성한다.

- **`should apply different theme classes for different usageHints`**
  - 검증: `'mediumFeature'`와 `'avatar'`를 각각 독립 렌더링하면 두 컨테이너 모두 `section` 요소가 존재하고 태그명이 `'SECTION'`이다. `usageHint`가 `section`의 테마 클래스 병합에 영향을 준다는 것을 양측 렌더링 성공으로 확인한다.

#### describe: `Fit Mode`

- **`should default to fill fit mode`**
  - 검증: `fit` 속성 미지정 시 `section.style.getPropertyValue('--object-fit') === 'fill'`.

- **`should set --object-fit CSS variable for fit="${fit}"` (파라미터화)**
  - 대상 값: `['contain', 'cover', 'fill', 'none', 'scale-down']` (배열 `fitModes`).
  - 검증: 각 `fit` 값을 전달하면 `section`의 인라인 CSS 변수 `--object-fit`이 해당 값으로 설정된다.
  - `fitModes.forEach`로 5개의 개별 테스트를 동적 생성한다.

- **`should set different --object-fit for different fit inputs`**
  - 검증: `fit: 'cover'`와 `fit: 'contain'`을 각각 렌더링하면 `--object-fit`이 각각 `'cover'`, `'contain'`이고 서로 다르다.

#### describe: `Theme Support`

- **`should render within section container`**
  - 검증: `section` 요소가 DOM에 존재하고, `section` 내에 `img` 요소가 중첩되어 있다.

- **`should apply theme classes to section`**
  - 검증: `section.className`이 정의되어 있고(`toBeDefined`), 타입이 `'string'`이며, `classList.length > 0`이다. 테마가 section에 레이아웃 클래스를 부여함을 확인.

#### describe: `Structure`

- **`should have correct DOM structure`**
  - 검증: `.a2ui-image`의 자식이 1개이고 태그명이 `'SECTION'`, 그 section의 자식이 1개이고 태그명이 `'IMG'`이다. DOM 구조는 `div.a2ui-image > section > img`.

## 동작 흐름

각 테스트는 `createSimpleMessages`로 `Image` 컴포넌트 메시지를 생성한다. 핵심 렌더링 로직은 `url`이 빈 문자열이면 `null`을 반환하여 아무것도 렌더링하지 않는다. 비어 있지 않은 URL은 `section[style="--object-fit: <fit>"] > img[src=<url>][alt=""]` 구조로 렌더링되며, `section`에는 `usageHint`에 따른 테마 클래스가 병합된다. `fit` 속성의 기본값은 `'fill'`이다. 파라미터화 테스트는 배열에 `forEach`를 호출하여 동적으로 `it` 블록을 등록한다.
