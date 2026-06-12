# renderers/angular/src/v0_9/catalog/basic/video.component.ts

## 개요

A2UI v0.9 Video 컴포넌트의 Angular 구현체다. `BasicCatalogComponent<typeof VideoApi>`를 상속하여 A2UI 프로토콜에서 전달된 `url` 프로퍼티를 reactive signal로 읽어 `<video>` 엘리먼트의 `src`에 바인딩한다. 표준 브라우저 비디오 플레이어(`controls` 속성 포함)를 렌더링하며 CSS 변수(`--a2ui-video-border-radius`)로 스타일을 커스터마이즈할 수 있다.

## 의존성

### 외부 패키지
- `@angular/core`: `Component`, `computed`, `ChangeDetectionStrategy`
- `@a2ui/web_core/v0_9/basic_catalog`: `VideoApi`

### 저장소 내부 모듈
- [`./basic-catalog-component`](./basic-catalog-component.ts.md)

## Exports

| 이름 | 종류 |
|------|------|
| `VideoComponent` | Angular 컴포넌트 클래스 |

## 상세 명세

### `VideoComponent`

**데코레이터**: `@Component`  
**selector**: `'a2ui-v09-video'`  
**standalone**: `true`  
**imports**: `[]`  
**changeDetection**: `ChangeDetectionStrategy.OnPush`

**상속**: `BasicCatalogComponent<typeof VideoApi>` — props 입력 signal, surfaceId, componentId, dataContextPath를 부모로부터 상속받는다.

#### 필드

| 이름 | 타입 | 설명 |
|------|------|------|
| `url` | `Signal<string \| undefined>` (readonly) | `props()['url']?.value()`를 `computed`로 파생한 signal. `url` prop이 없을 경우 `undefined`를 반환한다. |

#### 템플릿 구조

최상위 `div.a2ui-video-container` 안에 `video.a2ui-video` 엘리먼트가 위치한다. `[attr.src]="url() || null"` 바인딩을 사용하여 url이 falsy이면 `src` 어트리뷰트를 제거한다. `controls` 어트리뷰트로 브라우저 기본 컨트롤을 활성화한다. 브라우저가 video 태그를 지원하지 않으면 `"Your browser does not support the video tag."` 텍스트가 표시된다.

#### 스타일

- `.a2ui-video-container`: `width: 100%; max-width: 100%`
- `.a2ui-video`: `width: 100%; height: auto; display: block; border-radius: var(--a2ui-video-border-radius, 0)`

CSS 변수 `--a2ui-video-border-radius`의 기본값은 `0`이다.

## 동작 흐름

1. 부모 클래스(`BasicCatalogComponent`)로부터 `props()` signal을 제공받는다.
2. `url` computed signal이 `props()['url']?.value()`를 평가한다. `url` prop이 없거나 undefined이면 `undefined`를 반환한다.
3. 템플릿은 `url()` 값이 truthy이면 `<video src="...">`, falsy이면 `<video>` (src 어트리뷰트 없음)를 렌더링한다.
4. `ChangeDetectionStrategy.OnPush`로 인해 signal 값 변경 시에만 변경 감지가 발생한다.
