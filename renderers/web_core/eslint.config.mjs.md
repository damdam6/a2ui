# renderers/web_core/eslint.config.mjs

## 개요

`web_core` 패키지의 ESLint 설정 진입점 파일이다. 상위 디렉토리에 위치한 공유 프리셋(`eslint.preset.mjs`)을 그대로 재내보내기(re-export)하는 역할만 수행한다. 패키지별로 ESLint 설정 파일이 필요하지만 커스터마이징 없이 공통 규칙을 따를 때 사용하는 최소 설정 패턴이다.

## 의존성

### 저장소 내부 모듈
- [`../../eslint.preset.mjs`](../../eslint.preset.mjs.md) — 스프레드(`...preset`)로 전개되어 사용되는 공유 ESLint 프리셋 배열

### 외부 패키지
없음

## Exports

- `default` (배열) — `preset` 배열을 스프레드한 새 배열(`[...preset]`)

## 상세 명세

### default export
- **값:** `[...preset]` — `eslint.preset.mjs`의 기본 내보내기 배열을 스프레드하여 새 배열로 반환
- **동작:** `preset`을 직접 재내보내지 않고 스프레드함으로써 호출 측이 배열을 변경해도 원본 프리셋에 영향을 주지 않는 방어적 복사를 수행한다.

## 동작 흐름

파일 로드 시 `eslint.preset.mjs`를 import하고, 그 값을 스프레드한 새 배열을 `default`로 내보낸다. ESLint가 이 파일을 flat config로 읽으면 프리셋에 정의된 규칙 집합이 그대로 적용된다.
