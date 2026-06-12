# renderers/react/visual-parity/fixtures/components/index.ts

## 개요

`components/` 디렉토리 내 모든 컴포넌트 픽스처 모듈을 단일 진입점으로 집계하는 배럴(barrel) 파일이다. 각 컴포넌트 픽스처 파일의 모든 named export를 재-export하고, 각 파일의 `*Fixtures` 그룹 객체도 명시적으로 재-export한다. 외부 소비자는 이 파일 하나에서 모든 컴포넌트 픽스처와 픽스처 그룹에 접근할 수 있다.

## 의존성

### 저장소 내부 모듈

| 경로 | 설명 |
|---|---|
| [`./text`](./text.ts.md) | Text 컴포넌트 픽스처 |
| `./button` | Button 컴포넌트 픽스처 |
| `./icon` | Icon 컴포넌트 픽스처 |
| [`./image`](./image.ts.md) | Image 컴포넌트 픽스처 |
| `./divider` | Divider 컴포넌트 픽스처 |
| `./card` | Card 컴포넌트 픽스처 |
| [`./row`](./row.ts.md) | Row 컴포넌트 픽스처 |
| `./column` | Column 컴포넌트 픽스처 |
| [`./list`](./list.ts.md) | List 컴포넌트 픽스처 |
| [`./tabs`](./tabs.ts.md) | Tabs 컴포넌트 픽스처 |
| `./checkbox` | CheckBox 컴포넌트 픽스처 |
| [`./textField`](./textField.ts.md) | TextField 컴포넌트 픽스처 |
| [`./slider`](./slider.ts.md) | Slider 컴포넌트 픽스처 |
| `./dateTimeInput` | DateTimeInput 컴포넌트 픽스처 |
| [`./multipleChoice`](./multipleChoice.ts.md) | MultipleChoice 컴포넌트 픽스처 |
| [`./video`](./video.ts.md) | Video 컴포넌트 픽스처 |
| `./audioPlayer` | AudioPlayer 컴포넌트 픽스처 |
| [`./modal`](./modal.ts.md) | Modal 컴포넌트 픽스처 |

### 외부 패키지

없음

## Exports

두 단계의 re-export로 구성된다.

**1단계 — 와일드카드 re-export (`export * from`)**

아래 모듈의 모든 named export를 그대로 노출한다: `./text`, `./button`, `./icon`, `./image`, `./divider`, `./card`, `./row`, `./column`, `./list`, `./tabs`, `./checkbox`, `./textField`, `./slider`, `./dateTimeInput`, `./video`, `./audioPlayer`, `./modal`

`./multipleChoice`는 와일드카드 대신 `{multipleChoice}` 이름 지정 export 방식으로만 노출된다 — `export {multipleChoice} from './multipleChoice'`

**2단계 — 픽스처 그룹 명시적 re-export**

각 컴포넌트 파일의 집계 객체를 이름 지정 방식으로 재-export한다:

| export 이름 | 출처 |
|---|---|
| `textFixtures` | `./text` |
| `buttonFixtures` | `./button` |
| `iconFixtures` | `./icon` |
| `imageFixtures` | `./image` |
| `dividerFixtures` | `./divider` |
| `cardFixtures` | `./card` |
| `rowFixtures` | `./row` |
| `columnFixtures` | `./column` |
| `listFixtures` | `./list` |
| `tabsFixtures` | `./tabs` |
| `checkboxFixtures` | `./checkbox` |
| `textFieldFixtures` | `./textField` |
| `sliderFixtures` | `./slider` |
| `dateTimeInputFixtures` | `./dateTimeInput` |
| `multipleChoiceFixtures` | `./multipleChoice` |
| `videoFixtures` | `./video` |
| `audioPlayerFixtures` | `./audioPlayer` |
| `modalFixtures` | `./modal` |

## 동작 흐름

이 파일은 런타임 로직 없이 순수하게 re-export만 수행한다. `multipleChoice`가 와일드카드 방식이 아닌 명시적 named export로 처리되는 것이 유일한 특이 사항으로, 이는 해당 모듈의 export 구조상 이름 충돌 또는 의도적인 노출 제한 때문으로 보인다.
