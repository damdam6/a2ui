# agent_sdks/python/a2ui_agent/src/a2ui/a2a/extension.py

## 개요

A2A(Agent-to-Agent) 프로토콜에서 A2UI 확장 기능을 협상하는 유틸리티 모듈이다. 에이전트 카드에 A2UI 확장을 등록하는 `AgentExtension` 객체 생성, 클라이언트가 요청한 확장과 에이전트가 지원하는 확장을 비교해 최적(최신) 버전을 선택, 그리고 선택된 확장을 요청 컨텍스트에 활성화하는 기능을 제공한다.

## 의존성

### 외부 패키지
- `logging`
- `typing` — `Optional`, `List`
- `a2a.server.agent_execution.RequestContext`
- `a2a.types` — `AgentCard`, `AgentExtension`
- `packaging.version` — `parse as parse_version` (내부 함수에서 지연 import)

## Exports

- `A2UI_EXTENSION_BASE_URI` (상수)
- `AGENT_EXTENSION_SUPPORTED_CATALOG_IDS_KEY` (상수)
- `AGENT_EXTENSION_ACCEPTS_INLINE_CATALOGS_KEY` (상수)
- `get_a2ui_agent_extension` (함수)
- `try_activate_a2ui_extension` (함수)

비공개(모듈 내부 전용):
- `_agent_extensions` (함수)
- `_requested_a2ui_extensions` (함수)
- `_select_newest_a2ui_extension` (함수)

## 상세 명세

### 상수

- `A2UI_EXTENSION_BASE_URI: str = "https://a2ui.org/a2a-extension/a2ui"` — A2UI 확장의 기본 URI 접두사.
- `AGENT_EXTENSION_SUPPORTED_CATALOG_IDS_KEY: str = "supportedCatalogIds"` — AgentExtension params의 지원 카탈로그 ID 키 이름.
- `AGENT_EXTENSION_ACCEPTS_INLINE_CATALOGS_KEY: str = "acceptsInlineCatalogs"` — AgentExtension params의 인라인 카탈로그 수락 여부 키 이름.

### `get_a2ui_agent_extension(version: str, accepts_inline_catalogs: bool = False, supported_catalog_ids: List[str] = []) -> AgentExtension`

에이전트 카드에 등록할 `AgentExtension` 객체를 생성한다.

- `params` 딕셔너리를 빈 상태로 시작한다.
- `accepts_inline_catalogs`가 `True`이면 `"acceptsInlineCatalogs": True`를 params에 추가한다(기본값 `False`일 때는 생략해 페이로드를 최소화).
- `supported_catalog_ids`가 비어있지 않으면 `"supportedCatalogIds": [...]`를 params에 추가한다.
- `uri`는 `"{A2UI_EXTENSION_BASE_URI}/v{version}"` 형태로 구성된다.
- `description`은 `"Provides agent driven UI using the A2UI JSON format."`으로 고정된다.
- `params`가 비어있으면 `None`을 전달한다.

### `_agent_extensions(agent_card: AgentCard) -> List[str]` (비공개)

에이전트 카드에 등록된 확장 URI 중 `A2UI_EXTENSION_BASE_URI`로 시작하는 것들만 추출해 리스트로 반환한다. `agent_card`, `capabilities`, `extensions` 중 어느 하나라도 `None`이거나 해당 속성이 없으면 빈 리스트를 반환한다.

### `_requested_a2ui_extensions(context: RequestContext) -> List[str]` (비공개)

요청 컨텍스트에서 클라이언트가 요청한 A2UI 확장 URI 목록을 수집한다.

1. `context.requested_extensions`에서 `A2UI_EXTENSION_BASE_URI`로 시작하는 문자열 항목을 추가한다.
2. `context.message.extensions`에서도 동일 조건으로 추가한다.
3. 두 소스가 없거나 `None`이면 빈 리스트를 반환한다.

### `_select_newest_a2ui_extension(requested_extensions: List[str], agent_advertised_extensions: List[str]) -> Optional[str]` (비공개)

요청된 확장 URI와 에이전트가 광고하는 확장 URI의 교집합을 구하고, 그 중 버전이 가장 높은 URI를 반환한다.

- 교집합이 없으면 `None`을 반환한다.
- 버전 비교를 위해 `_version_key` 내부 함수를 정의한다: URI에서 `"{A2UI_EXTENSION_BASE_URI}/v"` 접두사를 제거한 문자열을 `packaging.version.parse`로 파싱한다.
- `max(matched_extensions, key=_version_key)`로 가장 높은 버전의 URI를 선택한다.

### `try_activate_a2ui_extension(context: RequestContext, agent_card: AgentCard) -> Optional[str]`

A2UI 확장 협상의 진입점. 클라이언트가 요청했고 에이전트가 지원하는 A2UI 확장이 있으면 활성화하고 버전 문자열을 반환한다.

1. `_requested_a2ui_extensions(context)`로 요청 목록을 얻는다. 비어있으면 `None` 반환.
2. `_agent_extensions(agent_card)`로 에이전트 지원 목록을 얻는다. 비어있으면 `None` 반환.
3. `_select_newest_a2ui_extension`으로 최신 공통 URI를 선택한다.
4. URI가 선택되면 `context.add_activated_extension(selected_uri)`를 호출하고, URI에서 `"{A2UI_EXTENSION_BASE_URI}/v"` 접두사를 제거한 버전 문자열을 반환한다.
5. 공통 URI가 없으면 `None` 반환.

## 동작 흐름

에이전트 초기화 시 `get_a2ui_agent_extension`으로 에이전트 카드에 확장을 등록하고, 각 요청 처리 시 `try_activate_a2ui_extension`을 호출해 클라이언트와 에이전트 간의 A2UI 확장 버전을 협상한다. 협상이 성공하면 반환된 버전 문자열을 이후 A2UI 파트 생성에 활용한다.
