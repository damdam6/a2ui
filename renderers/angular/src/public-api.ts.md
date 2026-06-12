# renderers/angular/src/public-api.ts

## 개요

Angular 렌더러 라이브러리의 최상위 공개 API 진입점이다. 라이브러리 버전 상수를 선언하고, v0_8 하위 패키지의 모든 공개 심볼을 재내보낸다. 외부 소비자는 이 파일 하나만 임포트하면 라이브러리 전체 공개 API에 접근할 수 있다.

## 의존성

### 외부 패키지
없음

### 저장소 내부 모듈
- [`./v0_8/public-api`](./v0_8/public-api.ts.md)

## Exports

| 이름 | 종류 |
|------|------|
| `A2UI_ANGULAR_VERSION` | 상수 (`string`) |
| `./v0_8/public-api`의 모든 심볼 | 재내보내기 (`export *`) |

## 상세 명세

### 상수: `A2UI_ANGULAR_VERSION`
- 타입: `string`
- 값: `'0.9.0'`
- 이 Angular 렌더러 라이브러리의 현재 배포 버전을 나타낸다.

### 재내보내기: `export * from './v0_8/public-api'`
`./v0_8/public-api.ts`가 내보내는 모든 심볼을 그대로 전달한다. 포함 항목: 카탈로그(`DEFAULT_CATALOG`, `registerStandardComponents`), 각 컴포넌트 클래스(`AudioPlayer`, `Button`, `Card` 등), 설정(`config`), 데이터 처리 모듈(`data/index`), 렌더링 모듈(`rendering/index`), 타입 네임스페이스(`Types`).

## 동작 흐름

파일이 임포트되는 즉시 `A2UI_ANGULAR_VERSION` 상수를 노출하고, `./v0_8/public-api`가 내보내는 모든 심볼을 그대로 전달(re-export)한다. 별도 실행 로직이나 사이드 이펙트는 없다.
