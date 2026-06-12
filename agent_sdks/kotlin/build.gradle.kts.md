# agent_sdks/kotlin/build.gradle.kts

## 개요

Kotlin JVM 라이브러리인 `a2ui-agent` 모듈의 Gradle 빌드 스크립트다. Kotlin 직렬화, ADK(Google Agent Development Kit), A2A(Agent-to-Agent) SDK 등 핵심 의존성을 선언하고, A2UI 사양 JSON 파일을 빌드 타임에 리소스로 복사하는 커스텀 태스크를 정의한다. 저장소 루트의 `specification/` 디렉터리를 탐색하여 버전별 JSON 스키마를 생성된 리소스 경로에 배치한다.

## 의존성

외부 패키지만 사용하며 저장소 내부 모듈 import는 없다.

## Exports

빌드 스크립트이므로 Kotlin 심볼을 export하지 않는다. 그러나 이 파일이 정의하는 주요 구성 요소는 다음과 같다.

- `copySpecs` 태스크 (Copy 타입) — 버전별 JSON 스키마를 리소스로 복사
- `findRepoRoot()` 함수 — 빌드 스크립트 내부 헬퍼

## 상세 명세

### 플러그인

| 플러그인 | 버전 |
|---|---|
| `kotlin("jvm")` | `2.1.10` |
| `kotlin("plugin.serialization")` | `2.1.10` |
| `java-library` | — |
| `com.ncorti.ktfmt.gradle` | `0.19.0` |
| `org.jetbrains.kotlinx.kover` | `0.9.1` |

코드 포맷터는 `ktfmt { googleStyle() }` 설정으로 Google 스타일을 적용한다.

### 프로젝트 메타데이터

- `version`: `"0.1.0"`
- `group`: `"com.google.a2ui"`
- JVM 툴체인 대상: Java 21
- 저장소: `mavenCentral()`

### 의존성 목록

| 구성 | 좌표 | 버전 |
|---|---|---|
| `api` | `org.jetbrains.kotlinx:kotlinx-serialization-json` | `1.6.3` |
| `implementation` | `com.networknt:json-schema-validator` | `2.0.1` |
| `implementation` | `com.fasterxml.jackson.core:jackson-databind` | `2.17.2` |
| `api` | `com.google.adk:google-adk` | `0.9.0` |
| `api` | `com.google.adk:google-adk-a2a` | `0.9.0` |
| `api` | `io.github.a2asdk:a2a-java-sdk-client` | `1.0.0.Alpha3` |
| `api` | `com.google.genai:google-genai` | `1.43.0` |
| `testImplementation` | `kotlin("test")` | — |
| `testImplementation` | `io.mockk:mockk` | `1.13.11` |
| `testImplementation` | `com.fasterxml.jackson.dataformat:jackson-dataformat-yaml` | `2.17.2` |

테스트 실행기는 `tasks.test { useJUnitPlatform() }` 으로 JUnit Platform을 사용한다.

### `copySpecs` 태스크

타입: `Copy`. `findRepoRoot()`로 저장소 루트를 찾은 뒤 아래 파일들을 `layout.buildDirectory.dir("generated/resources/specs")` 경로에 복사한다.

| 원본 경로 (저장소 루트 기준) | 대상 서브디렉터리 |
|---|---|
| `specification/v0_8/json/server_to_client.json` | `com/google/a2ui/assets/0.8` |
| `specification/v0_8/json/standard_catalog_definition.json` | `com/google/a2ui/assets/0.8` |
| `specification/v0_9/json/server_to_client.json` | `com/google/a2ui/assets/0.9` |
| `specification/v0_9/json/common_types.json` | `com/google/a2ui/assets/0.9` |
| `specification/v0_9/catalogs/basic/catalog.json` | `com/google/a2ui/assets/0.9/catalogs/basic` |

`sourceSets.main.resources`에 위 태스크 출력이 추가되므로, 런타임에 클래스패스에서 해당 JSON 파일들을 읽을 수 있다.

### `findRepoRoot(): File`

프로젝트 디렉터리(`project.projectDir`)부터 상위 디렉터리로 거슬러 올라가며 `specification` 이름의 하위 디렉터리가 존재하는 디렉터리를 반환한다. 루트까지 탐색해도 찾지 못하면 `GradleException`을 던진다.

## 동작 흐름

빌드 시 `copySpecs` 태스크가 먼저 실행되어 사양 JSON 파일을 생성 리소스 경로로 복사하고, 이후 컴파일 단계에서 해당 경로가 클래스패스 리소스로 포함된다. 테스트는 JUnit Platform으로 실행된다.
