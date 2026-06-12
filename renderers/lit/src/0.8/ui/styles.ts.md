# renderers/lit/src/0.8/ui/styles.ts

## 개요

이 파일은 A2UI Lit 컴포넌트 전체에서 공유하는 구조적 CSS 스타일(`structuralStyles`)을 빌드하고 내보내는 단일 목적 모듈이다. 브라우저 환경에서는 Constructable CSSStyleSheet API를 이용해 스타일시트를 구성하고, SSR 환경(`typeof window === 'undefined'`)에서는 빈 배열을 반환한다. 동일한 `CSSStyleSheet` 객체를 여러 컴포넌트에서 공유하므로 CSS가 중복 파싱되지 않는다.

## 의존성

### 외부 패키지
- `lit` — `CSSResultGroup` (타입 import 전용)
- `@a2ui/web_core/styles/index` — `Styles` 네임스페이스 (`Styles.structuralStyles` CSS 문자열)

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 |
|---|---|
| `structuralStyles` | 상수 (`CSSResultGroup`) |

## 상세 명세

### 함수: `buildStructuralStyles()` (비공개 헬퍼)

시그니처: `const buildStructuralStyles = (): CSSResultGroup`

외부로 노출되지 않는 내부 팩토리 함수.

동작 단계:
1. `typeof window === 'undefined'`를 확인한다. `true`이면(SSR 환경) 빈 배열 `[]`을 즉시 반환한다.
2. `new CSSStyleSheet()`로 빈 Constructable CSSStyleSheet 인스턴스를 생성한다.
3. `styleSheet.replaceSync(Styles.structuralStyles)`를 호출하여 `@a2ui/web_core/styles/index`에서 가져온 CSS 문자열을 동기적으로 시트에 적용한다.
4. 완성된 `styleSheet`를 반환한다.
5. 2~3단계 중 예외가 발생하면 `'Failed to construct structural styles.'` 메시지와 원인(`cause: e`)을 담은 새 `Error`를 throw한다.

### 상수: `structuralStyles`

`buildStructuralStyles()`를 모듈 최상위 레벨에서 즉시 호출한 결과를 저장한 상수. 타입은 `CSSResultGroup`. Lit 컴포넌트의 `static styles` 배열에 포함시켜 사용한다.

## 동작 흐름

모듈이 처음 import될 때 `buildStructuralStyles()`가 한 번 실행되어 `structuralStyles`에 결과가 고정된다. 이후 `Row`, `Slider`, `Text`, `Tabs` 등 다수의 컴포넌트가 이 상수를 `static styles` 배열의 첫 번째 요소로 포함하여 공통 구조 스타일을 상속한다.
