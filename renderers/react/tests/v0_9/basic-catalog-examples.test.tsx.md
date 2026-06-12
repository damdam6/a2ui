# renderers/react/tests/v0_9/basic-catalog-examples.test.tsx

## 개요

v0.9 basic catalog의 JSON 예제 파일을 파일시스템에서 동적으로 읽어, 각 파일을 실제 `MessageProcessor`와 `A2uiSurface`로 렌더링하여 오류 없이 완료되는지 검증하는 스모크 테스트 파일이다. 예제 디렉토리의 모든 `.json` 파일에 대해 자동으로 테스트 케이스가 생성된다.

## 의존성

### 외부 패키지
- `vitest`: `describe`, `it`, `expect`
- `@testing-library/react`: `render`
- `react`: `React`, `React.StrictMode`
- `@a2ui/web_core/v0_9`: `MessageProcessor`
- `@a2ui/react/v0_9`: `A2uiSurface`, `basicCatalog`
- `fs`: Node.js 파일 시스템 모듈 (`/// <reference types="node" />` 필요)
- `path`: Node.js 경로 모듈

### 저장소 내부 모듈
없음 (직접적인 내부 모듈 import 없음; `@a2ui/react/v0_9`는 패키지 alias).

## Exports

없음 (테스트 파일).

## 테스트 케이스 명세

### 설정 (describe 블록 내 setup 로직)

1. `examplesDir` 경로를 `process.cwd()`로부터 `../../specification/v0_9/catalogs/basic/examples`로 resolve한다.
2. 해당 디렉토리가 존재하지 않으면 즉시 `throw new Error(...)` (테스트 수집 단계에서 실패).
3. `fs.readdirSync(examplesDir)`로 `.json`으로 끝나는 파일 목록을 가져온다.

### 동적 생성 테스트: `it('should successfully render ${file}')`

각 `.json` 파일마다 하나의 테스트 케이스가 생성된다.

**과정**:
1. `fs.readFileSync`로 파일 내용을 읽고 `JSON.parse`한다.
2. `messages` 배열 추출: 파싱 결과가 배열이면 그대로, 객체이면 `.messages` 프로퍼티를 사용, 없으면 빈 배열.
3. `surfaceId` 결정:
   - 기본값: 파일명에서 `.json` 제거한 문자열.
   - `messages` 중 `createSurface` 키를 가진 메시지가 있으면 해당 `surfaceId` 사용.
   - 없으면 `messages` 배열 맨 앞에 다음 메시지를 `unshift`로 추가:
     ```
     { version: 'v0.9', createSurface: { surfaceId, catalogId: basicCatalog.id } }
     ```
4. `new MessageProcessor([basicCatalog as any])`로 프로세서 생성 후 `processor.processMessages(messages)` 실행.
5. `processor.model.getSurface(surfaceId)` 호출 → surface가 `undefined`가 아님을 확인.
6. `<React.StrictMode><A2uiSurface surface={surface as any} /></React.StrictMode>` 렌더링.
7. `container.firstChild`가 truthy인지 확인 (렌더 결과가 비어있지 않음).

## 동작 흐름

파일 시스템에서 동적으로 테스트를 생성하는 데이터 기반 테스트 패턴이다. describe 블록이 평가될 때 예제 파일 목록이 결정되고, 각 파일에 대해 `it` 블록이 등록된다. `createSurface` 메시지 유무에 따라 messages 배열을 보정한 뒤, `MessageProcessor`로 메시지를 처리하고 `A2uiSurface`로 렌더링하여 어떠한 예외도 발생하지 않고 DOM 노드가 생성되었는지를 최소 검증한다.
