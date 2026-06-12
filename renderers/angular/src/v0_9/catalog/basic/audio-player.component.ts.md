# renderers/angular/src/v0_9/catalog/basic/audio-player.component.ts

## 개요

A2UI v0.9 사양의 `AudioPlayer` 컴포넌트를 Angular로 구현한 파일이다. `BasicCatalogComponent`를 상속하며, `url`과 `description` 두 속성을 computed signal로 노출하여 오디오 플레이어 UI를 렌더링한다. 선택적 설명 텍스트와 네이티브 `<audio controls>` 요소로 구성되며, CSS 커스텀 속성으로 배경·테두리·패딩을 커스터마이징할 수 있다.

## 의존성

### 외부 패키지

- `@angular/core` — `Component`, `computed`, `ChangeDetectionStrategy`
- `@a2ui/web_core/v0_9/basic_catalog` — `AudioPlayerApi`

### 저장소 내부 모듈

- [`./basic-catalog-component`](basic-catalog-component.ts.md) — `BasicCatalogComponent`

## Exports

| 이름 | 종류 |
|---|---|
| `AudioPlayerComponent` | 클래스 (Angular `@Component`) |

## 상세 명세

### `AudioPlayerComponent` 클래스

데코레이터:
```
@Component({
  selector: 'a2ui-v09-audio-player',
  standalone: true,
  imports: [],
  changeDetection: ChangeDetectionStrategy.OnPush,
})
```

`BasicCatalogComponent<typeof AudioPlayerApi>`를 상속한다.

#### computed 필드

- `description = computed(() => this.props()['description']?.value())` — `props()` signal에서 `'description'` 키의 bound property 값을 꺼낸다. `undefined`이면 `undefined` 반환.
- `url = computed(() => this.props()['url']?.value())` — `props()` signal에서 `'url'` 키의 bound property 값을 꺼낸다. `undefined`이면 `undefined` 반환.

두 필드 모두 `readonly`이며, 부모 클래스 `CatalogComponent`의 `props` signal에 반응적으로 연결된다.

#### 템플릿 구조

최상위: `<div class="a2ui-audio-player">` (flex column 컨테이너)

- `@if (description())` 조건 분기: 참이면 `<div class="a2ui-audio-description">{{ description() }}</div>` 렌더링.
- `<audio [attr.src]="url() || null" controls class="a2ui-audio">` — `url()`이 falsy이면 `src` 속성을 제거(`null` 바인딩), `controls` 속성으로 기본 브라우저 컨트롤 활성화. 폴백 텍스트: `'Your browser does not support the audio tag.'`.

#### 인라인 스타일

- `.a2ui-audio-player`: `display: flex`, `flex-direction: column`, `gap: var(--a2ui-spacing-xs, 0.25rem)`, `background: var(--a2ui-audioplayer-background, transparent)`, `border-radius: var(--a2ui-audioplayer-border-radius, 0)`, `padding: var(--a2ui-audioplayer-padding, 0)`, `width: 100%`.
- `.a2ui-audio-description`: `font-size: var(--a2ui-font-size-s, 0.875rem)`, `color: var(--a2ui-text-caption-color, light-dark(#666, #aaa))`.
- `.a2ui-audio`: `width: 100%`.

#### 커스터마이징 CSS 변수

| 변수 | 기본값 | 설명 |
|---|---|---|
| `--a2ui-audioplayer-background` | `transparent` | 플레이어 배경 |
| `--a2ui-audioplayer-border-radius` | `0` | 테두리 반경 |
| `--a2ui-audioplayer-padding` | `0` | 내부 여백 |

## 동작 흐름

1. 컴포넌트가 생성될 때 `BasicCatalogComponent` 생성자가 `injectBasicCatalogStyles()`를 호출하여 기본 카탈로그 스타일을 주입한다.
2. `Renderer`(또는 v0.9의 컴포넌트 호스트)가 `props` input을 설정하면 `url`과 `description` computed signal이 자동으로 재계산된다.
3. `ChangeDetectionStrategy.OnPush`와 Angular signal의 결합으로, signal 값이 변경될 때만 뷰가 재렌더링된다.
4. `url`이 falsy이면 `<audio>` 요소에 `src` 속성이 제거되어 브라우저가 오디오를 로드하지 않는다.
5. `description`이 undefined/falsy이면 설명 div가 DOM에서 완전히 제거된다.
