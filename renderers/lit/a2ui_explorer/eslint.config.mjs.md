# renderers/lit/a2ui_explorer/eslint.config.mjs

## 개요

`a2ui_explorer` 패키지의 ESLint 설정 파일이다. 저장소 루트에 위치한 공유 ESLint 프리셋을 불러와 그대로 재사용하며, 추가적인 규칙이나 오버라이드 없이 프리셋의 배열을 전개(spread)하여 기본 export로 내보낸다. 파일 자체의 로직은 단 한 줄로 구성된 위임 구조다.

## 의존성

### 저장소 내부 모듈
- [`../../../eslint.preset.mjs`](../../../eslint.preset.mjs.md) — 저장소 공통 ESLint 규칙 프리셋 (배열 형태)

### 외부 패키지
없음.

## Exports

- `default` (배열) — `preset` 배열을 전개한 flat config 배열. ESLint flat config 형식(`eslint.config.mjs`)에서 사용된다.

## 상세 명세

### default export

`[...preset]` — `preset` 배열의 모든 요소를 새 배열에 전개하여 내보낸다. 이 방식은 호출자가 나중에 항목을 추가할 수 있도록 배열을 복사하는 패턴이기도 하지만, 이 파일에서는 별도 추가 없이 원본 프리셋과 동일한 규칙 집합을 그대로 사용한다.

## 동작 흐름

파일이 로드되면 공유 프리셋을 import하고, 해당 배열을 spread하여 기본값으로 export한다. ESLint는 이 export를 `a2ui_explorer` 디렉토리에 대한 flat config 설정으로 인식한다.
