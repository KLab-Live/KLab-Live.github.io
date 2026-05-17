# KLab-Live.github.io

## 저장소 역할

- 사용자의 GitHub Pages 사이트 (`https://klab-live.github.io/`)
- iOS / macOS 앱들의 보조 자산 호스팅:
  - 앱 소개 / 랜딩 페이지
  - 개인정보 처리방침
  - 탭별 수식 설명 자료
  - `app-ads.txt` (AdSense 인증)

## 디렉토리 구조

- `index.html` — 앱 목록 랜딩 페이지
- `app-ads.txt` — AdSense 인증 (`pub-4943155389049995`)
- `foc_toolkit/` — FOC Toolkit 수식 자료 + `privacy/`
- `bode_lab/` — Bode Lab 수식 자료 (개인정보 처리방침 없음)
- `filtercraft/` — FilterCraft 앱 소개 + `privacy/`

## 앱별 개인정보 처리방침 호스팅 위치

| 앱 | 이 저장소 내 경로 | App Store가 실제로 참조하는 URL |
|---|---|---|
| FOC Toolkit | `foc_toolkit/privacy/` | `https://klab-live.github.io/foc-toolkit-privacy/` (별도 저장소 `KLab-Live/foc-toolkit-privacy`) |
| FilterCraft | `filtercraft/privacy/` | 이 저장소 그대로 사용 |
| Bode Lab | `bode_lab/privacy/` | 이 저장소 그대로 사용 (App Store 미등록 상태에서 작성됨) |

## ⚠️ FOC Toolkit 개인정보 처리방침 주의사항

- 이 저장소의 `foc_toolkit/privacy/`는 **현재 App Store가 참조하지 않는다.**
- App Store가 파싱하는 실제 URL은 별도 저장소 `KLab-Live/foc-toolkit-privacy`다.
- 따라서 FOC Toolkit 개인정보 처리방침을 수정해야 할 경우:
  - **단기**: `KLab-Live/foc-toolkit-privacy` 저장소도 함께 업데이트해야 한다.
  - **장기**: 추후 App Store 참조 URL을 이 저장소로 이전할 예정이며, 그 시점 이후로는 이 저장소만 수정하면 된다.

## 개인정보 처리방침 템플릿 규칙

모든 앱의 개인정보 처리방침 페이지는 **동일한 템플릿**을 사용한다. 신규 앱 추가 시에도 반드시 이 규칙을 따른다.

- **공통 CSS**: `/assets/privacy/css/style.css` (Jekyll Slate 테마 기반)
- **공통 JS**: `/assets/privacy/js/scale.fix.js` (모바일 뷰포트 스케일 보정)
  - 신규 페이지에서는 **절대경로**로 참조한다:
    ```html
    <link rel="stylesheet" href="/assets/privacy/css/style.css">
    <script src="/assets/privacy/js/scale.fix.js"></script>
    ```
  - 폰트(Noto Sans)는 CSS 내부에서 `../fonts/...` 상대경로로 자동 로드된다.
- **공통 HTML 구조**:
  ```html
  <div class="wrapper">
    <header>
      <h1><a href="…">앱 이름</a></h1>
      <p>한 줄 설명</p>
      <p class="view"><a href="…">앱 홈으로 돌아가기</a></p>
    </header>
    <section>
      <!-- 본문 (h1, h2, h3, p, ul 등) -->
    </section>
  </div>
  ```
## 배포

- main 브랜치에 push하면 GitHub Pages가 자동으로 서빙한다. 별도 빌드 단계 없음.

## 공통 규칙

- 커밋 메시지는 한글로 작성 (전역 규칙 준수).
- `.claude/` 디렉토리는 커밋에서 제외.
