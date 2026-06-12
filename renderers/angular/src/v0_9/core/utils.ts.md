# renderers/angular/src/v0_9/core/utils.ts

## 개요

A2UI Angular 렌더러에서 Preact Signal 생태계와 Angular Signal 시스템을 연결하는 핵심 유틸리티 함수들을 제공한다. `toAngularSignal`은 A2UI 데이터 모델의 반응성을 Angular 변경 감지에 올바르게 전파하며, `getNormalizedPath`는 반복 컬렉션 내 자식 컴포넌트를 위한 절대 경로를 생성한다.

## 의존성

### 외부 패키지
- `@angular/core` — `DestroyRef`, `Signal`, `signal as angularSignal`, `NgZone`
- `@a2ui/web_core/v0_9` — `Signal as PreactSignal`, `effect`, `signal as preactSignal`

### 저장소 내부 모듈

없음.

## Exports

- `preactSignal` (re-export) — `@a2ui/web_core/v0_9`의 `signal`을 `preactSignal`로 재내보냄
- `toAngularSignal` (함수)
- `getNormalizedPath` (함수)

## 상세 명세

### `export { preactSignal }`

`@a2ui/web_core/v0_9`에서 가져온 `signal`을 `preactSignal`이라는 이름으로 재내보낸다. 다른 모듈이 이 파일을 통해 Preact Signal 생성자를 임포트할 수 있도록 한다.

---

### `function toAngularSignal<T>(preactSignal: PreactSignal<T>, destroyRef: DestroyRef, ngZone?: NgZone): Signal<T>`

**제네릭**: `T` — Signal이 보유하는 값의 타입.

**매개변수**:
| 이름 | 타입 | 설명 |
|------|------|------|
| `preactSignal` | `PreactSignal<T>` | 소스 Preact Signal (A2UI 데이터 모델에서 유래) |
| `destroyRef` | `DestroyRef` | Angular 컴포넌트 생명주기 관리용 |
| `ngZone` | `NgZone \| undefined` | 선택적. OnPush 컴포넌트의 올바른 변경 감지를 위해 필요한 경우 제공 |

**반환**: 읽기 전용 Angular `Signal<T>`

**동작 로직**:
1. `preactSignal.peek()`을 초기 값으로 사용하여 Angular Signal `s = angularSignal(preactSignal.peek())`를 생성한다. `peek()`을 사용하여 초기화 시 Preact 이펙트 구독을 트리거하지 않는다.
2. `effect()`로 Preact 이펙트를 등록한다. 이펙트 내부에서:
   - `ngZone`이 있으면: `ngZone.run(() => s.set(preactSignal.value))`로 Angular 존 내에서 업데이트를 수행한다.
   - `ngZone`이 없으면: `s.set(preactSignal.value)`를 직접 호출한다.
   - 이펙트는 `preactSignal.value`를 읽으므로, Preact Signal이 변경될 때마다 자동으로 재실행된다.
3. `effect()`가 반환한 `dispose` 함수를 `destroyRef.onDestroy()`에 등록한다. 컴포넌트 소멸 시:
   - `dispose()`를 호출하여 Preact 이펙트를 해제한다.
   - `(preactSignal as any).unsubscribe`가 존재하면 호출한다 (일부 `DataContext.resolveSignal`이 반환하는 Signal은 AbortController를 위한 커스텀 `unsubscribe`를 가진다).
4. `s.asReadonly()`를 반환하여 외부에서 값을 직접 변경할 수 없도록 한다.

**경계 케이스**: `preactSignal`에 `unsubscribe` 메서드가 없으면 그냥 건너뛴다.

---

### `function getNormalizedPath(path: string | undefined, dataContextPath: string, index: number): string`

**매개변수**:
| 이름 | 타입 | 설명 |
|------|------|------|
| `path` | `string \| undefined` | 컴포넌트 속성에서 온 상대 또는 절대 경로 |
| `dataContextPath` | `string` | 현재 컨텍스트의 기준 경로 |
| `index` | `number` | 반복 컬렉션 내 자식 컴포넌트의 인덱스 |

**반환**: 인덱스가 포함된 완전한 절대 경로 문자열.

**동작 로직**:
1. `normalized = path || ''` — `path`가 `undefined` 또는 빈 문자열이면 빈 문자열 사용.
2. `normalized`가 `'/'`로 시작하지 않으면 (상대 경로인 경우):
   - `base = dataContextPath === '/' ? '' : dataContextPath` — `dataContextPath`가 `'/'`이면 기본 경로를 빈 문자열로 처리하여 이중 슬래시를 방지한다.
   - `normalized = \`${base}/${normalized}\``
3. `normalized`가 `'/'`로 끝나면 마지막 슬래시를 제거한다: `normalized.slice(0, -1)`.
4. `\`${normalized}/${index}\``를 반환하여 인덱스를 경로 끝에 추가한다.

**경계 케이스**:
- 절대 경로(`/absolute/`)는 `dataContextPath`와 관계없이 처리되며 끝 슬래시만 제거된 후 인덱스가 추가된다.
- 빈 `path`와 `dataContextPath === '/'` 조합은 `'/index'` 형태의 경로를 생성한다.

## 동작 흐름

이 파일은 두 가지 독립적인 유틸리티를 제공한다. `toAngularSignal`은 `ComponentBinder` 서비스에서 각 컴포넌트 속성을 Angular Signal로 변환할 때 호출되며, Preact 이펙트와 Angular Signal을 연결하는 브릿지 역할을 한다. `getNormalizedPath`는 `ComponentHostComponent`에서 반복 컬렉션의 자식 컴포넌트 경로를 계산할 때 사용된다.
