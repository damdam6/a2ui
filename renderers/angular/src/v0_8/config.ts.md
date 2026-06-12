# renderers/angular/src/v0_8/config.ts

## 개요

Angular 애플리케이션에 a2ui 렌더러를 설정하는 진입점 함수를 제공한다. `provideA2UI`를 호출하면 `Catalog`와 `Theme` 두 핵심 의존성을 Angular DI 컨테이너에 등록하는 `EnvironmentProviders` 객체가 반환된다.

## 의존성

### 외부 패키지
- `@angular/core` — `EnvironmentProviders`, `makeEnvironmentProviders`

### 저장소 내부 모듈
- [`./rendering`](./rendering/index.ts.md) — `Catalog`, `Theme` (렌더링 모듈 re-export)
- [`./types`](./types.ts.md) — `Theme as ThemeType` (타입 전용 import)

## Exports

| 이름 | 종류 |
|---|---|
| `provideA2UI` | 함수 |

## 상세 명세

### `provideA2UI(config: {catalog: Catalog; theme: ThemeType}): EnvironmentProviders`

**매개변수**
- `config.catalog`: `Catalog` 인스턴스 — 컴포넌트 목록을 담은 카탈로그 객체.
- `config.theme`: `ThemeType` — 테마 설정 데이터 객체 (렌더링 모듈의 `Theme` 클래스가 아닌 `types`의 `Theme` 타입).

**동작 로직**
1. `makeEnvironmentProviders`를 호출하여 두 개의 provider 배열을 생성.
2. 첫 번째 provider: `{provide: Catalog, useValue: config.catalog}` — 전달된 `catalog` 인스턴스를 그대로 주입.
3. 두 번째 provider: `{provide: Theme, useFactory: () => { ... }}` — 팩토리 함수에서 `new Theme()`를 생성하고 `theme.update(config.theme)`를 호출한 뒤 반환. 이를 통해 전달된 테마 데이터를 `Theme` 서비스 인스턴스에 적용.
4. 완성된 `EnvironmentProviders`를 반환.

## 동작 흐름

애플리케이션의 `bootstrapApplication` 또는 `ApplicationConfig.providers`에서 `provideA2UI({catalog, theme})`를 호출하면, Angular가 `Catalog`와 `Theme` 토큰을 DI 트리에 등록한다. 이후 a2ui의 모든 컴포넌트와 서비스는 이 두 의존성을 주입받아 동작한다.
