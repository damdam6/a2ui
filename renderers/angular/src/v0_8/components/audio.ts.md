# renderers/angular/src/v0_8/components/audio.ts

## 개요

a2ui v0.8 명세의 `AudioPlayer` UI 컴포넌트를 구현하는 Angular 스탠드얼론 컴포넌트 파일이다. `DynamicComponent`를 상속하며, `url` 입력으로 받은 `StringValue`를 HTML `<audio>` 요소의 `src`에 바인딩하여 오디오를 재생한다. 테마의 `additionalStyles.AudioPlayer`를 오디오 요소에 직접 적용한다.

## 의존성

### 외부 패키지
- `@angular/core` — `ChangeDetectionStrategy`, `Component`, `computed`, `input`

### 저장소 내부 모듈
- [`../rendering/dynamic-component`](../rendering/dynamic-component.ts.md) — `DynamicComponent` 기반 클래스
- `../types` (타입 전용) — `AudioPlayerNode`, `StringValue`

## Exports

| 이름 | 종류 |
|------|------|
| `AudioPlayer` | 클래스 (Angular 컴포넌트) |

## 상세 명세

### 클래스: `AudioPlayer`
- 데코레이터: `@Component`
- 셀렉터: `a2ui-audio`
- `changeDetection`: `ChangeDetectionStrategy.OnPush`
- 상속: `DynamicComponent<AudioPlayerNode>`
- 스탠드얼론 컴포넌트 (별도 `imports` 없음)

#### 템플릿
`<audio controls [src]="resolvedUrl()" [style]="theme.additionalStyles?.AudioPlayer">` 단일 요소. `controls` 속성이 항상 활성화되어 기본 브라우저 UI를 표시한다.

#### 스타일
- `:host` — `display: flex`
- `audio` — `width: 100%`

#### 입력 (Signal input)

| 이름 | 타입 | 필수 여부 |
|------|------|-----------|
| `url` | `StringValue \| null` | 필수 (`input.required`) |

`url`은 부모(`DynamicComponent`)의 `surfaceId`, `component`, `weight` 외에 이 컴포넌트 전용으로 추가된 유일한 입력이다.

#### 계산 신호

| 이름 | 접근 제어 | 설명 |
|------|-----------|------|
| `resolvedUrl` | `protected readonly` | `this.resolvePrimitive(this.url())`를 호출하여 `StringValue`를 실제 문자열로 해석한다. `url()`이 `null`이거나 해석 불가능하면 `null`을 반환한다. |

## 동작 흐름

컴포넌트가 초기화되면 Angular가 `url` 입력 신호를 설정하고, `resolvedUrl` 계산 신호가 `url()`의 값을 `resolvePrimitive`를 통해 문자열로 변환한다. 변환된 URL이 `<audio src>` 속성에 바인딩되고, 테마의 추가 스타일이 `<audio style>` 속성으로 적용된다. 입력이 변경되면 OnPush 변경 감지에 의해 DOM이 자동으로 갱신된다.
