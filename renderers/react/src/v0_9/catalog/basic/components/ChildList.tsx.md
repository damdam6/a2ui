# renderers/react/src/v0_9/catalog/basic/components/ChildList.tsx

## 개요

`ChildList`는 자식 컴포넌트 참조 목록(`ResolvedChildList`)을 순회하여 각 자식을 `buildChild` 콜백으로 렌더링하는 범용 헬퍼 컴포넌트다. 자식 참조는 단순 `ComponentId` 문자열이거나 `{id, basePath}` 객체일 수 있으며, 두 형태를 모두 처리한다. `Column`과 `List` 등 자식 목록을 가지는 레이아웃 컴포넌트가 이 컴포넌트를 재사용한다.

## 의존성

### 외부 패키지
- `react` — `React`, `React.FC`, `React.ReactNode`, `React.Fragment` 사용
- `@a2ui/web_core/v0_9` — `ComponentContext`, `ComponentId` 타입 가져오기

### 저장소 내부 모듈
없음

## Exports

| 이름 | 종류 |
|------|------|
| `ChildList` | 상수 (`React.FC<{childList, context, buildChild}>`) |

## 상세 명세

### 타입 정의 (비공개)

#### `ResolvedChildRef`
```
type ResolvedChildRef =
  | ComponentId
  | { id: ComponentId; basePath: string; }
```
자식 참조의 두 가지 형태를 나타내는 유니온 타입. 단순 ID 문자열이거나, ID와 basePath를 포함하는 객체다.

#### `ResolvedChildList`
`type ResolvedChildList = ResolvedChildRef[]` — 자식 참조 배열 타입.

### `ChildList`

**시그니처**:
```
React.FC<{
  childList: ResolvedChildList;
  context: ComponentContext;
  buildChild: (id: ComponentId, basePath?: string) => React.ReactNode;
}>
```

**동작 로직**:
1. `childList`가 배열이 아닌 경우 `null`을 반환한다 (방어적 처리).
2. `<>...</>` (React Fragment) 안에서 `childList.map`을 수행한다.
3. 각 `childRef`에 대해 타입을 분기한다:
   - `typeof childRef === 'string'`이면 단순 ID로 간주하고, `buildChild(childRef)`를 호출한다. key는 `` `${childRef}-${index}` ``.
   - 그렇지 않으면 `{id, basePath}` 객체로 간주하고, `buildChild(childRef.id, childRef.basePath)`를 호출한다. key는 `` `${childRef.id}-${childRef.basePath}` ``.
4. 각 항목을 `<React.Fragment key={...}>` 로 감싸 DOM 노드 없이 고유 key를 부여한다.

**주의**: `context` prop은 시그니처에 포함되어 있으나 렌더 함수에서 구조분해 시 사용되지 않는다(소비하지 않음). 인터페이스 일관성을 위해 선언만 되어 있다.

## 동작 흐름

부모 컴포넌트가 `childList`, `context`, `buildChild`를 props로 전달하면, `ChildList`는 목록을 순회하면서 자식 참조 형태에 따라 적절한 인수로 `buildChild`를 호출하고 그 결과를 Fragment 배열로 반환한다. 내부 상태 없음.
