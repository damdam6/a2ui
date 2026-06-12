# renderers/angular/src/v0_9/catalog/basic/choice-picker.component.spec.ts

## 개요

`ChoicePickerComponent`의 Angular 단위 테스트 파일이다. Jasmine + `TestBed`를 사용하여 컴포넌트 생성, 옵션 렌더링, 선택 변경 시 `onUpdate` 호출, 칩 UI의 활성 상태 토글 동작을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`, `TestBed`
- `@angular/core` — `signal as angularSignal`
- `@a2ui/web_core/v0_9` — `DynamicString`

### 저장소 내부 모듈
- [`./choice-picker.component`](./choice-picker.component.ts.md)
- [`../../core/a2ui-renderer.service`](../../core/a2ui-renderer.service.ts.md)
- [`../../core/component-binder.service`](../../core/component-binder.service.ts.md)
- [`../../core/test-utils`](../../core/test-utils.ts.md)

## 픽스처 / 모킹

- **`mockRendererService`**: `getSurface` 스파이가 빈 `componentsModel`과 빈 `catalog.components`를 반환.
- **`mockBinder`**: `bind` 메서드 스파이.
- **`defaultProps`**: `label=''`, `options=[]`, `value=[]`, `variant='mutuallyExclusive'`, `displayStyle='checkbox'`, `isValid=true`, `validationErrors=[]`.
- `surfaceId='test-surface'`, `componentId='test-choice-picker'`, `dataContextPath='/'`로 설정.

## 테스트 케이스

| 테스트 이름 | 검증 동작 |
|---|---|
| `should create` | 컴포넌트 인스턴스가 truthy인지 확인 |
| `should render options` | `options=[{label:'Opt 1', value:'1'}, {label:'Opt 2', value:'2'}]`, `value=['1']` 설정 후 `.a2ui-option-label` 엘리먼트가 2개 존재하고 첫 번째 텍스트가 `'Opt 1'`을 포함하는지 확인 |
| `should call onUpdate when option selected` | `value` prop에 `onUpdate` 스파이를 주입한 뒤 `value="2"` input 클릭 시 `onUpdate(['2'])`가 호출되는지 확인 (단일 선택 모드) |
| `should render chips and toggle selection` | `displayStyle='chips'`, `variant='multipleSelection'`, `value=['c1']` 설정 후: `.a2ui-chip` 2개 렌더링 확인, 첫 번째 칩에 `active` 클래스 있음 확인, 두 번째 칩 클릭 시 `onUpdate(['c1', 'c2'])` 호출 확인, 첫 번째 칩 클릭 시 `onUpdate([])` 호출 확인 |
