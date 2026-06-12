# renderers/angular/src/v0_9/public-api.ts

## 개요

A2UI Angular 렌더러 v0.9의 공개 API 진입점 파일이다. 코어 서비스, 컴포넌트, 카탈로그, 그리고 기본 제공 UI 컴포넌트들을 하나의 모듈에서 일괄 재내보내어 소비자가 단일 임포트 경로로 필요한 모든 심볼에 접근할 수 있도록 한다.

## 의존성

### 외부 패키지

없음.

### 저장소 내부 모듈

| 내보내는 파일 | 링크 |
|--------------|------|
| `./core/a2ui-renderer.service` | [`./core/a2ui-renderer.service`](./core/a2ui-renderer.service.ts.md) |
| `./core/component-host.component` | [`./core/component-host.component`](./core/component-host.component.ts.md) |
| `./core/surface.component` | [`./core/surface.component`](./core/surface.component.ts.md) |
| `./core/catalog_component` | [`./core/catalog_component`](./core/catalog_component.ts.md) |
| `./core/component-binder.service` | [`./core/component-binder.service`](./core/component-binder.service.ts.md) |
| `./core/types` | [`./core/types`](./core/types.ts.md) |
| `./core/utils` | [`./core/utils`](./core/utils.ts.md) |
| `./core/markdown` | [`./core/markdown`](./core/markdown.ts.md) |
| `./catalog/types` | [`./catalog/types`](./catalog/types.ts.md) |
| `./catalog/basic/basic-catalog` | [`./catalog/basic/basic-catalog`](./catalog/basic/basic-catalog.ts.md) |
| `./catalog/basic/text.component` | — |
| `./catalog/basic/row.component` | — |
| `./catalog/basic/column.component` | — |
| `./catalog/basic/button.component` | — |
| `./catalog/basic/text-field.component` | — |
| `./catalog/basic/image.component` | — |
| `./catalog/basic/icon.component` | — |
| `./catalog/basic/video.component` | — |
| `./catalog/basic/audio-player.component` | — |
| `./catalog/basic/list.component` | — |
| `./catalog/basic/card.component` | — |
| `./catalog/basic/tabs.component` | — |
| `./catalog/basic/modal.component` | — |
| `./catalog/basic/divider.component` | — |
| `./catalog/basic/check-box.component` | — |
| `./catalog/basic/choice-picker.component` | — |
| `./catalog/basic/slider.component` | — |
| `./catalog/basic/date-time-input.component` | — |

## Exports

모든 항목은 `export * from '...'` 형태로 참조 모듈의 모든 공개 심볼을 그대로 재내보낸다. 구체적으로 재내보내는 주요 심볼들:

**코어**:
- `A2uiRendererService`, `A2UI_RENDERER_CONFIG` (renderer service)
- `ComponentHostComponent` (component host)
- `SurfaceComponent` (surface component)
- `ComponentBinder`, `Child` (binder service)
- `ComponentTemplate`, `BoundProperty`, `ExtendedProps`, `ComponentApiToProps` (types)
- `toAngularSignal`, `getNormalizedPath`, `preactSignal` (utils)
- `MarkdownRenderer`, `DefaultMarkdownRenderer`, `MarkdownRendererOptions`, `provideMarkdownRenderer` (markdown)

**카탈로그**:
- `BasicCatalog`, `BasicCatalogBase` (basic catalog)
- Basic 카탈로그의 모든 UI 컴포넌트 클래스 및 API 타입 (Text, Row, Column, Button, TextField, Image, Icon, Video, AudioPlayer, List, Card, Tabs, Modal, Divider, CheckBox, ChoicePicker, Slider, DateTimeInput)

## 동작 흐름

이 파일은 순수 배럴(barrel) 모듈로 런타임 로직이 없다. v0.9 라이브러리 소비자는 이 파일 하나를 임포트하여 Angular 렌더러의 모든 공개 API에 접근한다. Angular 라이브러리 빌드 시스템(`ng-packagr`)이 이 파일을 진입점으로 사용한다.
