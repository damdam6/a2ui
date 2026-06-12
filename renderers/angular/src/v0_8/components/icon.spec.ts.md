# renderers/angular/src/v0_8/components/icon.spec.ts

## 개요

`Icon` 컴포넌트에 대한 Angular 단위 테스트 파일이다. 컴포넌트 생성, 아이콘 이름 렌더링, null 처리, 그리고 camelCase/TitleCase → snake_case 변환 로직을 검증한다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `ComponentFixture`, `TestBed`
- `@angular/platform-browser`: `By`

### 저장소 내부 모듈
- [`./icon`](./icon.ts.md): 테스트 대상 `Icon` 컴포넌트
- [`../data/processor`](../data/processor.ts.md): `MessageProcessor` 서비스
- [`../rendering/theming`](../rendering/theming.ts.md): `Theme` 서비스
- [`../rendering/catalog`](../rendering/catalog.ts.md): `Catalog` 서비스

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 픽스처 / 모킹 설정 (`beforeEach`)

- `mockTheme`: `new Theme()` 후 `components`를 `{Icon: 'icon-class'}` 로 설정.
- `TestBed` 제공자: `MessageProcessor → {resolvePrimitive: (p) => p?.literalString || p}`, `Theme → mockTheme`, `Catalog → {}`.
- 입력값 설정: `surfaceId: 'surface-1'`, `component: {id: 'icon-1', type: 'Icon', weight: 1}`, `weight: 1`, `name: null`.

### 테스트 케이스

| 케이스명 | 검증 동작 | 픽스처/모킹 |
|----------|-----------|-------------|
| `should create` | 컴포넌트 인스턴스가 truthy인지 확인 | 기본 설정 |
| `should render icon name inside span if provided` | `name: {literalString: 'home'}` 입력 시 `.g-icon` span이 존재하고 textContent가 `'home'`인지, `section`의 className이 `'icon-class'`인지 확인 | `setInput('name', ...)` |
| `should NOT render anything if name is null` | `name: null` 입력 시 `.g-icon` span과 `section` 모두 렌더링되지 않는지 확인 | `name` 입력 null |
| `should convert camelCase name to snake_case` | `name: {literalString: 'arrowForward'}` 입력 시 `.g-icon` textContent가 `'arrow_forward'`인지 확인 | camelCase 입력 |
| `should convert TitleCase name to snake_case` | `name: {literalString: 'ArrowForward'}` 입력 시 `.g-icon` textContent가 `'arrow_forward'`인지 확인 | TitleCase 입력 |

## 동작 흐름

각 테스트는 `setInput`으로 `name`을 설정하고 `detectChanges`를 호출한 뒤, `fixture.debugElement.query(By.css(...))`로 DOM 요소를 조회해 렌더링 여부와 텍스트 내용을 검증한다.
