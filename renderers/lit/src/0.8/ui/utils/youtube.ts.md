# renderers/lit/src/0.8/ui/utils/youtube.ts

## 개요

YouTube URL을 판별하고 embed 형식으로 변환하는 순수 유틸리티 함수 모음이다. embed URI, share URI(`youtu.be`), watch URI, Shorts URI 등 YouTube의 여러 URL 형식을 감지하고, embed URL(`https://www.youtube.com/embed/<id>`)로 정규화하거나 비디오 ID를 추출한다. 외부 의존성이 전혀 없는 독립적인 모듈이다.

## 의존성

### 외부 패키지

없음

### 저장소 내부 모듈

없음

## Exports

| 이름 | 종류 |
|---|---|
| `isEmbedUri` | 함수 |
| `isShareUri` | 함수 |
| `isWatchUri` | 함수 |
| `isShortsUri` | 함수 |
| `convertShareUriToEmbedUri` | 함수 |
| `convertWatchOrShortsUriToEmbedUri` | 함수 |
| `videoIdFromWatchOrShortsOrEmbedUri` | 함수 |
| `createWatchUriFromVideoId` | 함수 |

## 상세 명세

### 함수: `isEmbedUri`

- **시그니처:** `isEmbedUri(uri: string | null): boolean`
- **동작:** `uri`가 `null`이거나 falsy이면 즉시 `false`를 반환한다. 그 외에는 정규식 `/^https:\/\/www\.youtube\.com\/embed\//`로 테스트하여 그 결과를 반환한다.

### 함수: `isShareUri`

- **시그니처:** `isShareUri(uri: string | null): boolean`
- **동작:** `uri`가 `null`이거나 falsy이면 즉시 `false`를 반환한다. 그 외에는 정규식 `/^https:\/\/youtu\.be\//`로 테스트하여 결과를 반환한다.

### 함수: `isWatchUri`

- **시그니처:** `isWatchUri(uri: string): boolean`
- **동작:** 정규식 `/^https:\/\/www\.youtube\.com\/watch\?v=/`로 `uri`를 테스트하여 결과를 반환한다. `null` 가드 없음 — 호출 측에서 null/undefined 처리를 담당해야 한다.

### 함수: `isShortsUri`

- **시그니처:** `isShortsUri(uri: string): boolean`
- **동작:** 정규식 `/^https:\/\/www\.youtube\.com\/shorts\//`로 `uri`를 테스트하여 결과를 반환한다.

### 함수: `convertShareUriToEmbedUri`

- **시그니처:** `convertShareUriToEmbedUri(uri: string): string | null`
- **동작:**
  1. 정규식 `/^https:\/\/youtu\.be\/(.*?)(?:[&\\?]|$)/`로 `uri`를 매칭한다.
  2. 매칭 실패 시 `null`을 반환한다.
  3. 성공 시 캡처 그룹 1번의 값을 `embedId`로 사용해 `https://www.youtube.com/embed/${embedId}` 문자열을 반환한다.
- **경계 케이스:** `?` 또는 `&`로 시작하는 쿼리 파라미터는 ID 이후 위치하면 캡처에서 제외된다.

### 함수: `convertWatchOrShortsUriToEmbedUri`

- **시그니처:** `convertWatchOrShortsUriToEmbedUri(uri: string): string | null`
- **동작:**
  1. 정규식 `/^https:\/\/www\.youtube\.com\/(?:shorts\/|embed\/|watch\?v=)(.*?)(?:[&\\?]|$)/`로 `uri`를 매칭한다. `shorts/`, `embed/`, `watch?v=` 세 가지 형식을 비캡처 그룹으로 처리한다.
  2. 매칭 실패 시 `null`을 반환한다.
  3. 성공 시 캡처된 ID로 `https://www.youtube.com/embed/${embedId}` 문자열을 반환한다.

### 함수: `videoIdFromWatchOrShortsOrEmbedUri`

- **시그니처:** `videoIdFromWatchOrShortsOrEmbedUri(uri: string): string | null`
- **동작:** `convertWatchOrShortsUriToEmbedUri`와 동일한 정규식을 사용하지만, embed URL 전체 대신 캡처된 비디오 ID 문자열(`matches[1]`)만 반환한다. 매칭 실패 시 `null`.

### 함수: `createWatchUriFromVideoId`

- **시그니처:** `createWatchUriFromVideoId(id: string): string`
- **동작:** `https://www.youtube.com/watch?v=${id}` 형태의 watch URL을 템플릿 리터럴로 조합해 반환한다. 실패 경우 없음.

## 동작 흐름

소비자는 먼저 `is*Uri` 함수로 URL 형식을 판별한 뒤, `convert*` 또는 `videoIdFrom*` 함수로 embed URL이나 비디오 ID를 추출한다. 모든 함수는 순수 함수(side-effect 없음)이며 정규식 기반으로 동작한다. `convertShareUriToEmbedUri`와 `convertWatchOrShortsUriToEmbedUri`는 서로 다른 URL 형식을 처리하므로 용도에 맞게 선택하여 사용해야 한다.
