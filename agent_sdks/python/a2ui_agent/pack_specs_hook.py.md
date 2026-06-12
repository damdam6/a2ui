# agent_sdks/python/a2ui_agent/pack_specs_hook.py

## 개요

Hatchling 빌드 시스템을 위한 커스텀 빌드 훅으로, Python 패키지를 빌드할 때 저장소 루트의 JSON 스펙 파일(스키마 및 기본 카탈로그)을 `src/a2ui/assets/` 디렉토리로 복사하는 역할을 한다. 빌드 시점에만 실행되며, 소스 코드의 `constants.py`와 `utils.py`를 동적으로 로드해 경로 정보를 얻는다. 빌드 환경이 sdist(PKG-INFO 존재)인 경우에는 이미 번들된 에셋이 있다고 가정하고 복사를 생략한다.

## 의존성

### 외부 패키지
- `importlib.util` — 파일 경로 기반 모듈 동적 로드
- `os`, `sys`, `shutil` — 파일 시스템 조작
- `hatchling.builders.hooks.plugin.interface.BuildHookInterface` — Hatchling 빌드 훅 기반 클래스

### 저장소 내부 모듈 (동적 로드, 정적 import 문 없음)
- `a2ui.schema.constants` — `SPEC_VERSION_MAP`, `A2UI_ASSET_PACKAGE`, `SPECIFICATION_DIR` 상수 제공
- `a2ui.schema.utils` — `find_repo_root` 유틸리티 제공
- `a2ui.basic_catalog.constants` — `BASIC_CATALOG_PATHS` 상수 제공

## Exports

- `load_module` (함수)
- `PackSpecsBuildHook` (클래스)

## 상세 명세

### `load_module(project_root: str, rel_path: str, filename: str, module_name: str) -> module`

파일 시스템 경로로부터 Python 모듈을 직접 로드하는 헬퍼 함수.

- `project_root`와 `rel_path`(`.`을 os 구분자로 변환), `filename`을 결합해 절대 경로를 구성한다(`project_root/src/<rel_path>/<filename>`).
- 해당 경로가 존재하지 않으면 `RuntimeError`를 발생시킨다.
- `project_root/src`를 `sys.path` 맨 앞에 추가해 절대 import가 작동하도록 한다(중복 추가 방지).
- `importlib.util.spec_from_file_location`으로 모듈 스펙을 생성하고, `module.__package__`를 `rel_path`로 설정해 상대 import 컨텍스트를 부여한다.
- `sys.modules`에 `module_name` 키로 등록한 뒤 `spec.loader.exec_module(module)`로 실행한 후 모듈을 반환한다.
- 스펙이나 로더가 없으면 `RuntimeError`를 발생시킨다.

### `PackSpecsBuildHook(BuildHookInterface)`

Hatchling 빌드 훅 클래스. `initialize` 메서드만 오버라이드한다.

#### `initialize(self, version: str, build_data: dict) -> None`

빌드 초기화 단계에서 호출된다.

1. `load_module`을 통해 `a2ui.schema.constants`, `a2ui.schema.utils`, `a2ui.basic_catalog.constants` 세 모듈을 동적으로 로드한다.
2. 로드된 모듈에서 `SPEC_VERSION_MAP`, `A2UI_ASSET_PACKAGE`, `SPECIFICATION_DIR` 및 `BASIC_CATALOG_PATHS` 값을 추출한다.
3. `a2ui_utils.find_repo_root(project_root)`를 호출해 저장소 루트를 찾는다.
   - 저장소 루트를 찾지 못하고 `PKG-INFO` 파일이 존재하면 sdist 상태로 판단하고 함수를 조기 반환한다.
   - `PKG-INFO`도 없으면 `RuntimeError`를 발생시킨다.
4. `target_base`를 `project_root/src/<A2UI_ASSET_PACKAGE(점을 슬래시로 변환)>`로 설정한다.
5. `_pack_schemas`와 `_pack_basic_catalogs`를 순서대로 호출한다.

#### `_pack_schemas(self, repo_root: str, spec_map: dict, target_base: str) -> None`

`spec_map`의 각 버전 키에 대해 `target_base/<ver>` 디렉토리를 생성하고(`exist_ok=True`), 해당 버전의 모든 스키마 파일을 `_copy_schema`로 복사한다.

#### `_pack_basic_catalogs(self, repo_root: str, catalog_paths: dict, target_base: str) -> None`

`catalog_paths`의 각 버전 키에 대해 `target_base/<ver>` 디렉토리를 생성하고, 해당 버전의 모든 카탈로그 파일을 `_copy_schema`로 복사한다.

#### `_copy_schema(self, repo_root: str, source_rel_path: str, target_dir: str) -> None`

`repo_root/<source_rel_path>` 소스 파일을 `target_dir/<filename>`으로 복사한다(`shutil.copy2` 사용). 소스 파일이 존재하지 않으면 경고 메시지를 출력하고 반환한다(에러 미발생).

## 동작 흐름

빌드 시 Hatchling이 `PackSpecsBuildHook.initialize`를 호출하면, 저장소 내부의 상수 모듈들을 런타임에 동적으로 로드하여 경로 맵을 얻는다. 이후 저장소 루트에서 버전별 JSON 스키마와 카탈로그 파일을 찾아 패키지 내 `assets` 디렉토리로 복사함으로써, 배포된 wheel에 스펙 파일이 포함되도록 보장한다.
