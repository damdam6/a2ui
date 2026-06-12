# agent_sdks/python/a2ui_core/src/a2ui/core/basic_catalog/function_apis.py

## 개요

자동 생성된 파일로, basic catalog에서 지원하는 내장 함수들의 API 명세(이름, 인수 스키마, 반환 타입)를 Pydantic 기반으로 선언한다. 각 함수는 `Args` 모델 클래스와 `Api` 클래스 쌍으로 구성되며, `Api` 클래스는 `FunctionApi`를 상속해 함수의 메타데이터를 클래스 속성으로 보유한다. 이 파일은 직접 수정하지 않는다.

## 의존성

### 외부 패키지
- `typing`: `Any`, `Dict`, `List`, `Literal`, `Optional`, `Union`, `Annotated`
- `pydantic`: `BaseModel`, `Field`, `ConfigDict`

### 저장소 내부 모듈
- [`../schema/common_types.py`](../schema/common_types.py.md): `StrictBaseModel`, `DynamicString`, `DynamicNumber`, `DynamicBoolean`, `DynamicValue`, `DynamicStringList`
- [`../catalog/functions.py`](../catalog/functions.py.md): `FunctionApi`

## Exports

각 항목은 `(Args 클래스, Api 클래스)` 쌍으로 export됨.

| Args 클래스 | Api 클래스 | `name` | `return_type` |
|---|---|---|---|
| `RequiredArgs` | `RequiredApi` | `"required"` | `"boolean"` |
| `RegexArgs` | `RegexApi` | `"regex"` | `"boolean"` |
| `LengthArgs` | `LengthApi` | `"length"` | `"boolean"` |
| `NumericArgs` | `NumericApi` | `"numeric"` | `"boolean"` |
| `EmailArgs` | `EmailApi` | `"email"` | `"boolean"` |
| `FormatStringArgs` | `FormatStringApi` | `"formatString"` | `"string"` |
| `FormatNumberArgs` | `FormatNumberApi` | `"formatNumber"` | `"string"` |
| `FormatCurrencyArgs` | `FormatCurrencyApi` | `"formatCurrency"` | `"string"` |
| `FormatDateArgs` | `FormatDateApi` | `"formatDate"` | `"string"` |
| `PluralizeArgs` | `PluralizeApi` | `"pluralize"` | `"string"` |
| `OpenUrlArgs` | `OpenUrlApi` | `"openUrl"` | `"void"` |
| `AndArgs` | `AndApi` | `"and"` | `"boolean"` |
| `OrArgs` | `OrApi` | `"or"` | `"boolean"` |
| `NotArgs` | `NotApi` | `"not"` | `"boolean"` |

## 상세 명세

각 함수 API는 `Args` 클래스(인수 스키마)와 `Api` 클래스(메타데이터)로 이루어진다. `Api` 클래스는 `FunctionApi`를 상속하며 `name`, `args`, `return_type`을 클래스 속성으로 선언한다.

---

### `required` — 필수 값 검증

**`RequiredArgs(StrictBaseModel)`**
- `value: Any` — 검사할 값 (필수).

**`RequiredApi(FunctionApi)`**
- `name = "required"`, `args = RequiredArgs`, `return_type = "boolean"`
- 의미: 값이 `None`, 빈 문자열 `""`, 빈 리스트 `[]`가 아닌지 확인.

---

### `regex` — 정규식 매칭 검증

**`RegexArgs(StrictBaseModel)`**
- `value: DynamicString` — 검사할 문자열 (필수).
- `pattern: str` — 매칭할 정규식 패턴 (필수).

**`RegexApi(FunctionApi)`**
- `name = "regex"`, `args = RegexArgs`, `return_type = "boolean"`

---

### `length` — 문자열 길이 범위 검증

**`LengthArgs(StrictBaseModel)`**
- `value: DynamicString` — 검사할 문자열 (필수).
- `min: Optional[int] = None` — 최소 허용 길이.
- `max: Optional[int] = None` — 최대 허용 길이.

**`LengthApi(FunctionApi)`**
- `name = "length"`, `args = LengthArgs`, `return_type = "boolean"`

---

### `numeric` — 숫자 범위 검증

**`NumericArgs(StrictBaseModel)`**
- `value: DynamicNumber` — 검사할 숫자 (필수).
- `min: Optional[float] = None` — 최소 허용 값.
- `max: Optional[float] = None` — 최대 허용 값.

**`NumericApi(FunctionApi)`**
- `name = "numeric"`, `args = NumericArgs`, `return_type = "boolean"`

---

### `email` — 이메일 형식 검증

**`EmailArgs(StrictBaseModel)`**
- `value: DynamicString` — 검사할 문자열 (필수).

**`EmailApi(FunctionApi)`**
- `name = "email"`, `args = EmailArgs`, `return_type = "boolean"`

---

### `formatString` — 템플릿 문자열 포맷

**`FormatStringArgs(StrictBaseModel)`**
- `value: DynamicString` — `${...}` 보간이 포함된 템플릿 문자열 (필수).

