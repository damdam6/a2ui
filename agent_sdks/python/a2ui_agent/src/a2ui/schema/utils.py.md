# agent_sdks/python/a2ui_agent/src/a2ui/schema/utils.py

## 개요

A2UI 스키마 조작에 필요한 범용 유틸리티 함수들을 모아둔 모듈이다. 번들된 패키지 리소스, 로컬 assets 디렉토리, 소스 저장소 순으로 폴백하며 스키마 파일을 로드하는 3단계 로딩 전략을 구현한다. 또한 스키마 dict를 배열 형태로 감싸거나, dict를 재귀적으로 병합하는 헬퍼 함수를 제공한다.

## 의존성

### 외부 패키지
- `json` (표준 라이브러리)
- `logging` (표준 라이브러리)
- `os` (표준 라이브러리)
- `importlib.resources` (표준 라이브러리)
- `typing.Any`, `typing.Dict`

### 저장소 내부 모듈
- [`./constants`](./constants.py.md) — `A2UI_ASSET_PACKAGE`, `SPECIFICATION_DIR`, `ENCODING` 상수
- [`./catalog_provider`](./catalog_provider.py.md) — `FileSystemCatalogProvider` 클래스

## Exports

- `find_repo_root` (함수)
- `load_from_bundled_resource` (함수)
- `wrap_as_json_array` (함수)
- `deep_update` (함수)

## 상세 명세

### `find_repo_root(start_path: str) -> str | None`

`start_path`를 절대 경로로 변환한 후, `SPECIFICATION_DIR` 이름을 가진 디렉토리가 존재하는 상위 디렉토리를 찾을 때까지 부모 경로를 순차적으로 올라간다. 파일시스템의 루트(`parent == current`)에 도달하면 `None`을 반환한다. 저장소 루트를 찾으면 해당 경로 문자열을 반환한다.

---

### `load_from_bundled_resource(version: str, resource_key: str, spec_map: Dict[str, Dict[str, str]]) -> Dict[str, Any]`

지정한 `version`과 `resource_key`에 해당하는 스키마 JSON을 3단계 전략으로 로드한다.

**단계 1 — 번들 패키지 리소스:**
`spec_map`에서 `version`에 해당하는 내부 맵을 꺼낸다. `version`이 존재하지 않으면 `ValueError`를 발생시킨다. `resource_key`가 맵에 없으면 `None`을 반환한다. 키가 존재하면 상대 경로(`rel_path`)와 파일명(`filename`)을 추출하고, `importlib.resources.files(A2UI_ASSET_PACKAGE)`로 번들 리소스를 탐색해 `<version>/<filename>` 경로의 파일을 `ENCODING`으로 열어 JSON으로 파싱해 반환한다. 실패하면 `logging.debug`로 기록하고 다음 단계로 넘어간다.

**단계 2 — 로컬 assets 디렉토리 폴백:**
현재 파일(`__file__`)에서 세 단계 상위로 이동해 `assets/<version>/<filename>` 경로를 구성한다. 해당 경로가 존재하면 `FileSystemCatalogProvider`를 생성해 `.load()`로 반환한다. 실패하면 `logging.debug`로 기록하고 다음 단계로 넘어간다.

**단계 3 — 소스 저장소 폴백:**
`find_repo_root`로 저장소 루트를 탐색하고, `<repo_root>/<rel_path>` 경로가 존재하면 `FileSystemCatalogProvider`로 로드해 반환한다. 이 단계도 실패하면 `IOError`를 발생시킨다.

---

### `wrap_as_json_array(a2ui_schema: dict[str, Any]) -> dict[str, Any]`

LLM이 메시지 목록을 생성하도록 지시받으므로, 단일 스키마 오브젝트를 배열 스키마로 감싼다. `a2ui_schema`가 비어 있으면 `ValueError`를 발생시킨다. 정상 입력의 경우 `{"type": "array", "items": a2ui_schema}` 형태의 dict를 반환한다.

---

### `deep_update(d: dict, u: dict) -> dict`

`u`의 모든 키를 순회하며 `d`를 재귀적으로 갱신한다. 값이 `dict`이면 `d.get(k, {})`에 대해 재귀 호출하고, 그렇지 않으면 단순 대입한다. 변경된 `d`를 반환한다.

## 동작 흐름

`load_from_bundled_resource`가 이 모듈의 핵심 진입점이다. 호출자는 버전 문자열과 리소스 키를 제공하고, 함수는 패키지 번들 → 로컬 assets → 소스 저장소 순으로 스키마 JSON을 탐색한다. `wrap_as_json_array`는 로드된 스키마를 jsonschema 검증에 적합한 배열 래퍼로 변환할 때 사용된다. `deep_update`는 두 스키마 dict를 병합할 때 재귀적으로 적용된다.
