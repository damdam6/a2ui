# agent_sdks/kotlin/src/main/kotlin/com/google/a2ui/schema/CatalogProvider.kt

## 개요

카탈로그 스키마를 로드하는 전략 인터페이스(`A2uiCatalogProvider`)와 파일시스템 구현체(`FileSystemCatalogProvider`)를 정의하는 소형 파일이다. 로드 전략을 추상화하여 향후 다른 소스(HTTP 등)를 쉽게 추가할 수 있도록 설계되었다.

## 의존성

### 외부 패키지
- `java.io.File`, `java.io.IOException` — 파일 읽기 및 오류 처리
- `kotlinx.serialization.json.Json`, `kotlinx.serialization.json.JsonObject` — JSON 파싱

### 저장소 내부 모듈
없음.

## Exports

| 이름 | 종류 |
|---|---|
| `A2uiCatalogProvider` | interface |
| `FileSystemCatalogProvider` | class |

## 상세 명세

### interface `A2uiCatalogProvider`

```
interface A2uiCatalogProvider {
  fun load(): JsonObject
}
```

카탈로그 정의를 어떤 소스에서든 `JsonObject`로 반환하는 단일 메서드 인터페이스다. 구현체가 로드 로직(파일, 클래스패스 리소스, 네트워크 등)을 캡슐화한다.

---

### class `FileSystemCatalogProvider`

```
class FileSystemCatalogProvider(val path: String) : A2uiCatalogProvider
```

| 필드 | 타입 | 설명 |
|---|---|---|
| `path` | `String` | 로드할 JSON 파일의 로컬 파일시스템 경로 |

#### fun `load(): JsonObject`

1. `File(path).readText(Charsets.UTF_8)`로 파일 내용을 UTF-8 문자열로 읽는다.
2. `Json.parseToJsonElement(content) as JsonObject`로 파싱하여 반환한다.
3. `File`을 읽거나 파싱하는 과정에서 예외가 발생하면 `IOException("Could not load schema from ${path}: ${e.message}", e)`으로 래핑하여 던진다.

## 동작 흐름

`CatalogConfig.fromPath`가 `FileSystemCatalogProvider`를 생성하여 `CatalogConfig.provider`에 주입한다. `A2uiSchemaManager`의 `init` 블록에서 `config.provider.load()`가 호출될 때 파일을 읽어 JSON으로 파싱한 뒤 `A2uiCatalog` 생성에 사용된다.
