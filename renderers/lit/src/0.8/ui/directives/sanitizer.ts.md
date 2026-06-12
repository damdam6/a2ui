# renderers/lit/src/0.8/ui/directives/sanitizer.ts

## 개요

브라우저 DOM을 이용한 안전한 HTML 텍스트 이스케이프/언이스케이프 유틸리티 함수 두 개를 제공한다. `escapeNodeText`는 텍스트 노드 위치에서만 안전하게 사용할 수 있으며, `unescapeNodeText`는 HTML 엔티티 문자열을 원래 텍스트로 복원한다. Lit의 `render`와 표준 DOM API를 활용해 XSS 위험 없이 처리한다.

## 의존성

### 외부 패키지
- `lit` — `html`, `render`

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `escapeNodeText` | 함수 | 문자열을 HTML 텍스트 노드 안전 형태로 이스케이프 |
| `unescapeNodeText` | 함수 | HTML 엔티티가 포함된 문자열을 원본 텍스트로 디코드 |

## 상세 명세

### `escapeNodeText(str: string | null | undefined): string`

**목적:** 텍스트 노드 위치 삽입에만 안전하다. 속성 위치에서 사용할 경우 속성값이 이중 따옴표로 둘러싸인 경우에만 안전하고 그 외에는 위험하다는 주의사항이 명시되어 있다.

**동작:**
1. `document.createElement('div')`로 임시 컨테이너 `frag`를 생성한다.
2. Lit의 `render(html\`${str}\`, frag)`를 호출해 `str`을 Lit이 안전하게 텍스트로 처리하도록 한다. Lit은 내부적으로 텍스트를 이스케이프하므로 XSS가 방지된다.
3. `frag.innerHTML`에서 `<!--...-->` 형태의 HTML 주석을 정규식 `/<!--([^-]*)-->/gim`으로 모두 제거한 결과 문자열을 반환한다. 주석 제거는 Lit이 렌더링 마커로 삽입하는 주석 노드를 정리하기 위함이다.

**반환 타입:** `string`

---

### `unescapeNodeText(str: string | null | undefined): string`

**동작:**
1. `str`이 falsy(null, undefined, 빈 문자열 등)이면 즉시 빈 문자열 `''`을 반환한다.
2. `document.createElement('textarea')`로 임시 엘리먼트를 생성한다.
3. `frag.innerHTML = str`로 브라우저의 HTML 파서가 엔티티를 해석하게 한다.
4. `frag.value`를 반환한다. `textarea`의 `.value`는 파싱된 텍스트 내용이므로 HTML 엔티티가 원래 문자로 변환된 값이다.

**반환 타입:** `string`

## 동작 흐름

두 함수는 서로 독립적인 유틸리티다. `escapeNodeText`는 Lit 렌더 파이프라인을 활용해 입력 문자열을 이스케이프하고, `unescapeNodeText`는 `textarea` DOM 트릭으로 HTML 엔티티를 디코드한다. 두 함수 모두 실제 DOM 조작을 통해 처리하므로 브라우저 환경에서만 동작한다.
