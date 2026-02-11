# PDF Reformatter

CopyKiller 표절검사 결과 PDF를 CopyClean 전문가 리포트 형식으로 재구성하는 웹 앱입니다.

## 기능

- **PDF 업로드 및 분석**: 카피킬러 표절검사 PDF를 업로드하면 AI가 자동으로 메타데이터를 추출합니다.
- **전문가 리포트 생성**: 진단 결과서, 프리미엄 솔루션안, 심층 분석 결과지, 문장별 정밀 대조 결과를 하나의 리포트로 통합합니다.
- **문헌 표절 vs AI 유사도 구분**: 문헌 표절은 **초록색**, AI 유사도는 **주황색**으로 시각적 구분하여 표시합니다.
- **PDF 인쇄/저장**: 브라우저 인쇄 기능으로 전체 리포트를 PDF로 저장할 수 있습니다.

## 리포트 구성

| 페이지 | 내용 |
|--------|------|
| **1페이지** | 진단 결과서 — 표절률 차트, 메타데이터(검사번호, 문서명, 표절률 등), 진단 총평 |
| **2페이지** | 프리미엄 솔루션 — 결제 링크, QR 코드, 면책 문구 |
| **3페이지** | 심층 분석 결과지 — 출처별 유사도 분포 차트, 카테고리 비율, 제출 주의사항 |
| **4페이지~** | 문장별 정밀 대조 결과 — 원문 vs 비교 문헌/AI 대조 카드 (좌측 강조 텍스트, 우측 출처 정보) |

## 기술 스택

- **Vue 3** — 프론트엔드
- **Tailwind CSS** — 스타일링
- **PDF.js** — PDF 파싱
- **Chart.js** — 차트 시각화
- **Google Gemini 2.0 Flash** — 이미지 기반 메타데이터·문장 대조 추출 (Vision AI)

---

## 1~3페이지 기술 상세

페이지 1~3 구축에 사용된 기술들을 세부적으로 정리합니다.

### 페이지 1: 진단 결과서 (Chart)

| 기술 | 용도 |
|------|------|
| **Chart.js** | 막대 차트 — "제출자 평균(15%)" vs "현재 문서" 표절률 비교 |
| **Chart.js Custom Plugin** | `afterDatasetsDraw` 훅으로 막대 위에 퍼센트 라벨 직접 렌더링. 표절률 30% 초과 시 "+30% 초과" 주황색 칩(roundRect) 오버레이 |
| **Canvas API** | `ctx.roundRect`, `ctx.fillText`, `ctx.font` 등으로 커스텀 차트 라벨 그리기 |
| **Vue 3 Composition API** | `ref`, `computed`, `watch` — 반응형 표절률·판정 텍스트 바인딩 |
| **Tailwind Grid** | `grid-cols-2`, `grid-cols-4` — 메타정보 바, 대시보드·진단 총평 2단 레이아웃 |
| **Gemini Vision API** | 1~2페이지 이미지를 Base64로 전달해 표절률, 검사번호, 문서명, 문장 수 등 메타데이터 JSON 추출 |
| **텍스트 정규식 fallback** | AI 추출 실패 시 `extractFromText()`로 "표절률 N%", "검사번호", "전체 문장" 등 패턴 매칭 |
| **Base64 로고** | 인라인 `data:image/png;base64` 로고 임베딩 — 외부 리소스 없이 출력 |

### 페이지 2: 프리미엄 솔루션 (Dark Mode)

| 기술 | 용도 |
|------|------|
| **qrcode (QRCode.toDataURL)** | 결제 링크를 QR 코드 이미지로 변환. `paymentLink` 변경 시 `watch`로 자동 재생성 |
| **Tailwind Dark Palette** | `bg-slate-900`, `bg-slate-800`, `border-slate-700` — 다크 테마 카드·배경 |
| **CSS 변수/커스텀 클래스** | `.p2-container`, `.p2-card`, `.p2-highlight` — 블루 액센트, 카드 라운드, 그라디언트 |
| **Vue ref** | `paymentLink`, `servicePrice`, `qrCodeUrl` — 결제 링크·가격 동적 바인딩 |
| **폰트** | Google Fonts `Inter`, `Noto Sans KR` — 한글·영문 혼용 가독성 |

### 페이지 3: 심층 분석 결과지

