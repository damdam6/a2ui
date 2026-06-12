# renderers/angular/a2ui_explorer/src/app/version_injector.ts

## 개요

활성화된 A2UI 프로토콜 버전을 결정하는 Angular DI 토큰(`A2UI_VERSION`)을 정의하는 파일이다. URL 쿼리 파라미터 `version`을 읽어 유효한 버전 값이 있으면 해당 버전을 사용하고, 없거나 유효하지 않으면 `Version.V0_9`를 기본값으로 반환한다. 서버 사이드 렌더링 환경에서도 안전하게 동작하도록 `window` 존재 여부를 검사한다.

## 의존성

### 외부 패키지
- `@angular/core` — `InjectionToken`

### 저장소 내부 모듈
- [`./types`](./types.ts.md) — `Version`

## Exports

| 이름 | 종류 | 설명 |
|---|---|---|
| `A2UI_VERSION` | `InjectionToken<Version>` 상수 | 활성 A2UI 버전을 제공하는 DI 토큰 |

## 상세 명세

### `const A2UI_VERSION: InjectionToken<Version>`

- **타입**: `InjectionToken<Version>`
- **토큰 이름**: `'A2UI_VERSION'`
- **providedIn**: `'root'` — 앱 루트 수준에서 자동으로 제공된다.

**factory 함수 동작 로직:**

1. `typeof window !== 'undefined'`를 확인하여 브라우저 환경인지 검사한다 (SSR/Node.js 환경에서 `window` 접근 방지).
2. 브라우저 환경이라면 `new URLSearchParams(window.location.search)`로 현재 URL의 쿼리 파라미터를 파싱한다.
3. `urlParams.get('version')`으로 `version` 파라미터 값을 읽는다.
4. 읽은 값이 `Version.V0_8`(`'v0.8'`) 또는 `Version.V0_9`(`'v0.9'`)와 정확히 일치하면 해당 값을 `Version`으로 캐스팅하여 반환한다.
5. 위 조건에 해당하지 않는 모든 경우(파라미터 없음, 잘못된 값, 비브라우저 환경) — `Version.V0_9`를 반환한다.

## 동작 흐름

Angular DI 시스템이 `A2UI_VERSION` 토큰을 처음 요청할 때 factory가 실행된다. URL 파라미터를 읽어 버전을 동적으로 결정하므로, 브라우저 URL에 `?version=v0.8`을 추가하면 v0.8 렌더러가 활성화되고, 그 외에는 항상 v0.9가 사용된다. `types.ts`에서 이 토큰을 re-export하므로 소비 코드는 `types` 단일 경로에서 import할 수 있다.
