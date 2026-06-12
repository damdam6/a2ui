# renderers/web_core/src/v0_8/types/components.ts

## 개요

A2UI가 지원하는 모든 UI 컴포넌트의 TypeScript 인터페이스를 Zod 스키마로부터 도출하여 내보내는 파일이다. 각 인터페이스는 `z.infer<typeof XxxSchema>`로 스키마와 동기화되므로, 런타임 유효성 검사(Zod)와 정적 타입 시스템이 동일한 형태를 공유한다. 런타임 로직은 없으며 타입 선언만 포함한다.

## 의존성

- 외부 패키지: `zod` (`z` 타입 임포트)
- 저장소 내부 모듈:
  - `../schema/common-types.ts` — 아래 19개 스키마를 타입 임포트

## Exports

| 이름 | 종류 | 대응 스키마 |
|------|------|------|
| `Action` | 인터페이스 | `ActionSchema` |
| `Text` | 인터페이스 | `TextSchema` |
| `Image` | 인터페이스 | `ImageSchema` |
| `Icon` | 인터페이스 | `IconSchema` |
| `Video` | 인터페이스 | `VideoSchema` |
| `AudioPlayer` | 인터페이스 | `AudioPlayerSchema` |
| `Tabs` | 인터페이스 | `TabsSchema` |
| `Row` | 인터페이스 | `RowSchema` |
| `Column` | 인터페이스 | `ColumnSchema` |
| `List` | 인터페이스 | `ListSchema` |
| `Button` | 인터페이스 | `ButtonSchema` |
| `Modal` | 인터페이스 | `ModalSchema` |
| `Card` | 인터페이스 | `CardSchema` |
| `Divider` | 인터페이스 | `DividerSchema` |
| `TextField` | 인터페이스 | `TextFieldSchema` |
| `Checkbox` | 인터페이스 | `CheckboxSchema` |
| `DateTimeInput` | 인터페이스 | `DateTimeInputSchema` |
| `MultipleChoice` | 인터페이스 | `MultipleChoiceSchema` |
| `Slider` | 인터페이스 | `SliderSchema` |

## 상세 명세

각 인터페이스는 패턴 `export declare interface XxxComponent extends z.infer<typeof XxxSchema> {}` 형태로 선언된다. 본체는 비어 있으며 모든 필드는 대응하는 Zod 스키마(`../schema/common-types.ts`)에서 추론된다.

- `Action`: 컴포넌트에 부착되는 액션 정의 타입
- `Text`: 텍스트 컴포넌트 속성
- `Image`: 이미지 컴포넌트 속성
- `Icon`: 아이콘 컴포넌트 속성
- `Video`: 비디오 컴포넌트 속성
- `AudioPlayer`: 오디오 플레이어 컴포넌트 속성
- `Tabs`: 탭 컴포넌트 속성
- `Row`: 가로 배치 컨테이너 컴포넌트 속성
- `Column`: 세로 배치 컨테이너 컴포넌트 속성
- `List`: 목록 컴포넌트 속성
- `Button`: 버튼 컴포넌트 속성
- `Modal`: 모달 컴포넌트 속성
- `Card`: 카드 컴포넌트 속성
- `Divider`: 구분선 컴포넌트 속성
- `TextField`: 텍스트 입력 필드 속성
- `Checkbox`: 체크박스 컴포넌트 속성
- `DateTimeInput`: 날짜/시간 입력 컴포넌트 속성
- `MultipleChoice`: 복수 선택 컴포넌트 속성
- `Slider`: 슬라이더 컴포넌트 속성

## 동작 흐름

이 파일은 컴파일 타임 전용이다. 임포트 시 `zod`와 `common-types.ts`에서 타입 정보만 참조하며 어떠한 런타임 코드도 실행하지 않는다. 컴포넌트 속성 타입이 필요한 모든 곳에서 이 파일로부터 해당 인터페이스를 임포트하면 Zod 스키마와 자동으로 동기화된 타입 안정성을 얻는다.
