# renderers/lit/a2ui_explorer/vite.config.ts

## 개요

`a2ui_explorer` 서브 패키지를 위한 Vite 빌드 및 개발 서버 설정 파일이다. 빌드 타깃, 패키지 중복 제거, 파일 시스템 접근 허용 경로 세 가지 항목을 설정한다.

## 의존성

### 외부 패키지
- `vite` — `defineConfig` 함수

### 저장소 내부 모듈
없음

## Exports

- `default` (상수): `defineConfig(...)` 반환값 — Vite 설정 객체

## 상세 명세

### `default` (Vite 설정 객체)

`defineConfig`에 전달되는 설정 객체는 아래 세 섹션으로 구성된다.

**`build`**
- `target`: `'esnext'` — 최신 ECMAScript 기능을 그대로 출력하도록 빌드 타깃을 지정한다.

**`resolve`**
- `dedupe`: `['lit']` — `lit` 패키지가 다수의 의존성에서 각각 번들링되어 중복 인스턴스가 생기는 것을 방지하고, 항상 단일 인스턴스만 사용하도록 강제한다.

**`server`**
- `fs.allow`: `['../', '../../../specification']` — 개발 서버가 기본 루트(`a2ui_explorer/`) 외부의 파일도 서빙할 수 있도록 허용 경로를 명시한다. 부모 디렉토리(`renderers/lit/`)와 프로젝트 루트 기준 `specification` 디렉토리 접근을 허용한다.

## 동작 흐름

파일 실행 시 `defineConfig`를 호출하여 설정 객체를 생성하고 default export로 내보낸다. Vite CLI가 이 설정을 읽어 빌드(`vite build`) 및 개발 서버(`vite dev`) 동작을 결정한다.
