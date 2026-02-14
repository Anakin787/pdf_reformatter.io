# PDF Reformatter

CopyKiller 표절검사 결과 PDF를 CopyClean 전문가 리포트 형식으로 재구성하는 웹 앱입니다.

## 기능

- **PDF 업로드 및 분석**: 카피킬러 표절검사 PDF를 업로드하면 Gemini Vision AI가 자동으로 메타데이터를 추출합니다.
- **전문가 리포트 생성**: 진단 결과서, 프리미엄 솔루션안, 문장별 정밀 대조 결과를 하나의 리포트로 통합합니다.
- **문헌 표절 vs AI 유사도 구분**: 문헌 표절은 **초록색(teal)**, AI 유사도는 **주황색(orange)**으로 시각적 구분하여 표시합니다.
- **PDF 인쇄/저장**: 브라우저 인쇄 기능으로 전체 리포트를 PDF로 저장할 수 있습니다.

## 리포트 구성

| 페이지 | 내용 |
|--------|------|
| **1페이지** | 진단 결과서 — 표절률 차트, 메타정보(리포트ID, 발급일자, 문서명, 진단모델), 종합 판정, 정밀 진단 결과 요약, 유사성 분포, 카테고리 비율, 진단 총평·제출 주의사항 |
| **2페이지** | 프리미엄 솔루션 — 유사도 개선 제안, 결제 링크, QR 코드, 면책 문구 |
| **3페이지~** | 문장별 정밀 대조 결과 — 원문 vs 비교 문헌/AI 대조 카드 (좌측 강조 텍스트, 우측 출처 정보) |

## 기술 스택

- **Vue 3** — 프론트엔드 (CDN, 빌드 불필요)
- **Tailwind CSS** — 스타일링 (CDN)
- **PDF.js** — PDF 파싱 및 이미지 렌더링
- **Chart.js** — 표절률 비교 막대 차트
- **QRCode.js** — 결제 링크 QR 코드 생성
- **Google Gemini 2.0 Flash** — 이미지 기반 메타데이터·문장 대조 추출 (Vision AI)

---

## 시작하기

### 1. 클론 및 실행

```bash
git clone https://github.com/your-repo/pdf_reformatter.git
cd pdf_reformatter
```

단일 HTML 파일로 구성되어 있어 별도 빌드나 의존성 설치가 필요 없습니다.
로컬 서버를 띄워서 실행하세요 (CORS 이슈 방지를 위해 권장).

```bash
# 방법 1: npx serve (Node.js 설치 필요)
npx serve .

# 방법 2: Python 내장 서버
python -m http.server 8000

# 방법 3: VS Code Live Server 확장 사용
# PDF_Reformatter.html 우클릭 → "Open with Live Server"
```

브라우저에서 `http://localhost:8000/PDF_Reformatter.html` (또는 해당 포트)로 접속합니다.

> **참고:** `PDF_Reformatter.html`을 더블클릭으로 직접 열어도 동작하지만, 일부 브라우저에서 파일 프로토콜(`file://`) 제한으로 PDF 업로드가 안 될 수 있습니다.

### 2. API 키 설정

