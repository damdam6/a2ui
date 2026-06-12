# renderers/angular/a2ui_explorer/src/app/demo-catalog.ts

## 개요

`DemoCatalog`는 `BasicCatalogBase`를 상속하는 탐색기 전용 카탈로그 서비스로, 기본 카탈로그의 컴포넌트와 함수에 더해 커스텀 컴포넌트(`CustomSliderComponent`)와 데모용 함수(`capitalize`)를 추가 등록한다. `DemoComponent`의 provider에서 `AngularCatalog` 토큰을 이 클래스로 교체하여 사용된다.

## 의존성

### 외부 패키지
- `@angular/core` — `Injectable`
- `zod` — `z`
- `@a2ui/angular/v0_9` — `BasicCatalogBase`, `BASIC_FUNCTIONS`
- `@a2ui/web_core/v0_9` — `createFunctionImplementation`, `FunctionImplementation`

### 저장소 내부 모듈
- [`./custom-slider.component`](./custom-slider.component.ts.md) — `customSliderComponentDeclaration`

## Exports

| 이름 | 종류 |
|---|---|
| `DemoCatalog` | 클래스 (Angular 서비스, `BasicCatalogBase` 상속) |

## 상세 명세

### 클래스 `DemoCatalog extends BasicCatalogBase`

**데코레이터**: `@Injectable({ providedIn: 'root' })`

#### 생성자

매개변수 없음. 생성자 본문에서 직접 모든 설정을 조립한 후 `super(...)`를 호출한다.

**생성자 실행 단계:**

1. **`capitalize` 함수 구현 생성**

   `createFunctionImplementation`을 호출하여 `capitalizeImplementation: FunctionImplementation`을 생성한다.
   - 함수 명세: `{ name: 'capitalize', returnType: 'string', schema: z.object({ value: z.string().optional() }) as any }`
   - 구현 로직: `args.value || ''`를 `String()`으로 변환한 후, 첫 문자를 `toUpperCase()`로 변환하고 나머지를 `slice(1)`로 붙여 반환한다. 빈 문자열 입력 시 빈 문자열을 반환한다.

2. **함수 배열 조합**

   `const functions = [...BASIC_FUNCTIONS, capitalizeImplementation]`로 기본 함수 목록에 `capitalize`를 추가한다.

3. **`super()` 호출**

   `BasicCatalogBase`의 생성자에 다음 설정 객체를 전달한다:
   - `id: 'https://a2ui.org/specification/v0_9/catalogs/basic/catalog.json'` — 카탈로그 식별 URI
   - `components: {}` — 기본 카탈로그 컴포넌트를 직접 오버라이드하지 않음
   - `extraComponents: [customSliderComponentDeclaration]` — `CustomSliderComponent`를 추가 등록
   - `functions` — 위에서 조합한 함수 배열

## 동작 흐름

`DemoComponent` 초기화 시 `{ provide: AngularCatalog, useClass: DemoCatalog }` provider를 통해 이 클래스가 인스턴스화된다. 생성자에서 `BasicCatalogBase`에 커스텀 컴포넌트와 함수를 포함한 설정을 전달함으로써, 렌더러가 기본 카탈로그 컴포넌트뿐만 아니라 `CustomSlider` 컴포넌트와 `capitalize` 함수도 처리할 수 있게 된다.
