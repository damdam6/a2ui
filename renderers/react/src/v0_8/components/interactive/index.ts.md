# renderers/react/src/v0_8/components/interactive/index.ts

## 개요

interactive 컴포넌트 디렉토리의 barrel 파일로, 모든 인터랙티브 컴포넌트를 단일 진입점에서 재내보낸다. 외부 모듈에서 개별 파일 경로를 알 필요 없이 이 파일 하나만 import하면 모든 인터랙티브 컴포넌트에 접근할 수 있다.

## 의존성

### 저장소 내부 모듈
- `./Button` (Button.tsx): `Button` 컴포넌트
- [`./TextField`](./TextField.tsx.md): `TextField` 컴포넌트
- [`./CheckBox`](./CheckBox.tsx.md): `CheckBox` 컴포넌트
- [`./Slider`](./Slider.tsx.md): `Slider` 컴포넌트
- [`./DateTimeInput`](./DateTimeInput.tsx.md): `DateTimeInput` 컴포넌트
- [`./MultipleChoice`](./MultipleChoice.tsx.md): `MultipleChoice` 컴포넌트

## Exports

- `Button` — `./Button`에서 named re-export
- `TextField` — `./TextField`에서 named re-export
- `CheckBox` — `./CheckBox`에서 named re-export
- `Slider` — `./Slider`에서 named re-export
- `DateTimeInput` — `./DateTimeInput`에서 named re-export
- `MultipleChoice` — `./MultipleChoice`에서 named re-export

## 동작 흐름

각 컴포넌트 파일에서 named export를 그대로 재내보내는 단순 집합 모듈이다. 별도 로직 없음. 각 `export { X } from './X'` 구문이 해당 컴포넌트 파일의 named export를 그대로 다시 내보낸다.
