# renderers/react/src/v0_8/components/content/Image.tsx

## 개요

`Image` 컴포넌트는 URL로 지정된 이미지를 렌더링한다. `usageHint`(사용 목적 힌트)에 따라 테마 클래스를 병합하고, `fit` 속성을 `--object-fit` CSS 변수로 변환하여 `object-fit` 동작을 제어한다. Lit 렌더러의 `Styles.merge` 패턴을 그대로 따른다.

## 의존성

### 외부 패키지
- `react` — `memo`
- `@a2ui/web_core/types/types` — `Types.ImageNode`, `Types.StringValue` (타입 전용)

### 저장소 내부 모듈
- [`../../types`](../../types.ts.md) — `A2UIComponentProps` 타입
- [`../../hooks/useA2UIComponent`](../../hooks/useA2UIComponent.ts.md) — `useA2UIComponent` 훅
- [`../../lib/utils`](../../lib/utils.ts.md) — `classMapToString`, `stylesToObject`, `mergeClassMaps`

## Exports

- `Image` — named export, memo로 감싼 함수형 컴포넌트
- `default` — `Image`의 default export

## 상세 명세

### 타입 별칭

- `UsageHint`: `'icon' | 'avatar' | 'smallFeature' | 'mediumFeature' | 'largeFeature' | 'header'` — 이미지 사용 목적을 나타내는 리터럴 유니온 타입. 파일 내부에서만 사용된다.
- `FitMode`: `'contain' | 'cover' | 'fill' | 'none' | 'scale-down'` — CSS `object-fit` 값에 대응하는 리터럴 유니온 타입. 파일 내부에서만 사용된다.

### `Image`

**시그니처**: `memo(function Image({ node, surfaceId }: A2UIComponentProps<Types.ImageNode>): JSX.Element | null)`

**동작 단계:**
1. `useA2UIComponent(node, surfaceId)`로 `theme`과 `resolveString`을 획득한다.
2. `resolveString(props.url)`로 이미지 URL을 해석한다.
3. `resolveString((props as Record<string, unknown>).altText as Types.StringValue | undefined)`로 대체 텍스트를 해석한다. `altText`는 표준 `ImageNode` 타입에 없으므로 타입 단언을 사용한다.
4. `props.usageHint as UsageHint | undefined`로 사용 목적 힌트를 추출한다.
5. `(props.fit as FitMode) ?? 'fill'`로 fit 모드를 추출한다. 값이 없으면 기본값 `'fill'`을 사용한다.
6. `mergeClassMaps(theme.components.Image.all, usageHint ? theme.components.Image[usageHint] : {})`로 기본 클래스와 힌트별 클래스를 병합하여 `classes`를 생성한다.
7. 스타일 객체를 구성한다: `stylesToObject(theme.additionalStyles?.Image)`의 결과에 `'--object-fit': fit`을 스프레드로 추가한 뒤 `React.CSSProperties`로 단언한다.
8. URL이 null이면 `null`을 반환하고 중단한다.
9. `node.weight`가 있으면 `{'--weight': node.weight}`를 `hostStyle`로 설정하고, 없으면 `{}`를 사용한다.
10. 루트 `<div className="a2ui-image" style={hostStyle}>`를 렌더링한다.
11. 내부에 `<section className={classMapToString(classes)} style={style}>`를 렌더링한다.
12. section 내부에 `<img src={url} alt={altText || ''} />`를 렌더링한다. `altText`가 null이면 빈 문자열을 사용한다.

**경계 케이스:**
- URL이 null이면 컴포넌트 전체가 `null`을 반환한다. (스타일/클래스 계산 후 확인됨)
- `usageHint`가 없으면 `theme.components.Image.all` 클래스만 적용된다.
- `fit` 기본값은 `'fill'`이다.
- `altText`는 null인 경우 빈 문자열 `''`로 대체된다.

## 동작 흐름

URL/altText/hint/fit 해석 → 클래스 병합 → 스타일 객체 구성(CSS 변수 포함) → URL 없으면 조기 반환 → 호스트 스타일 계산 → 루트 div + section + `<img>` 렌더링.
