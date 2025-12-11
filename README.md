# Broken Filename Fixer

> Fix broken filenames when downloading files from legacy or misconfigured servers.

많은 오래된 웹사이트와 레거시 서버는 아직도 EUC-KR / CP949 같은 인코딩을 사용하거나, 잘못된 방식으로 파일명을 전송합니다.  
그 결과, 브라우저에서 파일을 다운로드할 때:

- `¾ÆÄ«´©¸® Áö¿ø¼_(¹æÇÐ).docx` 처럼 깨지거나  
- `&#4359;&#4469;&#4520;...` 같은 HTML 숫자 엔티티로 보이거나  
- 윈도우에서는 한글 자모가 분리된 상태(ㅅㅣㄴㅊㅓㅇㅅㅓ)로 저장되는 문제가 발생합니다.

**Broken Filename Fixer**는 이러한 다운로드 파일명을 자동으로 감지하고, 사람이 읽을 수 있는 자연스러운 파일명으로 복원해 주는 브라우저 확장입니다.

---

## Features

- EUC-KR / CP949 기반 모지바케(mojibake) 파일명 복원
- `&#1234;` 형태의 HTML 숫자 엔티티 파일명 → 유니코드 문자열로 복원
- 분리된 한글 자모(NFD) → 완성형 한글(NFC)로 정규화
- 휴리스틱 기반으로 잘 알려진 깨짐 패턴(예: `지원서`의 `서`가 `_`로 깨지는 경우) 복원
- 파일 내용은 건드리지 않고, 파일명만 로컬에서 수정

---

## How It Works

1. 브라우저의 `downloads` API를 통해 다운로드 이벤트를 감지합니다.
2. 원래 파일명을 검사하여 다음과 같은 패턴을 탐지합니다.
   - EUC-KR / CP949 모지바케
   - HTML 숫자 엔티티 (`&#숫자;`)
   - 분리된 한글 자모
3. 감지된 패턴에 따라:
   - 바이트 단위로 재구성 후 `TextDecoder("euc-kr")`로 디코딩
   - `&#숫자;` → `String.fromCharCode()` → `normalize("NFC")`
   - 알려진 깨짐 패턴에 휴리스틱 적용
4. 복원된 새 파일명을 브라우저에 제안하여, 사용자가 별도 수정 없이 정상적인 이름으로 저장할 수 있게 합니다.

> 모든 처리는 **사용자 브라우저 로컬에서만** 이루어지며, 어떤 다운로드 정보도 외부 서버로 전송하지 않습니다.

---

## Installation

### From source (developer mode)

1. 이 저장소를 클론하거나 ZIP으로 다운로드 후 압축을 풉니다.
   ```bash
   git clone https://github.com/USERNAME/broken-filename-fixer.git
    ```
2. Chrome 주소창에 `chrome://extensions`를 입력합니다.
3. 우측 상단에서 `Developer mode`를 활성화 합니다.
4. Load unpacked 버튼을 클릭하고, 이 프로젝트 폴더를 선택합니다.
5. 레거시 사이트에서 파일을 다운로드해보며 정상적으로 파일명이 복원되는지 확인합니다.

### From chrome web extension store: TBU

--- 

## Development

- **Platform**: Chrome Extension (Manifest V3)
- **Language**: JavaScript
- **Main APIs**:
  - `chrome.downloads`
  - `TextDecoder` (EUC-KR 등)
  - `String.prototype.normalize('NFC')`

로컬 개발 시에는:

1. `background.js`를 수정합니다.
2. `chrome://extensions`에서 확장 프로그램을 **Reload**한 뒤 동작을 테스트합니다.
3. 다양한 사이트/파일명 패턴을 수집해 테스트 케이스로 추가합니다.

---

## Roadmap

- 윈도우 환경에서 자모 분리 현상 추가 케이스 수집 및 자동 정규화 고도화
- 더 많은 레거시 인코딩(예: Shift-JIS 등) 지원 옵션 추가
- 도메인별/패턴별 규칙을 설정할 수 있는 Options 페이지 제공
- Firefox / Edge 등 다른 브라우저(WebExtension) 지원

---

## License

MIT License