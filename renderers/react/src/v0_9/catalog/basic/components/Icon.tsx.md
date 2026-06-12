# renderers/react/src/v0_9/catalog/basic/components/Icon.tsx

## 개요

`Icon` 컴포넌트는 a2ui Basic Catalog의 아이콘 UI 요소다. `IconApi` 스키마를 기반으로 생성되며, `props.name`이 `{svgPath: string}` 객체인 경우 인라인 SVG로 렌더링하고, 문자열인 경우 Material Symbols Outlined 폰트 기반 아이콘을 렌더링하는 두 가지 경로를 지원한다. camelCase 아이콘 이름을 Material Symbols의 snake_case 이름으로 변환하는 로직과 일부 특별한 이름의 오버라이드 테이블을 포함한다.

## 의존성

### 외부 패키지
- `react` — `React.CSSProperties` 타입 사용
- `@a2ui/web_core/v0_9/basic_catalog` — `IconApi` 스키마 가져오기

### 저장소 내부 모듈
- [`../../../adapter`](../../../adapter.tsx.md) — `createComponentImplementation` 유틸리티
- [`../utils`](../utils.ts.md) — `getBaseLeafStyle`, `useBasicCatalogStyles`

## Exports

| 이름 | 종류 |
|------|------|
| `Icon` | 상수 (React 컴포넌트, `createComponentImplementation` 반환값) |

## 상세 명세

### `ICON_NAME_OVERRIDES`

**타입**: `Record<string, string>`

camelCase 이름을 Material Symbols 이름으로 매핑하는 오버라이드 테이블. `toMaterialSymbol`의 일반 변환 로직으로는 올바른 결과가 나오지 않는 특수 케이스를 처리한다.

| 입력 키 | 출력 값 |
|---------|---------|
| `'play'` | `'play_arrow'` |
| `'rewind'` | `'fast_rewind'` |
| `'favoriteOff'` | `'favorite_border'` |
| `'starOff'` | `'star_border'` |

### `toMaterialSymbol(str: string): string`

**매개변수**: `str: string` — camelCase 아이콘 이름

**반환 타입**: `string` — Material Symbols 폰트용 snake_case 이름

**동작 로직**:
1. `ICON_NAME_OVERRIDES[str]`에 해당 키가 있으면 즉시 해당 값을 반환한다.
2. 없으면 `str.replace(/[A-Z]/g, letter => '_' + letter.toLowerCase())`를 적용하여 각 대문자 앞에 언더스코어를 추가하고 소문자로 변환한다. 예: `'shoppingCart'` → `'shopping_cart'`, `'skipPrevious'` → `'skip_previous'`.

### `Icon`

**시그니처**: `createComponentImplementation(IconApi, ({props}) => JSX.Element)`

**렌더 함수 동작 로직**:

1. `useBasicCatalogStyles()` 호출.
2. `isPath`: `typeof props.name === 'object' && props.name !== null && 'svgPath' in props.name` — SVG 경로 모드 여부 판단.
3. `baseStyle` 구성:
   - `...getBaseLeafStyle()` — `{boxSizing: 'border-box'}`
   - `display: 'inline-flex'`, `alignItems: 'center'`, `justifyContent: 'center'`
   - `fontSize: 'var(--a2ui-icon-size, var(--a2ui-font-size-xl, 24px))'`
   - `color: 'var(--a2ui-icon-color, inherit)'`
   - `lineHeight: 1`
4. `isPath`가 `true`이면 **SVG 모드** 반환:
   - `path = (props.name as {svgPath: string}).svgPath`
   - `<svg className="a2ui-icon svg" viewBox="0 0 24 24">` — `baseStyle`에 `fill: 'currentColor'`, `width/height: 'var(--a2ui-icon-size, 24px)'` 추가
   - `<path d={path}></path>` 내부에 삽입
5. `isPath`가 `false`이면 **폰트 모드** 반환:
   - `iconName = typeof props.name === 'string' ? toMaterialSymbol(props.name) : ''`
   - `fontStyle` = `baseStyle` + `fontFamily: 'var(--a2ui-icon-font-family, "Material Symbols Outlined", sans-serif)'`, `fontVariationSettings: 'var(--a2ui-icon-font-variation-settings, "FILL" 1)'`, `fontWeight: 'normal'`, `fontStyle: 'normal'`, `letterSpacing: 'normal'`, `textTransform: 'none'`
   - `<span className="material-symbols-outlined" style={fontStyle}>{iconName}</span>` 반환

**props**:
- `props.name` (`string | {svgPath: string} | undefined`) — 아이콘 식별자

## 동작 흐름

`props.name`의 타입에 따라 렌더링 경로가 분기된다. 문자열이면 `toMaterialSymbol`로 변환 후 폰트 클래스를 사용하는 `<span>`을 반환하고, 객체이면 직접 `<svg>/<path>`를 반환한다. 내부 상태 없음.
