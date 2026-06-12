# agent_sdks/kotlin/settings.gradle.kts

## 개요

Gradle 멀티 프로젝트 설정 파일이다. 이 모듈의 루트 프로젝트 이름을 `"a2ui-agent"`로 지정하는 단 한 줄의 설정만 포함한다.

## 의존성

없음.

## Exports

없음 (빌드 설정 파일).

## 상세 명세

- `rootProject.name = "a2ui-agent"` — Gradle이 이 빌드의 루트 프로젝트를 `a2ui-agent`로 인식하게 한다. 이 이름은 발행(publish) 아티팩트 이름이나 멀티 모듈 참조에 영향을 미친다.

## 동작 흐름

Gradle 빌드 초기화 단계에서 이 파일을 읽어 루트 프로젝트 이름을 설정한다.
