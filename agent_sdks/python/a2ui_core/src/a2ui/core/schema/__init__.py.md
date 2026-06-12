# agent_sdks/python/a2ui_core/src/a2ui/core/schema/__init__.py

## 개요

`schema` 패키지의 공개 API를 단일 네임스페이스로 집약한다. `common_types`, `constants`, `server_to_client`, `client_capabilities`, `client_to_server` 다섯 하위 모듈에서 선별한 심볼을 재노출하며 별도 로직은 없다. `constants`는 와일드카드(`*`) import를 사용해 모든 상수를 노출한다.

## 의존성

### 저장소 내부 모듈
- [`./common_types.py`](common_types.py.md) — `StrictBaseModel`, `DataBinding`, `FunctionCall`, `AccessibilityAttributes`, `CheckRule`, `ActionEvent`, `Action`, `ComponentCommon`
- [`./constants.py`](constants.py.md) — `*` (모든 상수)
- `./server_to_client.py` — `CreateSurfaceMessage`, `CreateSurface`, `UpdateComponentsMessage`, `UpdateComponents`, `UpdateDataModelMessage`, `UpdateDataModel`, `DeleteSurfaceMessage`, `DeleteSurface`, `A2uiMessage`, `A2uiMessageListWrapper`
- [`./client_capabilities.py`](client_capabilities.py.md) — `A2uiClientCapabilities`, `V09Capabilities`, `InlineCatalog`, `FunctionDefinition`
- [`./client_to_server.py`](client_to_server.py.md) — `A2uiClientMessage`, `A2uiClientActionMessage`, `A2uiClientErrorMessage`, `A2uiClientAction`, `A2uiValidationError`, `A2uiGenericError`, `A2uiClientError`, `A2uiClientDataModel`, `A2uiClientMessageList`, `A2uiClientMessageListWrapper`

## Exports

`common_types`에서:
- `StrictBaseModel`, `DataBinding`, `FunctionCall`, `AccessibilityAttributes`, `CheckRule`, `ActionEvent`, `Action`, `ComponentCommon`

`constants`에서 (`*`):
- `SPEC_VERSION`, `SPEC_BASE_URL`, `MSG_TYPE_CREATE_SURFACE`, `MSG_TYPE_UPDATE_COMPONENTS`, `MSG_TYPE_UPDATE_DATA_MODEL`, `MSG_TYPE_DELETE_SURFACE`, `CATALOG_COMPONENTS_KEY`, `SURFACE_ID_KEY`, `THEME_KEY`, `ROOT_ID`

`server_to_client`에서:
- `CreateSurfaceMessage`, `CreateSurface`, `UpdateComponentsMessage`, `UpdateComponents`, `UpdateDataModelMessage`, `UpdateDataModel`, `DeleteSurfaceMessage`, `DeleteSurface`, `A2uiMessage`, `A2uiMessageListWrapper`

`client_capabilities`에서:
- `A2uiClientCapabilities`, `V09Capabilities`, `InlineCatalog`, `FunctionDefinition`

`client_to_server`에서:
- `A2uiClientMessage`, `A2uiClientActionMessage`, `A2uiClientErrorMessage`, `A2uiClientAction`, `A2uiValidationError`, `A2uiGenericError`, `A2uiClientError`, `A2uiClientDataModel`, `A2uiClientMessageList`, `A2uiClientMessageListWrapper`

## 동작 흐름

import 선언만으로 구성된 집약 파일이다. 사용자는 `from a2ui.core.schema import StrictBaseModel` 등으로 하위 모듈 경로 없이 직접 접근할 수 있다.
