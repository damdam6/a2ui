# agent_sdks/python/a2ui_agent/src/a2ui/schema/catalog_provider.py

## 개요

A2UI 카탈로그 스키마를 로드하는 추상 인터페이스(`A2uiCatalogProvider`)와 파일시스템 기반 구현체(`FileSystemCatalogProvider`)를 정의한다. 다양한 저장소(로컬 파일, 향후 원격 등)에서 카탈로그를 로드하는 전략을 통일된 인터페이스로 추상화하는 역할을 한다.

## 의존성

### 외부 패키지
- `json` — JSON 파일 파싱
- `abc` — `ABC`, `abstractmethod`
- `json.decoder` — `JSONDecodeError`
- `typing` — `Any`, `Dict`

### 저장소 내부 모듈
- [`./constants`](./constants.py.md) — `ENCODING`

## Exports

- 추상 클래스: `A2uiCatalogProvider`
- 클래스: `FileSystemCatalogProvider`

## 상세 명세

### 추상 클래스 `A2uiCatalogProvider(ABC)`

카탈로그 로드 전략의 추상 기반 클래스이다.

#### 추상 메서드 `load(self) -> Dict[str, Any]`

카탈로그 정의를 로드하여 딕셔너리로 반환한다. 구현체는 이 메서드를 반드시 오버라이드해야 한다.

### 클래스 `FileSystemCatalogProvider(A2uiCatalogProvider)`

로컬 파일시스템에서 카탈로그를 로드하는 구현체이다.

#### `__init__(self, path: str)`

`self.path = path`로 파일 경로를 저장한다.

#### `load(self) -> Dict[str, Any]`

`self.path`를 `ENCODING`(`"utf-8"`) 인코딩으로 열어 `json.load()`로 파싱한다. `FileNotFoundError` 또는 `JSONDecodeError` 발생 시 `IOError`로 감싸 `f"Could not load schema from {self.path}: {e}"` 메시지와 함께 재발생시킨다.

## 동작 흐름

`FileSystemCatalogProvider(path)`를 생성하고 `load()`를 호출하면 지정 경로의 JSON 파일을 읽어 딕셔너리로 반환한다. 파일 미존재 또는 JSON 파싱 오류 시 `IOError`가 발생한다.