**`FormatStringApi(FunctionApi)`**
- `name = "formatString"`, `args = FormatStringArgs`, `return_type = "string"`

---

### `formatNumber` — 숫자 지역화 포맷

**`FormatNumberArgs(StrictBaseModel)`**
- `value: DynamicNumber` — 포맷할 숫자 (필수).
- `decimals: Optional[DynamicNumber] = None` — 소수점 자릿수. 기본값은 로케일에 따라 0 또는 2.
- `grouping: Optional[DynamicBoolean] = None` — true이면 천 단위 구분자 적용 (예: `1,000`). 기본 true.

**`FormatNumberApi(FunctionApi)`**
- `name = "formatNumber"`, `args = FormatNumberArgs`, `return_type = "string"`

---

### `formatCurrency` — 통화 포맷

**`FormatCurrencyArgs(StrictBaseModel)`**
- `value: DynamicNumber` — 금액 (필수).
- `currency: DynamicString` — ISO 4217 통화 코드 (예: `"USD"`, `"EUR"`) (필수).
- `decimals: Optional[DynamicNumber] = None` — 소수점 자릿수. 기본 0 또는 2.
- `grouping: Optional[DynamicBoolean] = None` — 천 단위 구분자 사용 여부. 기본 true.

**`FormatCurrencyApi(FunctionApi)`**
- `name = "formatCurrency"`, `args = FormatCurrencyArgs`, `return_type = "string"`

---

### `formatDate` — 날짜/시간 포맷

**`FormatDateArgs(StrictBaseModel)`**
- `value: DynamicValue` — 포맷할 날짜 값 (필수).
- `format: DynamicString` — Unicode TR35 날짜 패턴 문자열 (필수). 토큰 참조:
  - 연도: `yy` (26), `yyyy` (2026)
  - 월: `M` (1), `MM` (01), `MMM` (Jan), `MMMM` (January)
  - 일: `d` (1), `dd` (01), `E` (Tue), `EEEE` (Tuesday)
  - 시(12h): `h`, `hh` — `a`(AM/PM)와 함께 사용
  - 시(24h): `H`, `HH`
  - 분: `mm`, 초: `ss`, AM/PM: `a`
  - 예: `"MMM dd, yyyy"` → `"Jan 16, 2026"`, `"HH:mm"` → `"14:30"`

**`FormatDateApi(FunctionApi)`**
- `name = "formatDate"`, `args = FormatDateArgs`, `return_type = "string"`

---

### `pluralize` — 복수형 처리

**`PluralizeArgs(StrictBaseModel)`**
- `value: DynamicNumber` — 복수 카테고리 결정에 사용하는 숫자 (필수).
- `zero: Optional[DynamicString] = None` — 값이 0일 때 문자열.
- `one: Optional[DynamicString] = None` — 값이 1일 때 문자열.
- `two: Optional[DynamicString] = None` — 값이 2일 때 문자열 (아랍어, 웨일스어 등).
- `few: Optional[DynamicString] = None` — `few` 카테고리 문자열 (슬라브어 계열 소수).
- `many: Optional[DynamicString] = None` — `many` 카테고리 문자열.
- `other: DynamicString` — 기본/폴백 문자열 (필수).

**`PluralizeApi(FunctionApi)`**
- `name = "pluralize"`, `args = PluralizeArgs`, `return_type = "string"`

---

### `openUrl` — URL 열기

**`OpenUrlArgs(StrictBaseModel)`**
- `url: str` — 열 URL (필수).

**`OpenUrlApi(FunctionApi)`**
- `name = "openUrl"`, `args = OpenUrlArgs`, `return_type = "void"`

---

### `and` — 논리 AND

**`AndArgs(StrictBaseModel)`**
- `values: List[DynamicBoolean]` — 평가할 불린 값 목록 (필수).

**`AndApi(FunctionApi)`**
- `name = "and"`, `args = AndArgs`, `return_type = "boolean"`

---

### `or` — 논리 OR

**`OrArgs(StrictBaseModel)`**
- `values: List[DynamicBoolean]` — 평가할 불린 값 목록 (필수).

**`OrApi(FunctionApi)`**
- `name = "or"`, `args = OrArgs`, `return_type = "boolean"`

---

### `not` — 논리 NOT

**`NotArgs(StrictBaseModel)`**
- `value: DynamicBoolean` — 부정할 불린 값 (필수).

**`NotApi(FunctionApi)`**
- `name = "not"`, `args = NotArgs`, `return_type = "boolean"`

## 동작 흐름

이 파일은 순수 스키마 선언 파일이다. 모듈 로드 시 14쌍의 `Args`/`Api` 클래스가 정의되고, `FunctionApi.__init_subclass__` 훅에 의해 `name`, `return_type`, `schema(=args)` 속성이 클래스 수준에서 자동으로 정규화된다. 실제 실행 로직은 없으며, `function_impls.py`에서 각 `Api` 클래스를 `create_function_implementation()`에 전달하여 실행 가능한 구현체로 조합한다.
