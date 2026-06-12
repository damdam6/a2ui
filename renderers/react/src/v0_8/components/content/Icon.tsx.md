# renderers/react/src/v0_8/components/content/Icon.tsx

## 개요

`Icon` 컴포넌트는 Material Symbols Outlined 폰트를 사용하여 아이콘을 렌더링한다. `IconNode`의 `name` 속성(camelCase 형태)을 snake_case로 변환하여 `<span class="g-icon">` 내부에 텍스트 콘텐츠로 삽입한다. 이 방식은 Lit 렌더러의 접근법과 동일하다. 아이콘 이름이 없으면 렌더링을 생략한다.

## 의존성

### 외부 패키지
- `react` — `memo`
- `@a2ui/web_core/types/types` — `Types.IconNode` (타입 전용)

### 저장소 내부 모듈
- [`../../types`](../../types.ts.md) — `A2UIComponentProps` 타입
- [`../../hooks/useA2UIComponent`](../../hooks/useA2UIComponent.ts.md) — `useA2UIComponent` 훅
- [`../../lib/utils`](../../lib/utils.ts.md) — `classMapToString`, `stylesToObject`

## Exports

- `Icon` — named export, memo로 감싼 함수형 컴포넌트
- `default` — `Icon`의 default export

## 상세 명세

### `toSnakeCase(str: string): string`

비공개 헬퍼 함수. camelCase 문자열을 snake_case로 변환한다.

**동작:** 정규식 `/([A-Z])/g`로 모든 대문자를 찾아 앞에 `_`를 붙인 뒤 전체를 소문자로 변환한다. 예: `"shoppingCart"` → `"shopping_cart"`. Lit 렌더러와 동일한 방식이다.

### `Icon`

**시그니처**: `memo(function Icon({ node, surfaceId }: A2UIComponentProps<Types.IconNode>): JSX.Element | null)`

**동작 단계:**
1. `useA2UIComponent(node, surfaceId)`로 `theme`과 `resolveString`을 획득한다.
2. `resolveString(props.name)`으로 아이콘 이름을 해석한다.
3. 이름이 null이면 `null`을 반환하고 중단한다.
4. `toSnakeCase(iconName)`으로 이름을 snake_case로 변환하여 `snakeCaseName`에 저장한다.
5. `node.weight`가 `undefined`가 아니면 `{'--weight': node.weight}`를 `hostStyle`에 설정하고, 없으면 `{}`를 사용한다.
6. 루트 `<div className="a2ui-icon" style={hostStyle}>`를 렌더링한다.
7. 내부에 `<section className={classMapToString(theme.components.Icon)} style={stylesToObject(theme.additionalStyles?.Icon)}>`를 렌더링한다.
8. section 내부에 `<span className="g-icon">{snakeCaseName}</span>`을 렌더링한다.

**경계 케이스:**
- 이름이 null이면 컴포넌트 전체가 `null`을 반환한다.
- 이미 snake_case인 이름에 대해서도 `toSnakeCase`를 거치지만 변환이 없으므로 안전하다.

**사용 전제:** Material Symbols Outlined 폰트가 HTML에 로드되어 있어야 한다 (`<link href="https://fonts.googleapis.com/css2?family=Material+Symbols+Outlined" rel="stylesheet">`).

## 동작 흐름

이름 해석 → null 확인 → snake_case 변환 → 호스트 스타일 계산 → 루트 div + section + `<span class="g-icon">` 구조 렌더링.
