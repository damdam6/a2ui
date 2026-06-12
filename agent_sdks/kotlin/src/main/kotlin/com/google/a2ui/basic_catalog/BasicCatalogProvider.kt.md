# agent_sdks/kotlin/src/main/kotlin/com/google/a2ui/basic_catalog/BasicCatalogProvider.kt

## 개요

A2UI 기본(basic) 카탈로그의 로딩 및 설정 접근을 담당하는 파일이다. `BundledCatalogProvider`는 버전별로 번들된 패키지 리소스에서 JSON 카탈로그를 로드하며, `BasicCatalog` 싱글턴 객체는 버전별 경로 맵과 `CatalogConfig` 빌더 메서드를 제공한다. JVM 이름이 `BasicCatalogApi`로 고정되어 있어 Java에서도 일관된 API를 사용할 수 있다.

## 의존성

### 외부 패키지
- `kotlinx.serialization.json.JsonObject`
- `kotlinx.serialization.json.JsonPrimitive`

### 저장소 내부 모듈
- `com.google.a2ui.schema.A2uiCatalogProvider` (schema 패키지)
- `com.google.a2ui.schema.A2uiConstants` (schema 패키지)
- `com.google.a2ui.schema.A2uiVersion` (schema 패키지)
- `com.google.a2ui.schema.CatalogConfig` (schema 패키지)
- `com.google.a2ui.schema.SchemaResourceLoader` (schema 패키지)
- `com.google.a2ui.schema.resolveExamplesPath` (schema 패키지)

## Exports

- `BundledCatalogProvider` (class)
- `BasicCatalog` (object)

## 상세 명세

### `class BundledCatalogProvider(private val version: A2uiVersion) : A2uiCatalogProvider`

지정된 버전의 A2UI 카탈로그 JSON을 번들 리소스에서 로드하는 카탈로그 제공자다.

#### `override fun load(): JsonObject`

처리 단계:

1. `BasicCatalog.BASIC_CATALOG_PATHS[version]`에서 경로 맵을 가져온다. 없으면 빈 맵 사용.
2. `A2uiConstants.CATALOG_SCHEMA_KEY`로 상대 경로 문자열을 조회한다.
3. 버전에 따라 파일명을 추출한다:
   - `VERSION_0_9`: `"specification/v0_9/"` 이후 부분
   - `VERSION_0_9_1`: `"specification/v0_9_1/"` 이후 부분
   - 그 외: 마지막 `'/'` 이후 부분
4. `SchemaResourceLoader.loadFromBundledResource(version.value, filename)`으로 리소스를 로드하여 변경 가능한 맵으로 복사한다. 로드 실패 시 빈 맵 사용.
5. `A2uiConstants.CATALOG_ID_KEY`가 맵에 없으면, 경로 문자열에서 `"/json/"` 을 `"/"` 로 치환한 경로를 `A2uiConstants.BASE_SCHEMA_URL` 과 결합하여 `JsonPrimitive`로 삽입한다.
6. `"$schema"` 키가 없으면 `"https://json-schema.org/draft/2020-12/schema"` 값의 `JsonPrimitive`를 삽입한다.
7. `JsonObject(resource)`를 반환한다.

---

### `object BasicCatalog`

#### 상수/필드

| 이름 | 타입 | 값 |
|---|---|---|
| `BASIC_CATALOG_NAME` | `String` const | `"basic"` |
| `BASIC_CATALOG_PATHS` | `Map<A2uiVersion, Map<String, String>>` (`@JvmField`) | 아래 표 참조 |

`BASIC_CATALOG_PATHS` 내용:

| `A2uiVersion` | `CATALOG_SCHEMA_KEY` 경로 |
|---|---|
| `VERSION_0_8` | `"specification/v0_8/json/standard_catalog_definition.json"` |
| `VERSION_0_9` | `"specification/v0_9/catalogs/basic/catalog.json"` |
| `VERSION_0_9_1` | `"specification/v0_9_1/catalogs/basic/catalog.json"` |

#### `@JvmStatic @JvmOverloads fun getConfig(version: A2uiVersion, examplesPath: String? = null): CatalogConfig`

`CatalogConfig`를 생성하여 반환한다.
- `name`: `BASIC_CATALOG_NAME`(`"basic"`)
- `provider`: `BundledCatalogProvider(version)` 인스턴스
- `examplesPath`: `resolveExamplesPath(examplesPath)` 결과

Java 코드에서 catalog 설정을 쉽게 인스턴스화할 수 있는 진입점 역할을 한다.

## 동작 흐름

사용자(또는 상위 시스템)가 `BasicCatalog.getConfig(version)`으로 카탈로그 설정을 가져오고, 이를 인프라에 등록한다. 런타임에 카탈로그가 필요할 때 `BundledCatalogProvider.load()`가 호출되어 번들된 JSON 파일을 읽고 필요한 메타데이터(`$id`, `$schema`)를 자동으로 보강하여 반환한다.
