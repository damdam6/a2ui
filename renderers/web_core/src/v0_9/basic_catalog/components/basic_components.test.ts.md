# renderers/web_core/src/v0_9/basic_catalog/components/basic_components.test.ts

## 개요

`basic_components.ts`에서 내보내는 컴포넌트 API 스키마(현재는 `ImageApi`)의 Zod 파싱 동작을 검증하는 단위 테스트 파일이다. Node.js 내장 테스트 러너를 사용하며 외부 테스트 프레임워크에 의존하지 않는다.

## 의존성

- 외부 패키지: `node:test` (`describe`, `it`), `node:assert`
- 저장소 내부 모듈:
  - [`./basic_components.ts`](./basic_components.ts.md) — `ImageApi` 임포트

## Exports

없음 (테스트 파일이므로 공개 API를 내보내지 않음)

## 테스트 케이스 명세

### `describe('Basic Components Schema')`

#### `describe('ImageApi')`

`ImageApi.schema`(Zod 스키마)의 `parse` 메서드를 통한 유효성 검사 동작을 검증한다.

| 테스트명 | 검증 동작 | 입력 픽스처 |
|---|---|---|
| `'should parse valid image with description'` | `url`과 `description` 필드가 모두 있는 객체를 파싱했을 때, 결과의 `url`이 `'https://example.com/image.png'`이고 `description`이 `'An example image'`임을 확인 | `{ url: 'https://example.com/image.png', description: 'An example image' }` |
| `'should parse valid image without description'` | `url`만 있고 `description`이 없는 객체를 파싱했을 때, `url`이 올바르게 파싱되고 `description`이 `undefined`임을 확인 | `{ url: 'https://example.com/image.png' }` |
| `'should throw on invalid image'` | `url`에 숫자(`123`)를 전달하면 `ImageApi.schema.parse`가 예외를 던짐을 확인 | `{ url: 123 }` (잘못된 타입) |

**검증 방법:**
- 성공 케이스: `assert.strictEqual`로 개별 필드 값을 확인한다.
- 실패 케이스: `assert.throws`로 파싱 예외 발생을 확인한다.

## 동작 흐름

Node.js 테스트 러너가 파일을 실행하면 `describe` 블록이 등록되고 세 개의 `it` 콜백이 순차적으로 실행된다. 모든 검증은 동기적이며, `ImageApi.schema.parse`가 Zod 스키마 유효성 검사를 수행한다. 유효하지 않은 입력에 대해서는 Zod가 `ZodError`를 던지며, 이를 `assert.throws`가 캐치하여 테스트를 통과시킨다.
