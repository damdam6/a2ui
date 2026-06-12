# renderers/lit/src/0.8/ui/context/context.ts

## 개요

`context.ts`는 UI 서브시스템의 Lit 컨텍스트 객체를 하나로 모아 재내보내는 배럴(barrel) 모듈이다. `theme.ts`와 `markdown.ts`에서 정의된 모든 컨텍스트 심볼을 단일 진입점으로 노출하므로, 소비자는 개별 파일을 직접 참조하지 않아도 된다.

## 의존성

### 외부 패키지
없음.

### 저장소 내부 모듈
- [`./theme.js`](./theme.ts.md) — `theme`, `themeContext` 재내보내기
- [`./markdown.js`](./markdown.ts.md) — `markdown` 재내보내기

## Exports

- `theme` (재내보내기, `./theme.js`에서)
- `themeContext` (재내보내기, `./theme.js`에서, deprecated)
- `markdown` (재내보내기, `./markdown.js`에서)

## 동작 흐름

이 파일은 로직을 포함하지 않으며 두 개의 `export *` 선언만 있다. 임포트 시 `theme.ts`와 `markdown.ts`가 각각 실행되어 컨텍스트 객체가 생성된다.
