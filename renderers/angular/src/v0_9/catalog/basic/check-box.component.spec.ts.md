# renderers/angular/src/v0_9/catalog/basic/check-box.component.spec.ts

## 개요

`CheckBoxComponent`의 Angular 단위 테스트 파일이다. Jasmine + `TestBed`를 사용하여 체크박스의 생성, 레이블/체크 상태 렌더링, 토글 시 `onUpdate` 호출, 테마 primary 색상 반영을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`, `TestBed`
- `@angular/core` — `signal as angularSignal`

### 저장소 내부 모듈
- [`./check-box.component`](./check-box.component.ts.md)
- [`../../core/a2ui-renderer.service`](../../core/a2ui-renderer.service.ts.md)
- [`../../core/component-binder.service`](../../core/component-binder.service.ts.md)
- [`../../core/test-utils`](../../core/test-utils.ts.md)

## 픽스처 / 모킹

- **`mockRendererService`**: `getSurface` 스파이가 빈 `componentsModel`과 빈 `catalog.components`를 반환한다.
- **`mockBinder`**: `bind` 메서드 스파이.
- **`defaultProps`**: `label=''`, `value=false`, `isValid=true`, `validationErrors=[]`.
- `surfaceId='test-surface'`, `dataContextPath='/'`로 설정.

## 테스트 케이스

| 테스트 이름 | 검증 동작 |
|---|---|
| `should create` | 컴포넌트 인스턴스가 truthy인지 확인 |
| `should show label and checked state` | `label='Check me'`, `value=true` 설정 후 `<input>` 엘리먼트의 `checked`가 `true`이고 텍스트에 `'Check me'`가 포함되는지 확인 |
| `should call onUpdate when toggled` | `value` prop에 `onUpdate` 스파이를 주입한 뒤 `<input>` 클릭 시 `onUpdate(true)`가 호출되는지 확인 |
| `should apply primary color when checked` | `value=true`, `theme.primaryColor='rgb(255, 0, 0)'` 설정 후 `<input>`의 computed style `accentColor`가 `'rgb(255, 0, 0)'`인지 확인 |
