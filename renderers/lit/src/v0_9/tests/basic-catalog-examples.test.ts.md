# renderers/lit/src/v0_9/tests/basic-catalog-examples.test.ts

## 개요

저장소 내 `specification/v0_9/catalogs/basic/examples` 디렉토리에 있는 모든 JSON 예제 파일을 자동으로 탐색하여, 각 파일이 `basicCatalog`와 `MessageProcessor`로 오류 없이 처리되는지 검증하는 데이터 기반(data-driven) 통합 테스트다. 특정 파일명(`capitalized_text`)에 대해서는 데이터 바인딩 및 reactive pipeline 동작도 추가로 검증한다.

## 의존성

### 외부 패키지
- `node:assert` — `assert`
- `node:test` — `describe`, `it`
- `@a2ui/web_core/v0_9` — `MessageProcessor`, `GenericBinder`, `ComponentContext`
- `@a2ui/web_core/v0_9/basic_catalog` — `TextApi`
- `fs` (Node.js 내장) — `readdirSync`, `readFileSync`
- `path` (Node.js 내장) — `resolve`, `join`

### 저장소 내부 모듈
- `../catalogs/basic/index.js` — [`basicCatalog`](../catalogs/basic/index.js) (정적 import)

## Exports

없음 (테스트 파일)

## 테스트 케이스

### describe: `v0.9 Basic Catalog Examples`

**픽스처 / 모킹**
- 테스트 파일들은 `before`/`after` 훅 없이 모듈 초기화 시점에 탐색된다. DOM 설정이 불필요한 순수 데이터 처리 테스트이므로 JSDOM을 사용하지 않는다.
- `examplesDir`: `process.cwd()`로부터 `../../specification/v0_9/catalogs/basic/examples`를 `path.resolve`로 결정한다.
- `files`: `fs.readdirSync(examplesDir)`로 디렉토리를 읽고 `.json`으로 끝나는 파일만 필터링한다.
- 각 파일에 대해 `it` 테스트가 동적으로 생성된다. 테스트 이름은 `` `should successfully process ${file}` `` 형식이다.

---

#### 동적 생성 테스트: `should successfully process <파일명>.json` (파일별 반복)

- 검증 동작:
  1. 파일을 `readFileSync`로 읽고 `JSON.parse`한다.
  2. 데이터가 배열이면 그대로, 아니면 `data.messages || []`를 `messages` 배열로 사용한다.
  3. `surfaceId`를 파일명(`.json` 제거)으로 초기화한 뒤, `messages`에서 `createSurface`를 가진 메시지를 탐색한다.
     - 발견 시: `createMsg.createSurface.surfaceId`를 실제 `surfaceId`로 사용한다.
     - 미발견 시: `createSurface: { surfaceId, catalogId: basicCatalog.id }` 메시지를 `messages` 앞에 삽입(`unshift`)한다.
  4. `new MessageProcessor([basicCatalog])`를 생성하고 `processor.processMessages(messages)`를 호출한다.
  5. `processor.model.getSurface(surfaceId)`가 truthy임을 검증한다.
  6. `surface.componentsModel.get('root')`가 truthy임을 검증한다(루트 컴포넌트 존재 확인).
  7. 파일명에 `'capitalized_text'`가 포함된 경우 추가 검증을 수행한다:
     a. `surface.componentsModel.get('result_text')`가 존재하는지 확인한다.
     b. `new ComponentContext(surface, 'result_text')`와 `new GenericBinder<any>(context, TextApi.schema)`를 생성한다.
     c. `binder.subscribe(() => {})`를 호출하여 reactive pipeline을 활성화한다.
     d. `await new Promise(r => setTimeout(r, 0))`으로 초기 해석이 완료되기를 기다린다.
     e. `binder.snapshot.text`가 `''`(빈 문자열)임을 검증한다.
     f. `surface.dataModel.set('/inputValue', 'hello world')`로 데이터를 설정한다.
     g. 다시 한 마이크로태스크를 기다린 뒤 `binder.snapshot.text`가 `'Hello world'`임을 검증한다(첫 글자 대문자 변환 확인).
     h. `sub.unsubscribe()`와 `binder.dispose()`로 정리한다.

## 동작 흐름

모듈 로드 시 파일 시스템에서 예제 JSON 파일 목록을 수집하고, 파일마다 동적으로 테스트 케이스를 등록한다. 각 테스트는 독립적인 `MessageProcessor` 인스턴스를 생성하여 파일의 메시지를 처리하고, 서피스와 루트 컴포넌트의 존재를 검증한다. `capitalized_text` 파일에 대해서는 데이터 모델 변경 시 reactive 파이프라인이 올바르게 텍스트를 변환하는지까지 검증한다.
