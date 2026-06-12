# renderers/angular/src/v0_9/catalog/basic/text-field.component.spec.ts

## 개요

`TextFieldComponent`의 Angular 단위 테스트 파일이다. 컴포넌트 생성, 레이블 렌더링 조건, 입력값 바인딩, `inputType` 계산 로직, `onUpdate` 콜백 연동, 그리고 검증 오류 메시지 표시를 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`, `TestBed`
- `@angular/platform-browser`: `By`

### 저장소 내부 모듈
- [`./text-field.component`](./text-field.component.ts.md): 테스트 대상
- [`../../core/a2ui-renderer.service`](../../core/a2ui-renderer.service.ts.md): `A2uiRendererService`, `A2UI_RENDERER_CONFIG` (실제 서비스 사용, `catalogs: []` 설정)
- [`../../core/test-utils`](../../core/test-utils.ts.md): `setComponentProps`, `createBoundProperty`, `ComponentToProps`

## 픽스처 및 모킹

`A2uiRendererService`는 실제 서비스를 사용하며, `A2UI_RENDERER_CONFIG`에 `{catalogs: []}`를 주입한다. `ComponentBinder`는 별도로 mock하지 않는다.

### `defaultProps`
- `label`: `createBoundProperty('Username')`
- `value`: `createBoundProperty('testuser')`
- `variant`: `createBoundProperty('shortText')`
- `isValid`: `createBoundProperty(true)`
- `validationErrors`: `createBoundProperty<string[]>([])`

`surfaceId`는 `'surf1'`로 설정된다.

## 테스트 케이스

### `should create`
`detectChanges()` 후 `component`가 truthy임을 확인한다.

### `should render label if provided`
`detectChanges()` 후 `By.css('label')`로 쿼리한 엘리먼트의 `textContent`가 `'Username'`임을 검증한다.

### `should not render label if not provided`
`label`을 `createBoundProperty(null as any)`로 설정한 뒤 `detectChanges()`. `By.css('label')` 결과가 `null`(falsy)임을 검증하여, `@if(label())` 조건이 올바르게 동작함을 확인한다.

### `should render input with correct value`
`detectChanges()` 후 `By.css('input')`로 쿼리한 엘리먼트의 `nativeElement.value`가 `'testuser'`임을 검증한다.

### `should return correct input type based on variant`
`detectChanges()` 없이 computed signal을 직접 호출:
- 기본(`'shortText'`) → `component.inputType()`이 `'text'`
- `'obscured'` → `'password'`
- `'number'` → `'number'`

각 variant마다 `setComponentProps`로 업데이트하며 반환값을 검증한다.

### `should call onUpdate when input changes`
`detectChanges()` 후 `input` 엘리먼트의 `nativeElement.value`를 `'newuser'`로 바꾸고 `triggerEventHandler('input', {target: input.nativeElement})`를 호출한다. `component.props()['value']!.onUpdate`가 `'newuser'`와 함께 호출되었음을 jasmine spy로 검증한다.

### `should show error messages when checks fail`
`isValidProp`와 `errorsProp`를 별도 `createBoundProperty`로 생성하여 설정한다. 초기 상태(`isValid: true`, `errors: []`)에서 `.a2ui-error-message` 엘리먼트가 없음을 확인한다. 이후 `isValidProp.value.set(false)`, `errorsProp.value.set(['Value is required'])`를 호출하고 `detectChanges()` 후 `.a2ui-error-message`가 존재하고 텍스트에 `'Value is required'`가 포함됨을 검증한다.

### `should handle multiple error messages`
`isValid: false`, `validationErrors: ['Error 1', 'Error 2']`로 설정하고 `detectChanges()`. `queryAll(By.css('.a2ui-error-message'))`의 길이가 2이고, 각각 `'Error 1'`, `'Error 2'`를 포함함을 검증한다.
