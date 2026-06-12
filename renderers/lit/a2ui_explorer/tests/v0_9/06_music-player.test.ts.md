# renderers/lit/a2ui_explorer/tests/v0_9/06_music-player.test.ts

## 개요

A2UI 예제 `06_music-player.json`을 `<local-gallery>`에 로드하여 음악 플레이어 UI가 올바르게 렌더링되는지 검증하는 통합 테스트 파일이다. 곡 제목·아티스트·재생 시간 텍스트, 앨범 아트 이미지 `src`, 그리고 컨트롤 아이콘 이름의 렌더링 여부를 확인한다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`../utils/test-utils`](../utils/test-utils.ts.md) — `loadExample`, `getSurface`, `getDeepTextContent`, `querySelectorAllDeep`
- [`../../src/local-gallery`](../../src/local-gallery.ts.md) — `LocalGallery` 타입

## Exports

없음 (테스트 파일)

## 테스트 케이스 명세

### `describe('Example: Music Player')`

**픽스처 설정:**
- `gallery: LocalGallery`, `surface: HTMLElement`, `textContent: string`을 스위트 범위 변수로 선언.
- `beforeEach`: `loadExample('06_music-player.json')`으로 갤러리를 마운트하고, `getSurface(gallery)`로 `a2ui-surface` 엘리먼트를 획득한 뒤, `getDeepTextContent(surface)`를 `textContent`에 저장.
- `afterEach`: `gallery?.remove()`로 DOM에서 갤러리를 제거.

---

**`it('should render text content')`**
- 검증 동작: 곡 제목, 아티스트 이름, 현재 재생 위치, 전체 재생 시간이 렌더링된다.
- 기대 결과: `textContent`에 `'Blinding Lights'`, `'The Weeknd'`, `'1:48'`, `'4:22'` 포함.

**`it('should render image')`**
- 검증 동작: 앨범 아트 `<img>` 엘리먼트가 존재하고 올바른 `src` 속성 값을 갖는다.
- 구현: `querySelectorAllDeep(surface, 'img')[0]`으로 이미지 엘리먼트를 찾음.
- 기대 결과: `img`가 존재(truthy)하고 `img.getAttribute('src')`가 `'https://images.unsplash.com/photo-1493225457124-a3eb161ffa5f?w=300&h=300&fit=crop'`와 정확히 일치.

**`it('should render icons')`**
- 검증 동작: 플레이어 컨트롤 아이콘들(`skip_previous`, `skip_next`, `pause`)의 텍스트 콘텐츠가 렌더링된다.
- 기대 결과: `textContent`에 `'skip_previous'`, `'skip_next'`, `'pause'` 포함.

## 동작 흐름

모든 테스트는 `beforeEach`에서 준비된 `textContent`와 `surface`를 사용하는 정적 렌더링 검증이다. 이미지 테스트만 `querySelectorAllDeep`을 추가로 호출한다. 인터랙션 테스트는 없다.
