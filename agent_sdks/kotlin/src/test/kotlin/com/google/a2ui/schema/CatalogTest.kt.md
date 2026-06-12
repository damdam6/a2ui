# agent_sdks/kotlin/src/test/kotlin/com/google/a2ui/schema/CatalogTest.kt

## 개요

`A2uiCatalog`, `CatalogConfig`, `BasicCatalog` 관련 경로 해석 로직의 단위 테스트 클래스다. 예제 경로 처리 함수 `resolveExamplesPath`, `CatalogConfig.fromPath` 팩토리 메서드의 URL 스킴 처리 동작, 그리고 `BasicCatalog.getConfig`의 examples 경로 전달을 검증한다.

## 의존성

### 외부 패키지
- `kotlin.test.*` (Test, assertEquals, assertFailsWith, assertNull, assertTrue)

### 저장소 내부 모듈
- `com.google.a2ui.basic_catalog.BasicCatalog` — 기본 카탈로그 제공 객체
- `com.google.a2ui.schema.A2uiVersion` — 버전 열거형
- `com.google.a2ui.schema.CatalogConfig` — 카탈로그 설정 데이터 클래스 및 팩토리
- `com.google.a2ui.schema.FileSystemCatalogProvider` — 파일시스템 기반 카탈로그 제공자
- `resolveExamplesPath` — 동일 패키지 내 최상위 함수 (import 불필요)

## Exports

- `class CatalogTest` — JUnit 단위 테스트 클래스

## 테스트 케이스 명세

### `resolvesExamplesPathHandling()`

**검증 동작**: `resolveExamplesPath` 함수가 다양한 입력 형식을 올바르게 처리하는지 확인한다.

**픽스처**: 없음 (순수 함수 테스트).

**절차**:
1. `resolveExamplesPath(null)` → `assertNull` (null 입력 시 null 반환).
2. `resolveExamplesPath("/absolute/examples")` → `assertEquals("/absolute/examples", ...)` (절대 경로는 그대로 반환).
3. `resolveExamplesPath("file:///absolute/examples")` → `assertEquals("/absolute/examples", ...)` (`file://` 스킴 제거 후 반환).
4. `resolveExamplesPath("https://a2ui.org/examples")` → `assertFailsWith<IllegalArgumentException>`, 메시지에 `"Unsupported examples URL scheme"` 포함 단언.

---

### `stripsPrefixFromCatalogConfigFromPathSchemes()`

**검증 동작**: `CatalogConfig.fromPath`가 다양한 URL 스킴을 올바르게 처리하는지 확인한다.

**픽스처**: 없음.

**절차**:
1. 로컬 상대 경로 `"relative_path/to/catalog.json"` → `config.provider`가 `FileSystemCatalogProvider`이고 `.path`가 `"relative_path/to/catalog.json"` 임을 단언.
2. `file:///` 스킴 `"file:///absolute_path/to/catalog.json"` → `.path`가 `"/absolute_path/to/catalog.json"` (스킴 제거) 임을 단언.
3. `http://` 스킴 → `assertFailsWith<NotImplementedError>`, 메시지에 `"HTTP support is coming soon."` 포함 단언.
4. `ftp://` 스킴 → `assertFailsWith<IllegalArgumentException>`, 메시지에 `"Unsupported catalog URL scheme"` 포함 단언.

---

### `resolvesExamplesPathInBasicCatalogGetConfig()`

**검증 동작**: `BasicCatalog.getConfig`가 `examplesPath`로 `file://` 스킴 경로를 받았을 때 내부적으로 `resolveExamplesPath`를 통해 스킴을 제거한 순수 경로를 `config.examplesPath`로 저장하는지 확인한다.

**픽스처**: 없음.

**절차**:
1. `BasicCatalog.getConfig(version=A2uiVersion.VERSION_0_9, examplesPath="file:///absolute/examples")` 호출.
2. `config.examplesPath`가 `"/absolute/examples"` 임을 단언.

## 동작 흐름

세 테스트 모두 실제 파일시스템을 사용하지 않고 경로 문자열 변환 로직만 검증한다. 각 테스트는 독립적이며 공유 픽스처가 없다. 예외 케이스는 `assertFailsWith`로 검증하고 메시지 내용도 확인한다.