Google Gemini API 키가 필요합니다. [Google AI Studio](https://aistudio.google.com/apikey)에서 무료로 발급받을 수 있습니다.

`PDF_Reformatter.html` 내부의 `apiKey` 변수 값을 본인의 API 키로 교체하세요.

```javascript
const apiKey = "YOUR_GEMINI_API_KEY";
```

### 3. 사용 흐름

1. **PDF 업로드** — "분석 시작하기" 버튼 클릭 후 카피킬러 표절검사 결과 PDF 선택
2. **분석 대기** — AI가 PDF 페이지를 이미지로 변환하여 메타데이터 추출 및 문장별 대조 분석 수행 (약 10~30초 소요)
3. **결과 확인** — 스크롤하여 생성된 리포트 확인
4. **PDF 저장** — "PDF 다운로드" 버튼 클릭 → 인쇄 대화상자에서 "PDF로 저장" 선택

### 4. 사용자 설정 변경

`PDF_Reformatter.html` 파일에서 아래 항목을 직접 수정할 수 있습니다.

| 항목 | 줄 번호 | 변수명 | 기본값 |
|------|---------|--------|--------|
| Gemini API 키 | **515줄** | `apiKey` | `"YOUR_GEMINI_API_KEY"` |
| 결제 링크 | **518줄** | `paymentLink` | `"https://copyclean.ai/checkout/..."` |
| 서비스 가격 | **519줄** | `servicePrice` | `295000` (원) |

```javascript
// 515줄 — API 키
const apiKey = "YOUR_GEMINI_API_KEY";

// 518줄 — 결제 링크 (QR 코드에도 자동 반영)
const paymentLink = ref("https://copyclean.ai/checkout/CP-20260201-8829");

// 519줄 — 서비스 가격 (₩ 단위, 콤마 자동 포맷)
const servicePrice = ref(295000);
```

### 5. 제한 사항

- Gemini API 무료 등급은 분당 요청 수 제한이 있습니다. 대량 분석 시 유료 플랜이 필요할 수 있습니다.
- 입력 PDF는 카피킬러(또는 유사 표절검사 서비스) 결과 형식에 가까울수록 정확도가 높습니다.
- PDF 저장 시 Cursor 내장 브라우저 대신 Chrome/Edge 등 일반 브라우저를 사용하세요 (내장 브라우저에서 0KB 파일 생성 이슈).

---

## 1~2페이지 기술 상세

### 페이지 1: 진단 결과서

| 기술 | 용도 |
|------|------|
| **Chart.js** | 막대 차트 — "제출자 평균(15%)" vs "현재 문서" 표절률 비교 |
| **Chart.js Custom Plugin** | `afterDatasetsDraw` 훅으로 막대 위에 퍼센트 라벨 직접 렌더링. 기준 초과 시 "+N% 초과" 주황색 칩(roundRect) 오버레이 |
| **Canvas API** | `ctx.roundRect`, `ctx.fillText` 등으로 커스텀 차트 라벨 그리기 |
| **Vue 3 Composition API** | `ref`, `computed`, `watch` — 반응형 표절률·판정 텍스트 바인딩 |
| **Tailwind Grid** | `grid-cols-2`, `grid-cols-4`, `grid-cols-5` — 메타정보 바, 대시보드, 진단 요약 레이아웃 |
| **Gemini Vision API** | PDF 페이지 이미지를 Base64로 전달해 표절률, 문서명, 문장 수 등 메타데이터 JSON 추출 |
| **텍스트 정규식 fallback** | AI 추출 실패 시 `extractFromText()`로 "표절률 N%", "전체 문장" 등 패턴 매칭 |

### 페이지 2: 프리미엄 솔루션 (Dark Mode)

| 기술 | 용도 |
|------|------|
| **QRCode.toDataURL** | 결제 링크를 QR 코드 이미지로 변환. `paymentLink` 변경 시 `watch`로 자동 재생성 |
| **Tailwind Dark Palette** | `bg-slate-900`, `bg-slate-800` — 다크 테마 카드·배경 |
| **Vue ref** | `paymentLink`, `servicePrice`, `qrCodeUrl` — 결제 링크·가격 동적 바인딩 |

### 공통 인프라

| 기술 | 용도 |
|------|------|
| **A4 고정 레이아웃** | `.pdf-page` 794×1123px — PDF 출력 시 페이지 크기 고정 |
| **@media print** | `page-break-after: always`, `print-color-adjust: exact` — 배경색 인쇄 보존, nav 숨김, 페이지 나누기 |
| **PDF.js + Worker** | PDF 로딩, 텍스트 추출, 캔버스 렌더링 (메인 스레드 블로킹 방지) |
| **동적 페이지 분배** | 고정 카드 높이(200px) 기반으로 문장별 대조 카드를 여러 페이지에 자동 배분 |

---

## 프로젝트 구조

```
pdf_reformatter/
├── PDF_Reformatter.html   # 단일 페이지 앱 (HTML + CSS + JS)
├── origin.png             # CopyClean 로고 (투명 배경)
├── README.md
└── docs/
    └── daily/             # 작업일지
        ├── 2026-02-11.md
        ├── 2026-02-12.md
        ├── 2026-02-13.md
        └── 2026-02-14.md
```

## 작업일지

상세 변경 이력은 `docs/daily/` 폴더의 날짜별 마크다운 파일을 참고하세요.

## 라이선스

프로젝트 소유자에 따릅니다.
