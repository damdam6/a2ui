# renderers/angular/src/v0_9/core/test-utils.ts

## 개요

Angular 컴포넌트 단위 테스트에서 공통으로 사용하는 타입 안전 유틸리티 함수와 타입을 제공한다. `BoundProperty` 목 객체 생성과 컴포넌트 `props` 입력 설정을 타입 안전하게 처리할 수 있도록 돕는 테스트 전용 헬퍼 파일이다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`
- `@angular/core` — `signal`

### 저장소 내부 모듈
- [`./types`](./types.ts.md) — `BoundProperty`, `ComponentTemplate`

## Exports

- `ComponentToProps<C>` (타입)
- `setComponentProps` (함수)
- `createBoundProperty` (함수)

## 상세 명세

### `type ComponentToProps<C>`

제네릭 타입 유틸리티. 컴포넌트 타입 `C`에서 `props` 입력의 타입을 추출한다.

- `C`가 `{ props: () => infer P }` 형태이면 `P`를 반환한다.
- 그렇지 않으면 `never`를 반환한다.
- Signal 입력(`props: () => Props`)을 사용하는 컴포넌트에서 Props 타입을 자동으로 추론하는 데 사용된다.

---

### `function setComponentProps<T extends { props: () => any }>(fixture: ComponentFixture<T>, props: ComponentToProps<T>): void`

**매개변수**:
- `fixture: ComponentFixture<T>` — 대상 컴포넌트의 테스트 픽스처
- `props: ComponentToProps<T>` — 설정할 props 값. `ComponentToProps<T>`로 타입이 추론되므로 타입 안전하다.

**동작**: `fixture.componentRef.setInput('props', props)`를 호출하여 컴포넌트의 `props` Signal 입력을 업데이트한다. 직접 `setInput`을 호출하는 것보다 타입 안전한 래퍼 역할을 한다.

---

### `function createBoundProperty<T>(val: T, template?: ComponentTemplate): BoundProperty<T>`

**매개변수**:
- `val: T` — `BoundProperty`의 초기 값
- `template?: ComponentTemplate` — 선택적 템플릿 정보 (자식 컴포넌트 목록 렌더링 시 사용)

**반환**: `BoundProperty<T>`를 만족하는 객체 리터럴.

반환 객체의 구성:
- `value`: `signal(val)` — Angular Signal로 감싼 초기 값
- `raw`: `val` — 원시 값 그대로
- `template`: `template` 매개변수 값 (제공되지 않으면 `undefined`)
- `onUpdate`: `jasmine.createSpy('onUpdate')` — 업데이트 콜백 스파이

`satisfies BoundProperty<T>` 타입 어설션으로 반환 객체가 `BoundProperty<T>` 인터페이스를 준수함을 컴파일 타임에 검증한다.

## 동작 흐름

테스트 코드에서 `createBoundProperty`로 모의 `BoundProperty` 객체를 만들고, `setComponentProps`로 컴포넌트에 주입한다. 이 유틸리티들은 테스트 파일에서만 사용하도록 설계되었으며, 프로덕션 코드에서는 사용하지 않는다.
