# agent_sdks/kotlin/src/main/kotlin/com/google/a2ui/schema/SchemaResourceLoader.kt

## 개요

A2UI 스키마 파일을 JAR 번들 리소스로부터 로드하고, JSON 스키마 객체를 LLM 지시에 맞게 배열 래핑 형식으로 변환하는 유틸리티 싱글턴이다. JVM 파일 레벨 이름 `A2uiUtils`로 컴파일되어 Java 코드에서 정적 접근이 가능하다. 외부에서 직접 사용 가능한 두 개의 `@JvmStatic` 메서드를 제공하며, 스키마 로딩 실패 시 `IOException`을 던진다.

## 의존성

### 외부 패키지
- `java.io.IOException` — 스키마 로드 실패 예외
- `java.io.InputStream` — 리소스 스트림 읽기
- `java.util.logging.Logger` — 진단 로그 출력
- `kotlinx.serialization.json.Json` — JSON 파싱
- `kotlinx.serialization.json.JsonObject` — JSON 오브젝트 타입
- `kotlinx.serialization.json.JsonPrimitive` — JSON 원시값 타입

### 저장소 내부 모듈
- [`A2uiConstants`](A2uiConstants.kt.md) — `A2UI_ASSET_PACKAGE` 상수 제공 (파일 내 `A2uiConstants.A2UI_ASSET_PACKAGE`를 `A2UI_ASSET_PACKAGE`로 재노출)

## Exports

| 이름 | 종류 |
|------|------|
| `SchemaResourceLoader` | object (싱글턴) |
| `SchemaResourceLoader.A2UI_ASSET_PACKAGE` | `const val` (String) |
| `SchemaResourceLoader.loadFromBundledResource` | `@JvmStatic` 함수 |
| `SchemaResourceLoader.wrapAsJsonArray` | `@JvmStatic` 함수 |

## 상세 명세

### object SchemaResourceLoader

`@file:JvmName("A2uiUtils")` 어노테이션에 의해 JVM 바이트코드 파일명이 `A2uiUtils`로 지정된다. `package com.google.a2ui.schema`에 위치한다.

#### 필드

- `A2UI_ASSET_PACKAGE: String` (const val, public) — `A2uiConstants.A2UI_ASSET_PACKAGE`의 값인 `"com.google.a2ui.assets"`를 재노출.
- `logger: Logger` (private) — `SchemaResourceLoader::class.java.name`으로 초기화된 JUL 로거.

#### fun loadFromBundledResource(version: String, filename: String): JsonObject?

`@JvmStatic` 어노테이션으로 Java 정적 메서드로 노출된다.

동작 단계:
1. `resourcePath`를 `"/${A2UI_ASSET_PACKAGE.replace('.', '/')}/$version/$filename"` 형식으로 구성. 패키지 점(`.`)을 슬래시(`/`)로 치환하여 클래스패스 경로 형식으로 만든다. 예: 버전 `"v1"`, 파일 `"schema.json"` → `"/com/google/a2ui/assets/v1/schema.json"`.
2. `SchemaResourceLoader::class.java.getResourceAsStream(resourcePath)`로 스트림을 열고, 스트림이 null이 아닐 경우 `bufferedReader().use { ... }`로 전체 텍스트를 읽어 `Json.parseToJsonElement(...)` 후 `as JsonObject`로 캐스팅한다.
3. 스트림이 null이면 `logger.fine(...)` 로그 후 `IOException("Could not load schema $filename for version $version")`을 던진다.
4. 위 과정에서 임의의 `Exception`이 발생하면 `catch` 블록에서 `logger.fine(...)` 후 원인 예외를 래핑한 `IOException`을 던진다.
5. 반환 타입이 `JsonObject?`이지만 실제 정상 경로에서는 항상 non-null `JsonObject`를 반환하거나 `IOException`을 던진다 (null 반환은 코드상 도달 불가 경로).

#### fun wrapAsJsonArray(a2uiSchema: JsonObject): JsonObject

`@JvmStatic` 어노테이션으로 Java 정적 메서드로 노출된다.

동작 단계:
1. `require(a2uiSchema.isNotEmpty()) { "A2UI schema is empty" }`로 빈 스키마를 거부한다.
2. `JsonObject(mapOf("type" to JsonPrimitive("array"), "items" to a2uiSchema))`를 생성하여 반환한다. 즉 원본 스키마를 `{"type": "array", "items": <원본 스키마>}` 형태로 감싼다.
3. 이 래핑은 LLM이 메시지 목록을 생성하도록 지시받았기 때문에 스키마를 배열 형식으로 제시하기 위한 것이다 (KDoc 주석 설명).

## 동작 흐름

외부에서 특정 버전과 파일 이름으로 `loadFromBundledResource`를 호출하면, JAR 내부 클래스패스 리소스에서 JSON을 파싱하여 `JsonObject`를 반환한다. 이후 `wrapAsJsonArray`로 해당 스키마를 배열 items 형식으로 변환하여, JSON Schema 유효성 검사기 및 LLM 지시 생성에 사용한다. 오류는 모두 `IOException`으로 전파되며, 상세 원인은 `logger.fine` 레벨로 기록된다.
