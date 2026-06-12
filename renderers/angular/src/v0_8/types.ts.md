# renderers/angular/src/v0_8/types.ts

## 개요

v0.8 Angular 렌더러가 사용하는 모든 타입을 `@a2ui/web_core/v0_8` 패키지로부터 재내보내거나 래핑하여 정의한다. Angular 렌더러 레이어가 web core 패키지에 직접 의존하지 않고 이 파일을 통해 타입을 참조하도록 하는 단일 타입 진입점 역할을 한다. 대부분은 `WebCore.*` 타입의 alias이며, 일부는 Angular 렌더러에 특화된 자체 정의(`Component<P>`, `SurfaceID`, `FunctionCall`)를 포함한다.

## 의존성

### 외부 패키지

- `@a2ui/web_core/v0_8` — `WebCore` 네임스페이스로 임포트

### 저장소 내부 모듈

없음

## Exports

### 메시지 및 인프라 타입

| 이름 | 정의 |
|---|---|
| `Action` | `WebCore.Action` alias |
| `FunctionCall` | `unknown` (미사용 플레이스홀더) |
| `SurfaceID` | `string` |
| `StringValue` | `WebCore.StringValue` alias |
| `BooleanValue` | `WebCore.BooleanValue` alias |
| `NumberValue` | `WebCore.NumberValue` alias |
| `Surface` | `WebCore.Surface` alias |
| `A2UIClientEventMessage` | `WebCore.A2UIClientEventMessage` alias |
| `ClientToServerMessage` | `A2UIClientEventMessage` alias (동의어) |
| `ServerToClientMessage` | `WebCore.ServerToClientMessage` alias |

### 컴포넌트 및 인터페이스 타입

| 이름 | 정의 |
|---|---|
| `Component<P>` | 로컬 인터페이스 (아래 상세 참조) |
| `AnyComponentNode` | `WebCore.AnyComponentNode` alias |
| `CustomNode` | `WebCore.CustomNode` alias |
| `Theme` | `WebCore.Theme` alias |

### 노드 타입 (Explicit suffix)

`RowNode`, `ColumnNode`, `TextNode`, `ListNode`, `ImageNode`, `IconNode`, `VideoNode`, `AudioPlayerNode`, `ButtonNode`, `DividerNode`, `MultipleChoiceNode`, `TextFieldNode`, `CheckboxNode`, `SliderNode`, `DateTimeInputNode`, `TabsNode`, `ModalNode`, `CardNode` — 각각 대응하는 `WebCore.*Node` alias.

### Resolved Property 타입

`ResolvedRow`, `ResolvedColumn`, `ResolvedText`, `ResolvedList`, `ResolvedImage`, `ResolvedIcon`, `ResolvedVideo`, `ResolvedAudioPlayer`, `ResolvedButton`, `ResolvedDivider`, `ResolvedMultipleChoice`, `ResolvedTextField`, `ResolvedCheckbox`, `ResolvedSlider`, `ResolvedDateTimeInput`, `ResolvedTabs`, `ResolvedModal`, `ResolvedCard` — 각각 대응하는 `WebCore.Resolved*` alias.

### 마크다운 타입

| 이름 | 정의 |
|---|---|
| `MarkdownRenderer` | `WebCore.MarkdownRenderer` alias |
| `MarkdownRendererOptions` | `WebCore.MarkdownRendererOptions` alias |

## 상세 명세

### `Component<P = Record<string, unknown>>` 인터페이스

로컬 정의 인터페이스:

```
interface Component<P = Record<string, unknown>> {
  id: string;
  type: string;
  properties: P;
}
```

- `id: string` — 컴포넌트 고유 식별자.
- `type: string` — 카탈로그에서 조회할 컴포넌트 타입 이름.
- `properties: P` — 컴포넌트별 속성 객체. 제네릭 기본값은 `Record<string, unknown>`.
- `web_core`의 `AnyComponentNode`와 구조적으로 유사하지만, Angular 렌더러에서 단순한 타입 참조용으로 별도 정의되었다.

### `SurfaceID`

`type SurfaceID = string` — 서피스를 식별하는 단순 문자열 타입 alias. `DynamicComponent`와 `Renderer`에서 사용된다.

### `FunctionCall`

`type FunctionCall = unknown` — 현재 미사용이며 향후 확장을 위한 플레이스홀더다.

### `ClientToServerMessage`

`type ClientToServerMessage = A2UIClientEventMessage` — `A2UIClientEventMessage`의 동의어 alias. API 명명 일관성을 위해 제공된다.

## 동작 흐름

이 파일은 타입 선언만 포함하며 런타임 코드가 없다. TypeScript 컴파일 시 `@a2ui/web_core/v0_8`의 타입들이 이 파일을 통해 재내보내지며, 렌더러 내부 파일들은 `../types`에서 임포트하여 web core에 대한 직접 의존을 피한다.
