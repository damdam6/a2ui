# renderers/angular/src/v0_9/core/surface.component.ts

## 개요

A2UI 서피스 전체를 Angular 애플리케이션에 삽입하기 위한 고수준 진입점 컴포넌트이다. 서피스의 루트 컴포넌트(`id: 'root'`)를 렌더링하기 위해 내부적으로 `ComponentHostComponent`를 사용하며, `dataContextPath`의 기본값으로 `'/'`를 제공한다. 이 컴포넌트는 A2UI 서피스를 Angular 앱에 임베드하는 권장 방법이다.

## 의존성

### 외부 패키지
- `@angular/core` — `ChangeDetectionStrategy`, `Component`, `input`

### 저장소 내부 모듈
- [`./component-host.component`](./component-host.component.ts.md) — `ComponentHostComponent`

## Exports

- `SurfaceComponent` (클래스 / Angular 컴포넌트)

## 상세 명세

### `class SurfaceComponent`

**데코레이터 설정**

| 옵션 | 값 |
|------|----|
| `selector` | `'a2ui-v09-surface'` |
| `standalone` | `true` |
| `imports` | `[ComponentHostComponent]` |
| `host` | `{ style: 'display: contents;' }` |
| `changeDetection` | `ChangeDetectionStrategy.OnPush` |

`display: contents` 스타일을 호스트 요소에 적용하여 컴포넌트 자체는 레이아웃에 영향을 미치지 않는다.

**템플릿**

`<a2ui-v09-component-host>` 하나를 렌더링한다.
- `[componentKey]`에 `{ id: 'root', basePath: dataContextPath() }` 객체를 바인딩한다.
- `[surfaceId]`에 `surfaceId()` 신호 값을 바인딩한다.

**입력 신호 (Signal Input)**

| 이름 | 타입 | 기본값 | 설명 |
|------|------|--------|------|
| `surfaceId` | `string` (required) | — | 렌더링할 서피스의 고유 식별자. `input.required<string>()`으로 선언되어 반드시 제공해야 한다. |
| `dataContextPath` | `string` | `'/'` | 서피스의 데이터 모델 내에서 현재 상태를 나타내는 경로. 기본값은 루트 `'/'`이다. |

**상태 및 생명주기**

별도의 상태나 생명주기 훅 없음. 신호 입력의 변화는 `OnPush` 변경 감지 전략에 의해 처리된다.

## 동작 흐름

`SurfaceComponent`는 템플릿에서 두 입력 신호 값을 읽어 `ComponentHostComponent`에 전달한다. 외부에서 `surfaceId`를 필수로 제공해야 하며, `dataContextPath`는 선택적이다. 컴포넌트 자체는 `display: contents`로 투명한 래퍼 역할을 하며, 실제 렌더링 로직은 모두 `ComponentHostComponent`가 담당한다.
