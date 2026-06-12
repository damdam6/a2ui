# renderers/angular/src/v0_8/components/divider.spec.ts

## 개요

`Divider` 컴포넌트에 대한 Angular 단위 테스트 파일이다. 컴포넌트가 올바르게 생성되고 테마 클래스가 `<hr>` 요소에 적용되는지 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`, `TestBed`

### 저장소 내부 모듈
- [`./divider`](./divider.ts.md): 테스트 대상 `Divider` 컴포넌트
- [`../rendering/theming`](../rendering/theming.ts.md): `Theme` 서비스
- [`../data/processor`](../data/processor.ts.md): `MessageProcessor` 서비스
- [`../types`](../types.ts.md): `DividerNode` 타입

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 픽스처 / 모킹 설정 (`beforeEach`)

- `mockTheme`: `new Theme()` 후 `components`를 `{Divider: 'divider-class'}` 로 설정.
- `mockNode`: `DividerNode` 타입, `{id: 'div-1', type: 'Divider', weight: 1, properties: {}}`.
- `TestBed` 제공자: `Theme → mockTheme`, `MessageProcessor → {}`.
- 입력값 설정: `surfaceId: 'surface-1'`, `component: mockNode`, `weight: 1`.

### 테스트 케이스

| 케이스명 | 검증 동작 | 픽스처/모킹 |
|----------|-----------|-------------|
| `should create` | 컴포넌트 인스턴스가 truthy인지 확인 | 기본 설정 |
| `should apply theme class` | DOM의 `<hr>` 요소 `className`이 `'divider-class'`를 포함하는지 확인 | `mockTheme.components.Divider` |

## 동작 흐름

`TestBed`로 `Divider`를 컴파일·마운트한 후 `detectChanges`를 통해 뷰를 갱신하고, `querySelector('hr')`로 DOM 요소를 조회해 클래스 적용 여부를 검증한다.
