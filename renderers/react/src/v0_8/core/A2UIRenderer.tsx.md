# renderers/react/src/v0_8/core/A2UIRenderer.tsx

## 개요

특정 `surfaceId`에 해당하는 A2UI 서피스를 렌더링하는 진입점 컴포넌트 파일이다. A2UI 상태 스토어에서 서피스 데이터를 읽어 컴포넌트 트리를 `ComponentNode`로 위임 렌더링하고, `primaryColor`와 `font` 스타일 속성을 CSS 커스텀 프로퍼티로 변환하여 인라인 스타일로 주입한다. `React.memo`로 메모이제이션되어 있다.

## 의존성

### 외부 패키지
- `react` — `Suspense`, `useMemo`, `memo`, `ReactNode`

### 저장소 내부 모듈
- [`../hooks/useA2UI`](../hooks/useA2UI.ts.md) — `useA2UI` 훅
- [`./ComponentNode`](./ComponentNode.tsx.md) — `ComponentNode` 컴포넌트
- [`../registry/ComponentRegistry`](../registry/ComponentRegistry.ts.md) — `ComponentRegistry` 타입
- [`../lib/utils`](../lib/utils.ts.md) — `cn` 함수

## Exports

| 이름 | 종류 |
|------|------|
| `A2UIRendererProps` | 인터페이스 |
| `A2UIRenderer` | React 컴포넌트 (memo, named export) |
| `default` | `A2UIRenderer`의 default re-export |

## 상세 명세

### `DefaultLoadingFallback` (내부 상수)

`memo`로 감싼 React 함수 컴포넌트. `className="a2ui-loading"`, `padding: '16px'`, `opacity: 0.5` 스타일의 `div` 안에 `"Loading..."` 텍스트를 렌더링한다. 매 렌더 시 재생성되지 않도록 모듈 스코프에서 메모이제이션되어 있다.

### `A2UIRendererProps` 인터페이스

| 필드 | 타입 | 기본값 | 설명 |
|------|------|--------|------|
| `surfaceId` | `string` | 필수 | 렌더링할 서피스의 ID |
| `className` | `string` | 없음 | 서피스 컨테이너에 추가할 CSS 클래스 |
| `fallback` | `ReactNode` | `null` | 서피스가 아직 없을 때 표시할 대체 콘텐츠 |
| `loadingFallback` | `ReactNode` | 없음 | 지연 로딩 컴포넌트의 로딩 폴백 |
| `registry` | `ComponentRegistry` | 없음 | 커스텀 컴포넌트 레지스트리 |

### `A2UIRenderer` 컴포넌트

**시그니처:** `memo(function A2UIRenderer({surfaceId, className, fallback, loadingFallback, registry}: A2UIRendererProps): JSX.Element)`

**동작 단계:**
1. `useA2UI()`를 호출해 `getSurface`와 `version`을 얻는다. `version`을 destructuring함으로써 상태 변경 시 이 컴포넌트가 리렌더링되도록 구독한다.
2. `getSurface(surfaceId)`로 현재 서피스 데이터를 조회한다.
3. `surfaceStyles`를 `useMemo`로 계산한다. `surface?.styles`가 없으면 빈 객체 `{}`를 반환한다. 스타일 키별 처리:
   - `'primaryColor'`: `--p-0`(#000000)~`--p-100`(#ffffff) 범위의 CSS 커스텀 프로퍼티 팔레트를 생성한다. `--p-50`은 원본 값 그대로, 나머지는 `color-mix(in srgb, <value> X%, white/black Y%)` 형식으로 생성한다. 총 16개의 커스텀 프로퍼티(`--p-0`, `--p-5`, `--p-10`, `--p-15`, `--p-20`, `--p-25`, `--p-30`, `--p-35`, `--p-40`, `--p-50`, `--p-60`, `--p-70`, `--p-80`, `--p-90`, `--p-95`, `--p-98`, `--p-99`, `--p-100`)를 설정한다.
   - `'font'`: `--font-family`와 `--font-family-flex` 두 커스텀 프로퍼티에 동일 값을 설정한다.
   - 그 외 키는 무시한다.
   - 메모이제이션 의존성: `[surface?.styles]`.
4. `surface` 또는 `surface.componentTree`가 없으면 `<>{fallback}</>`를 반환한다.
5. `actualLoadingFallback`은 `loadingFallback`이 제공되면 그것을, 아니면 `<DefaultLoadingFallback />`을 사용한다.
6. 최종 렌더:
   - 최상위 `div`에 `className={cn('a2ui-surface', className)}`, `style={surfaceStyles}`, `data-surface-id={surfaceId}`, `data-version={version}` 속성을 부여한다.
   - 내부에 `<Suspense fallback={actualLoadingFallback}>`로 감싸고 `<ComponentNode node={surface.componentTree} surfaceId={surfaceId} registry={registry} />`를 렌더링한다.

## 동작 흐름

```
props 변경 없음 → memo가 리렌더링 차단
version 변경 (서버 메시지 수신)
  → useA2UI()가 새 version 반환
  → getSurface(surfaceId)로 최신 서피스 조회
  → surfaceStyles 재계산 (styles가 변경된 경우)
  → 서피스 없음 → fallback 반환
  → 서피스 있음 → 컴포넌트 트리 렌더링
    → Suspense로 지연 로딩 처리
    → ComponentNode로 루트 노드 렌더링 위임
```
