# renderers/angular/a2ui_explorer/src/app/tests/utils/test_utils.ts

## 개요

Angular 테스트 환경에서 A2UI 예제를 로드하고 렌더러가 안정화될 때까지 대기하는 헬퍼 함수들을 제공하는 모듈이다. Jasmine/TestBed 기반의 통합 테스트에서 `DemoComponent`를 설정·초기화하고, zoneless 비동기 렌더링 완료를 기다리며, DOM 요소를 조회하는 공통 유틸리티를 모은다.

## 의존성

### 외부 패키지
- `@angular/core/testing`: `TestBed`

### 저장소 내부 모듈
- [`../../demo.component`](../../demo.component.ts.md) — `DemoComponent`
- `../../generated/examples-bundle` (빌드 타임 생성, 소스 저장소에 없음) — `EXAMPLES_V08`, `EXAMPLES_V09`
- `../../../../../src/v0_9/core/markdown` — `provideMarkdownRenderer`
- [`../../types`](../../types.ts.md) — `A2UI_VERSION`, `Version`

## Exports

- `Version` (열거형, `../../types`에서 재내보내기)
- `loadExample` (비동기 함수)
- `wait` (함수)
- `getCanvas` (함수)
- `waitForCondition` (비동기 함수)

## 상세 명세

### `loadExample(exampleName: string, version: Version = Version.V0_9): Promise<ComponentFixture<DemoComponent>>`

TestBed를 구성해 `DemoComponent`를 마운트하고 지정된 예제를 선택한 뒤 렌더러가 안정화되면 fixture를 반환하는 비동기 헬퍼다.

**동작 단계**:
1. `TestBed.configureTestingModule({ imports: [DemoComponent], providers: [provideMarkdownRenderer(), { provide: A2UI_VERSION, useValue: version }] })`를 호출한다.
2. `TestBed.createComponent(DemoComponent)`로 fixture를 만들고 `fixture.detectChanges()`를 호출한다.
3. `version`에 따라 `EXAMPLES_V09` 또는 `EXAMPLES_V08` 배열에서 `ex.name === exampleName`을 기준으로 예제를 탐색한다.
4. `version === Version.V0_8`이고 정확한 이름 매칭에 실패한 경우: `'${exampleName} (basic)'`으로 재탐색하고, 그래도 없으면 `'${exampleName} (minimal)'`을 시도한다.
5. `expect(example).withContext('Example not found: ' + exampleName).toBeTruthy()`로 예제 존재 여부를 검증한다. 예제가 없으면 테스트가 이 시점에서 실패한다.
6. `component.selectExample(example!)`으로 예제를 선택하고 `fixture.detectChanges()`를 다시 호출한다.
7. `await whenSettled()`로 렌더링이 완전히 안정화될 때까지 기다린다.
8. fixture를 반환한다.

### `whenSettled(): Promise<void>` (비공개 헬퍼, 내부 사용)

zoneless 애플리케이션에서는 `NgZone.onStable`을 사용할 수 없으므로, 매크로태스크 큐에 20회 양보해 대기 중인 모든 마이크로태스크(Preact signal 리스너, Angular 변경 감지 사이클 포함)가 처리되도록 한다.

`MACROTASKS_TO_FLUSH` 상수는 `20`. `for (let i = 0; i < MACROTASKS_TO_FLUSH; i++) { await wait(0); }` 패턴으로 구현된다.

### `wait(ms: number): Promise<void>`

**시그니처**: `(ms: number) => Promise<void>`

`new Promise(resolve => setTimeout(resolve, ms))`를 반환한다. `ms` 밀리초 후에 이행되는 Promise다. `whenSettled`와 각 spec의 버튼 클릭 후 비동기 대기에 사용된다.

### `getCanvas(): HTMLDivElement`

**시그니처**: `() => HTMLDivElement`

`document.querySelector('.canvas-frame')!`을 반환한다. 비-null assertion 연산자(`!`)를 사용하므로 `.canvas-frame` 요소가 DOM에 마운트된 상태에서만 호출해야 한다.

### `waitForCondition(condition: () => boolean, timeout = 1000, interval = 50): Promise<boolean>`

**시그니처**: `(condition: () => boolean, timeout?: number, interval?: number) => Promise<boolean>`

동기 조건 함수가 `true`를 반환할 때까지 반복 폴링하는 비동기 헬퍼다.

**동작**:
1. `performance.now()`로 시작 시각을 기록한다.
2. 무한 루프에서 `condition()`을 호출해 `true`이면 `true`를 반환하고 종료한다.
3. `performance.now() - start > timeout`이면 `false`를 반환하고 종료한다.
4. 위 두 조건 모두 미충족 시 `await wait(interval)`로 다음 폴링까지 기다린다.

Jasmine 호환성을 위해 루프 내부에서 `expect`를 던지지 않고 `boolean` 반환값으로 결과를 전달한다. 조건 함수는 동기 함수만 지원한다.

## 동작 흐름

각 spec 파일의 `beforeEach`에서 `loadExample`을 호출해 TestBed를 초기화하고 예제를 렌더링한다. 이후 `getCanvas()`로 `.canvas-frame` DOM을 가져와 텍스트 내용을 검사하거나, fixture를 통해 컴포넌트 상태와 버튼 클릭 이벤트를 검증한다. `wait(ms)`와 `waitForCondition`은 비동기 상호작용 후 렌더러 응답을 기다리는 데 사용된다.
