# renderers/angular/src/v0_9/catalog/basic/card.component.spec.ts

## 개요

`CardComponent`의 Angular 단위 테스트 파일이다. `TestBed`와 Jasmine을 사용하여 컴포넌트 생성 및 자식 컴포넌트 호스트 렌더링을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing` — `ComponentFixture`, `TestBed`
- `@angular/core` — `Component`, `input`
- `@angular/platform-browser` — `By`
- `@a2ui/web_core/v0_9` — `ComponentModel`

### 저장소 내부 모듈
- [`./card.component`](./card.component.ts.md)
- [`../../core/a2ui-renderer.service`](../../core/a2ui-renderer.service.ts.md)
- [`../../core/component-binder.service`](../../core/component-binder.service.ts.md)
- [`../../core/test-utils`](../../core/test-utils.ts.md)

## 픽스처 / 모킹

- **`DummyTextComponent`**: `selector: 'dummy-text-for-card'`, 템플릿 `<div>{{text}}</div>`. `props`, `surfaceId`, `componentId`, `dataContextPath` input을 가지는 스탠드얼론 컴포넌트. 카탈로그의 `'Text'` 타입 컴포넌트 자리에 사용된다.
- **`mockRendererService`**: `getSurface` 스파이가 `componentsModel`(`child-1` ComponentModel 포함)과 `catalog`(`DummyTextComponent` 포함)를 가진 객체를 반환한다.
- **`mockBinder`**: `bind` 메서드 스파이.
- **`defaultProps`**: `child = {id: 'child-1', basePath: '/'}` 로 구성.
- `surfaceId = 'test-surface'`, `dataContextPath = '/'`로 설정.

## 테스트 케이스

| 테스트 이름 | 검증 동작 |
|---|---|
| `should create` | 컴포넌트 인스턴스가 truthy인지 확인 |
| `should render component-host for child` | `a2ui-v09-component-host` 엘리먼트가 존재하고 `componentKey()`가 `{id: 'child-1', basePath: '/'}` 인지 확인 |