| 기술 | 용도 |
|------|------|
| **Chart.js (대체)** | 막대 차트 대신 `computed`로 계산한 **유사성 분석 분포 바** — `flex` + `width: N%`로 동적 프로그레스 바 구현 (동일/의심/인용/AI유사 비율) |
| **distributionBar computed** | `totalSentences` 기준으로 `identical`, `suspicious`, `citation`, `aiSimilar` 퍼센트 계산 |
| **categoryRatios** | 학술논문/공공기관/웹 콘텐츠 비율 막대 — 각 카테고리별 `width: ratio%` 바 |
| **Gemini generateDiagnosisReport** | 전체 PDF 텍스트 + 표절률을 입력으로 `diagnosisSummary`(진단 총평 3개), `submissionNotes`(제출 주의사항 3개) 생성 |
| **replaceBrand** | AI 출력 중 "카피킬러" → "카피클린", "CopyKiller" → "CopyClean" 치환 |
| **p1-table** | `border-collapse`, `th/td` — Report ID, 검사 일련번호, 문서 원본명, 판정 결과 등 메타데이터 테이블 |
| **동적 verdict 색상** | `numericRate` 구간별 `verdict-danger`(빨강)/`verdict-safe`(초록) — 15%·30% 기준으로 판정 배지 색 변경 |

### 공통 인프라 (1~3페이지 공유)

| 기술 | 용도 |
|------|------|
| **A4 고정 레이아웃** | `.pdf-page` 794×1123px — PDF 출력 시 페이지 크기 고정 |
| **@media print** | `page-break-after: always`, `print-color-adjust: exact` — 배경색/그라디언트 인쇄 보존, nav 숨김, 페이지 나누기 |
| **PDF.js** | `getDocument`, `getPage`, `getTextContent`, `render` — PDF 로딩, 텍스트 추출, 캔버스 렌더(이미지화) |
| **PDF.js Worker** | CDN `pdf.worker.min.js` — 메인 스레드 블로킹 방지 |
| **Vue nextTick** | 차트·QR DOM 마운트 후 `renderChart()` 실행 |
| **jsPDF / html2canvas** | 의존성 포함되어 있으나 현재는 `window.print()` 기반 출력 사용 |

## 시작하기

### 1. 실행 방법

단일 HTML 파일로 구성되어 있어 별도 빌드 없이 브라우저에서 바로 실행할 수 있습니다.

```bash
# 로컬 서버로 실행 (권장 — CORS 이슈 방지)
npx serve .
# 또는
python -m http.server 8000
```

`PDF_Reformatter.html`을 직접 열어도 동작하지만, 일부 환경에서는 파일 API 제한으로 인해 업로드가 제대로 되지 않을 수 있습니다. 로컬 서버 사용을 권장합니다.

### 2. API 키 설정

Google Gemini API 키가 필요합니다. [Google AI Studio](https://aistudio.google.com/apikey)에서 발급받을 수 있습니다.

`PDF_Reformatter.html` 내부의 `apiKey` 변수 값을 본인의 API 키로 교체하세요.

```javascript
const apiKey = "YOUR_GEMINI_API_KEY";
```

### 3. 사용 흐름

1. **PDF 업로드** — 상단의 "PDF 업로드 및 분석" 버튼 클릭 후 카피킬러 표절검사 PDF 선택
2. **분석 대기** — AI가 1~2페이지 이미지를 분석해 메타데이터를 추출하고, 4페이지 이후 문장별 대조를 추출합니다 (약 10~20초 이상 소요)
3. **결과 확인** — 스크롤하여 생성된 리포트 확인
4. **PDF 저장** — "인쇄" 버튼 클릭 후 "대상: PDF로 저장" 선택

### 4. 제한 사항

- Gemini API 호출 수에 따라 비용이 발생할 수 있습니다.
- 입력 PDF는 카피킬러(또는 유사 표절검사 서비스) 결과 형식에 가까울수록 정확도가 높습니다.

## 프로젝트 구조

```
pdf_reformatter/
├── PDF_Reformatter.html   # 단일 페이지 앱 (HTML + CSS + JS)
├── README.md
└── docs/
    └── daily/             # 작업일지
        └── 2026-02-11.md
```

## 작업일지

상세 변경 이력은 `docs/daily/` 폴더의 날짜별 마크다운 파일을 참고하세요.

## 라이선스

프로젝트 소유자에 따릅니다.
