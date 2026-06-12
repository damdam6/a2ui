# renderers/angular/src/v0_9/core/utils.spec.ts

## 개요

`utils.ts`에서 제공하는 `toAngularSignal` 함수와 `getNormalizedPath` 함수의 단위 테스트 파일이다. Preact Signal에서 Angular Signal로의 변환 동작, 생명주기 정리(dispose), NgZone 통합, 그리고 경로 정규화 로직을 검증한다.

## 의존성

### 외부 패키지
- `@preact/signals-core` — `signal` (Preact Signal 생성)
- `@angular/core` — `DestroyRef`

### 저장소 내부 모듈
- [`./utils`](./utils.ts.md) — `toAngularSignal`, `getNormalizedPath`

## Exports

없음 (테스트 전용 파일).

## 테스트 케이스 명세

### `describe('toAngularSignal')`

#### 공통 픽스처 (`beforeEach`)

- `mockDestroyRef`: `jasmine.createSpyObj('DestroyRef', ['onDestroy'])`로 생성.
- `onDestroyCallback`: `DestroyRef.onDestroy`에 등록된 콜백을 캡처하여 테스트에서 수동으로 호출할 수 있도록 하는 로컬 변수. `onDestroy.and.callFake`를 통해 캡처하며, 함수는 등록 해제 함수 `() => {}`를 반환한다.

---

#### `it('should initialize with the current value of Preact signal')`

**검증 동작**: `toAngularSignal`이 반환한 Angular Signal의 초기 값이 Preact Signal의 현재 값과 일치한다.

**픽스처**: `preactSignal('initial')` → `toAngularSignal` 변환 후 `angSig()`가 `'initial'`이어야 한다.

---

#### `it('should update Angular signal when Preact signal changes')`

**검증 동작**: Preact Signal의 `value`를 직접 변경하면 Angular Signal이 즉시 업데이트된다.

**검증 단계**:
1. 초기값 `'initial'` 확인.
2. `pSig.value = 'updated'` 설정.
3. `angSig()`가 `'updated'`이어야 한다.

---

#### `it('should dispose Preact effect when DestroyRef triggers')`

**검증 동작**: `onDestroyCallback()`을 수동으로 호출(소멸 시뮬레이션)하면 이후 Preact Signal 변경이 Angular Signal에 반영되지 않는다.

**검증 단계**:
1. 초기값 `'initial'` 확인.
2. `onDestroyCallback()` 호출로 정리 실행.
3. `pSig.value = 'updated'` 설정.
4. `angSig()`는 여전히 `'initial'`이어야 한다 (업데이트 차단 확인).

---

#### `it('should call unsubscribe on Preact signal if available on destroy')`

**검증 동작**: Preact Signal에 `unsubscribe` 메서드가 있는 경우, `onDestroyCallback()` 호출 시 해당 메서드가 실행된다.

**픽스처/모킹**: `pSig.unsubscribe = unsubscribeSpy`로 스파이를 동적으로 추가.

**검증 단계**:
1. 소멸 전에는 `unsubscribeSpy`가 호출되지 않아야 한다.
2. `onDestroyCallback()` 이후 `unsubscribeSpy`가 호출되어야 한다.

---

#### `it('should run update within NgZone if provided')`

**검증 동작**: `ngZone` 인자가 제공되면 Angular Signal 업데이트가 `ngZone.run()` 내에서 실행된다.

**픽스처/모킹**: `jasmine.createSpyObj('NgZone', ['run'])`로 목 NgZone 생성. `run.and.callFake((fn) => fn())`으로 실제 함수 실행.

**검증 단계**:
1. 초기화 시 `ngZone.run`이 호출되어야 한다.
2. `pSig.value = 'updated'` 설정 후 `ngZone.run`이 다시 호출되어야 한다.
3. `angSig()`는 `'updated'`이어야 한다.

---

### `describe('getNormalizedPath')`

#### `it('should handle absolute paths')`

**검증 동작**: 절대 경로(슬래시로 시작)는 `dataContextPath`와 무관하게 경로에 인덱스만 추가된다.

| 입력 `(path, dataContextPath, index)` | 기대 결과 |
|---------------------------------------|-----------|
| `('/absolute', '/', 0)` | `'/absolute/0'` |
| `('/absolute/', '/base', 5)` | `'/absolute/5'` (끝 슬래시 제거 후 인덱스 추가) |

---

#### `it('should resolve relative paths against dataContextPath')`

**검증 동작**: 상대 경로는 `dataContextPath`를 기준으로 결합된 후 인덱스가 추가된다.

| 입력 | 기대 결과 |
|------|-----------|
| `('relative', '/', 2)` | `'/relative/2'` (기본 경로가 `'/'`이면 빈 문자열로 처리) |
| `('relative', '/base', 3)` | `'/base/relative/3'` |

---

#### `it('should handle empty paths')`

**검증 동작**: 빈 문자열 경로는 `dataContextPath`에 인덱스만 추가된 결과를 반환한다.

| 입력 | 기대 결과 |
|------|-----------|
| `('', '/', 1)` | `'/1'` |
| `('', '/base', 4)` | `'/base/4'` |

## 동작 흐름

`toAngularSignal` 테스트 그룹은 Preact Signal을 직접 생성하고 `beforeEach`에서 준비된 목 `DestroyRef`를 함께 전달하여 Angular Signal로 변환한다. 생명주기 정리는 `onDestroyCallback`을 수동으로 호출하여 시뮬레이션한다. `getNormalizedPath` 테스트 그룹은 순수 함수이므로 픽스처 없이 입출력 쌍을 직접 검증한다.
