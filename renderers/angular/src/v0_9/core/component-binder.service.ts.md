# renderers/angular/src/v0_9/core/component-binder.service.ts

## 개요

A2UI `ComponentContext`의 프로퍼티를 Angular reactive signal로 변환하는 서비스다. 리터럴 값, DataModel path 바인딩, 단일 자식 참조, 자식 목록 확장, 유효성 검사 룰을 각각 다른 방식으로 처리하며, 모든 결과는 `BoundProperty` 형태로 반환된다. `providedIn: 'root'`로 등록된 싱글턴 서비스다.

## 의존성

### 외부 패키지
- `@angular/core`: `DestroyRef`, `Injectable`, `inject`, `NgZone`
- `@a2ui/web_core/v0_9`: `ComponentContext`, `computed`

### 저장소 내부 모듈
- [`./utils`](./utils.ts.md) — `toAngularSignal`
- [`./types`](./types.ts.md) — `BoundProperty`, `ComponentTemplate`

## Exports

| 이름 | 종류 |
|------|------|
| `Child` | 인터페이스 |
| `ComponentBinder` | Angular Injectable 클래스 |

## 상세 명세

### `Child`

인터페이스.

| 필드 | 타입 | 설명 |
|------|------|------|
| `id` | `string` | 자식 컴포넌트 ID |
| `basePath` | `string` | 데이터 컨텍스트 기준 경로 |

---

### `ComponentBinder`

**데코레이터**: `@Injectable({ providedIn: 'root' })`

#### 필드

| 이름 | 접근 | 타입 | 설명 |
|------|------|------|------|
| `destroyRef` | private | `DestroyRef` | inject로 획득. signal cleanup에 사용. |
| `ngZone` | private | `NgZone` | inject로 획득. Angular zone 안에서 signal 업데이트를 보장. |

#### `bind(context: ComponentContext): Record<string, BoundProperty>`

매개변수: `context: ComponentContext`  
반환: `Record<string, BoundProperty>` — 프로퍼티 키 → `BoundProperty` 객체 맵

**동작 로직 (단계별)**:

1. `context.componentModel.properties`에서 모든 키를 순회한다.

2. 각 키-값(`key`, `value`)에 대해 프로퍼티 분류를 수행한다:
   - **`isChildListTemplate`**: `value`가 객체이고 `'componentId'`와 `'path'` 두 키를 모두 가지면 true.
   - **`isBoundPath`**: `value`가 객체이고 `'path'` 키만 가지면(componentId 없음) true.

3. **`isChildListTemplate` = true** 인 경우:
   - `context.dataContext.resolveSignal({ path: value.path })`로 배열 signal을 획득.
   - `context.dataContext.nested(value.path)`로 중첩 컨텍스트를 생성.
   - `computed`로 배열을 순회하며 각 인덱스에 대해 `{ id: value.componentId, basePath: listContext.nested(String(i)).path }` 형태의 `Child` 배열을 생성.

4. **그 외** (리터럴, path 바인딩, null 등):
   - `context.dataContext.resolveSignal(value)`로 preact signal을 획득.

5. **`child`, `trigger`, `content` 키** 특수 처리:
   - 원본 signal 값이 falsy이면 null 반환.
   - 값이 `{ id: ... }` 형태 객체이면 그대로 반환.
   - 값이 문자열이면 `{ id: val, basePath: context.dataContext.path }` 형태로 변환.

6. **`children` 키** 특수 처리:
   - `value.componentId`와 `value.path`가 모두 존재하면 `template = { id, path }`로 설정.
   - `computed`로 배열을 순회하며: 항목이 `{ id }` 형태 객체이면 그대로, 문자열이면 `{ id: item, basePath: context.dataContext.path }`로 변환.

7. `toAngularSignal(preactSig, this.destroyRef, this.ngZone)`으로 Angular signal(`angSig`)로 변환.

8. `bound[key]` 에 `BoundProperty` 객체를 저장:
   - `value`: Angular signal (`angSig`)
   - `raw`: 원본 값 (`value`)
   - `template`: ChildList 템플릿이면 `{ id, path }`, 그 외 `undefined`
   - `onUpdate`: `isBoundPath`이면 `(newValue) => context.dataContext.set(value.path, newValue)`, 그 외 no-op (`() => {}`)

9. **`checks` 키** 추가 처리:
   - `value`가 배열이면 각 룰을 처리한다.
   - 각 룰에서 `condition = rule.condition || rule`, `message = rule.message || 'Validation failed'`로 추출.
   - 각 조건에 대해 `context.dataContext.resolveSignal(condition)`으로 signal을 획득하고 `{ conditionSig, message }` 객체를 생성.
   - `isValidPreactSig = computed(() => ruleResults.every(r => !!r.conditionSig.value))` 생성.
   - `validationErrorsPreactSig = computed(() => ruleResults.filter(r => !r.conditionSig.value).map(r => r.message))` 생성.
   - `bound['isValid']`와 `bound['validationErrors']`에 각각 Angular signal로 변환한 `BoundProperty`를 저장. 두 항목 모두 `raw: null`, `onUpdate: () => {}`.

10. 완성된 `bound` 객체를 반환한다.

## 동작 흐름

`bind()`는 컴포넌트가 마운트될 때 한 번 호출되며, 컴포넌트 모델이 업데이트되면 `ComponentHostComponent`에 의해 재호출된다. 반환된 `BoundProperty` 객체의 `value` signal은 DataModel이 변경될 때 자동으로 재평가된다. path 바인딩의 `onUpdate` 콜백은 사용자 입력 등 컴포넌트에서 발생한 변경사항을 DataModel에 역방향으로 전파한다.
