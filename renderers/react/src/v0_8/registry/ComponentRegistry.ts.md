# renderers/react/src/v0_8/registry/ComponentRegistry.ts

## 개요

A2UI 컴포넌트 타입 이름과 React 컴포넌트 구현을 매핑하는 레지스트리 클래스 파일이다. 싱글톤 패턴으로 전역 기본 레지스트리를 제공하면서도 인스턴스를 직접 생성하여 격리된 커스텀 레지스트리를 만들 수 있다. `React.lazy` 기반 지연 로딩을 투명하게 지원하며, 지연 로딩된 컴포넌트를 내부 캐시에 보관하여 중복 생성을 방지한다.

## 의존성

### 외부 패키지
- `react` — `lazy`, `ComponentType`
- `@a2ui/web_core/types/types` — `Types` 네임스페이스 전체 (타입 전용)

### 저장소 내부 모듈
- [`../types`](../types.ts.md) — `A2UIComponentProps`, `ComponentLoader`, `ComponentRegistration`

## Exports

| 이름 | 종류 |
|------|------|
| `ComponentRegistry` | 클래스 |

## 상세 명세

### `ComponentRegistry` 클래스

#### 정적 필드

| 이름 | 타입 | 설명 |
|------|------|------|
| `_instance` | `ComponentRegistry \| null` | 싱글톤 인스턴스 저장소; 초깃값 `null` |

#### 인스턴스 필드

| 이름 | 타입 | 설명 |
|------|------|------|
| `registry` | `Map<string, ComponentRegistration>` | 타입명→등록 정보 맵 |
| `lazyCache` | `Map<string, ComponentType<A2UIComponentProps>>` | lazy 래핑된 컴포넌트 캐시 |

#### `static getInstance(): ComponentRegistry`
`_instance`가 `null`이면 `new ComponentRegistry()`를 생성하여 할당한다. 항상 동일한 전역 인스턴스를 반환한다.

#### `static resetInstance(): void`
`_instance`를 `null`로 초기화한다. 주로 테스트에서 싱글톤 상태를 초기화할 때 사용한다.

#### `register<T extends Types.AnyComponentNode>(type: string, registration: ComponentRegistration<T>): void`
`this.registry.set(type, registration)`으로 컴포넌트를 등록한다. 이미 존재하는 타입에 대해 덮어쓰기된다. `lazyCache`는 갱신하지 않으므로 재등록 시 캐시가 오래된 상태로 남을 수 있다(재등록 후 `unregister` → `register` 패턴 권장).

#### `unregister(type: string): void`
`this.registry.delete(type)`과 `this.lazyCache.delete(type)` 두 맵 모두에서 항목을 삭제한다.

#### `has(type: string): boolean`
`this.registry.has(type)` 결과를 반환한다.

#### `get(type: string): ComponentType<A2UIComponentProps> | null`
1. `this.registry.get(type)`으로 등록 정보를 조회한다. 없으면 `null` 반환.
2. `registration.lazy === true`이고 `typeof registration.component === 'function'`인 경우:
   - `this.lazyCache.get(type)`을 확인한다. 캐시가 있으면 바로 반환.
   - 없으면 `lazy(registration.component as ComponentLoader)`로 lazy 컴포넌트를 생성하고 `this.lazyCache.set(type, lazyComponent)`으로 캐시한 뒤 반환.
3. lazy가 아닌 경우: `registration.component as ComponentType<A2UIComponentProps>`를 반환한다.

#### `getRegisteredTypes(): string[]`
`Array.from(this.registry.keys())`를 반환한다.

#### `clear(): void`
`this.registry.clear()`와 `this.lazyCache.clear()`를 모두 호출한다.

## 동작 흐름

```
싱글톤 접근
  → getInstance() → 최초면 new ComponentRegistry() 생성, 이후 동일 인스턴스 반환

컴포넌트 등록
  → register('Button', {component: ButtonComponent}) — 즉시 로딩
  → register('Modal', {component: () => import('./Modal'), lazy: true}) — 지연 로딩

컴포넌트 조회 (ComponentNode에서 호출)
  → get('Button') → ButtonComponent 반환 (non-lazy)
  → get('Modal') → lazy로 등록됨
    → lazyCache에 없으면 React.lazy() 생성 후 캐시
    → lazyCache에 있으면 캐시된 lazy 컴포넌트 반환

테스트 격리
  → resetInstance() → 싱글톤 초기화 → 다음 getInstance()에서 새 인스턴스 생성
```
