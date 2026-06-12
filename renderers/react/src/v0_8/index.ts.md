# renderers/react/src/v0_8/index.ts

## 개요

`v0_8` 패키지의 공개 API 진입점(barrel) 파일이다. 코어 컴포넌트, 훅, 레지스트리, 테마, 유틸리티, 타입, 그리고 모든 기본 제공 컴포넌트(콘텐츠/레이아웃/인터랙티브)를 하나의 파일에서 re-export한다. 외부 소비자는 이 파일에서만 import하면 된다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`./core/A2UIProvider`](./core/A2UIProvider.tsx.md)
- [`./core/A2UIRenderer`](./core/A2UIRenderer.tsx.md)
- [`./core/A2UIViewer`](./core/A2UIViewer.tsx.md)
- [`./core/ComponentNode`](./core/ComponentNode.tsx.md)
- [`./hooks/useA2UI`](./hooks/useA2UI.ts.md)
- [`./hooks/useA2UIComponent`](./hooks/useA2UIComponent.ts.md)
- [`./registry/ComponentRegistry`](./registry/ComponentRegistry.ts.md)
- `./registry/defaultCatalog`
- `./theme/ThemeContext`
- `./theme/litTheme`
- [`./lib/utils`](./lib/utils.ts.md)
- `./types`
- `./components/content/Text`
- `./components/content/Image`
- `./components/content/Icon`
- `./components/content/Divider`
- `./components/content/Video`
- `./components/content/AudioPlayer`
- `./components/layout/Row`
- `./components/layout/Column`
- `./components/layout/List`
- `./components/layout/Card`
- `./components/layout/Tabs`
- `./components/layout/Modal`
- `./components/interactive/Button`
- `./components/interactive/TextField`
- `./components/interactive/CheckBox`
- `./components/interactive/Slider`
- `./components/interactive/DateTimeInput`
- `./components/interactive/MultipleChoice`

## Exports

### 코어 컴포넌트 및 훅

| 이름 | 종류 |
|------|------|
| `A2UIProvider` | React 컴포넌트 |
| `useA2UIActions` | 훅 |
| `useA2UIState` | 훅 |
| `useA2UIContext` | 훅 |
| `A2UIProviderProps` | 타입 |
| `A2UIRenderer` | React 컴포넌트 |
| `A2UIRendererProps` | 타입 |
| `A2UIViewer` | React 컴포넌트 |
| `A2UIViewerProps` | 타입 |
| `ComponentInstance` | 타입 |
| `A2UIActionEvent` | 타입 |
| `ComponentNode` | React 컴포넌트 |
| `useA2UI` | 훅 |
| `UseA2UIResult` | 타입 |
| `useA2UIComponent` | 훅 |
| `UseA2UIComponentResult` | 타입 |

### 레지스트리

| 이름 | 종류 |
|------|------|
| `ComponentRegistry` | 클래스 |
| `registerDefaultCatalog` | 함수 |
| `initializeDefaultCatalog` | 함수 |

### 테마

| 이름 | 종류 |
|------|------|
| `ThemeProvider` | React 컴포넌트 |
| `useTheme` | 훅 |
| `useThemeOptional` | 훅 |
| `litTheme` | 상수 (테마 객체) |
| `defaultTheme` | 상수 (테마 객체) |

### 유틸리티

| 이름 | 종류 |
|------|------|
| `cn` | 함수 |
| `classMapToString` | 함수 |
| `stylesToObject` | 함수 |

### 타입 재수출

`Types`, `Primitives`, `AnyComponentNode`, `Surface`, `SurfaceID`, `Theme`, `ServerToClientMessage`, `A2UIClientEventMessage`, `Action`, `DataValue`, `MessageProcessor`, `StringValue`, `NumberValue`, `BooleanValue`, `A2UIComponentProps`, `ComponentRegistration`, `ComponentLoader`, `OnActionCallback`, `A2UIProviderConfig`

### 콘텐츠 컴포넌트

`Text`, `Image`, `Icon`, `Divider`, `Video`, `AudioPlayer`

### 레이아웃 컴포넌트

`Row`, `Column`, `List`, `Card`, `Tabs`, `Modal`

### 인터랙티브 컴포넌트

`Button`, `TextField`, `CheckBox`, `Slider`, `DateTimeInput`, `MultipleChoice`

## 동작 흐름

이 파일은 로직 없이 re-export만 수행한다. 파일이 import되면 각 하위 모듈이 로드되고 공개 API가 단일 네임스페이스로 노출된다.
