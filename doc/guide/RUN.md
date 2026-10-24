# 옥션허브(AuctionFlow Pro) 실행 방법

> 기준일: 2026-10-24 · Next.js 16.2.4 / React 19 / Node.js 22 에서 확인. Mac 기준.

## 결론 (3줄 요약)

```bash
git clone https://github.com/peter-cho-70/Auctionhub.git && cd Auctionhub
npm install
npm run dev        # → http://localhost:3000 (자동으로 /cases 로 이동)
```

API 키가 없어도 **물건 관리·체크리스트·강의 노트·PDF 업로드**는 동작한다. 키는 주소 검색, 주변 시세, 지도 같은 **외부 API 기능**에만 필요하다.

## 1. 준비물

| 항목 | 내용 |
|---|---|
| Node.js | 20 이상 권장(개발 환경 v22). `node -v` 로 확인 |
| npm | Node 와 함께 설치됨 |
| 브라우저 | Chrome / Safari. 데이터는 **브라우저 localStorage + IndexedDB** 에 저장되므로 같은 주소·같은 브라우저로 열어야 데이터가 보인다 |

## 2. 설치

```bash
npm install
```

`postinstall` 이 `scripts/copy-pdfjs-public.mjs` 를 실행해 PDF.js 워커 파일을 `public/` 에 복사한다. 설치 중 이 단계가 실패하면 PDF 업로드가 동작하지 않는다.

## 3. 환경 변수 (선택)

프로젝트 루트에 `.env.local` 파일을 만들고 필요한 키만 채운다. `.env*` 는 `.gitignore` 대상이라 커밋되지 않는다.

> 확인 필요: README 는 `cp .env.example .env.local` 을 안내하지만 저장소에 `.env.example` 파일이 **없다**. 아래 표를 보고 직접 만들면 된다.

| 변수 | 용도 | 없을 때 |
|---|---|---|
| `JUSO_CONF_KEY` | 도로명주소 검색(표준 주소·법정동·PNU) | 표준 주소 검색 불가 |
| `MOLIT_API_KEY` / `DATA_GO_KR_SERVICE_KEY` | 국토부 실거래가, 토지이용규제 | 주변 시세 수집 불가 |
| `NEXT_PUBLIC_NAVER_MAP_CLIENT_ID` | 화면에 네이버 지도 표시(`ncpKeyId`) | 지도 미표시 |
| `NAVER_MAP_CLIENT_ID` / `NAVER_MAP_CLIENT_SECRET` | 서버 지오코딩(주소→좌표) | 좌표 변환 불가 |
| `GEMINI_API_KEY` | 시세 결과 문장 해석·위험 코멘트 | 해석 코멘트 생략 |
| `NEXT_PUBLIC_SUPABASE_URL` / `NEXT_PUBLIC_SUPABASE_ANON_KEY` | 클라우드 동기화(선택) | **로컬 전용**으로 동작 |

```bash
# .env.local 예시 (값은 본인 키로 교체. 이 파일은 절대 공유·커밋하지 않는다)
JUSO_CONF_KEY=
MOLIT_API_KEY=
NEXT_PUBLIC_NAVER_MAP_CLIENT_ID=
NAVER_MAP_CLIENT_SECRET=
GEMINI_API_KEY=
```

환경 변수를 바꾸면 개발 서버를 **재시작**해야 반영된다. 발급 방법은 [ADDRESS_PUBLIC_API](guide/ADDRESS_PUBLIC_API.md) 참고.

## 4. 실행

### 개발 모드

```bash
npm run dev
```

브라우저에서 <http://localhost:3000> 을 열면 `/cases`(물건 목록)로 이동한다. 개발 모드는 webpack(`next dev --webpack`)으로 실행된다.

### 운영 모드(빌드 후 실행)

```bash
npm run build
npm start          # 기본 포트 3000
```

포트 변경: `npm run dev -- -p 4600` 또는 `npm start -- -p 4600`.

### 주요 화면

| 경로 | 설명 |
|---|---|
| `/cases` | 물건 목록(대시보드) |
| `/cases/[id]` | 물건 상세(원문/PDF·분석·임장·수익률 등 탭) |
| `/study` | 공부하기(강의 요약, 원본 DOCX 다운로드) |

## 5. 데이터 주의 (중요)

- 물건·PDF·분석 데이터는 **서버가 아니라 브라우저**에 저장된다. 다른 포트(`:3000` ↔ `:4600`)나 다른 브라우저는 **별도 저장소**라서 데이터가 비어 보인다.
- 비어 보일 때: **데이터** 탭 → 로컬 스냅샷 복원 / JSON 가져오기 / 백업 물건 복원.
- 초기화·JSON 가져오기(replace) 전에는 먼저 **JSON 내보내기**로 백업한다.
- `public/seed/app-data.json` 은 배포용 샘플이다. 본인 데이터를 대체하는 용도로 쓰지 않는다.

## 6. 문제 해결

| 증상 | 원인 / 조치 |
|---|---|
| `EADDRINUSE: port 3000` | 이미 다른 프로세스가 3000 사용 중. `lsof -iTCP:3000 -sTCP:LISTEN` 으로 확인 후 종료하거나 `-p` 로 다른 포트 사용 |
| PDF 업로드가 동작하지 않음 | `npm install` 을 다시 실행해 `postinstall`(PDF.js 워커 복사)이 성공했는지 확인 |
| 주소 검색·시세가 비어 있음 | `.env.local` 키 누락. 채운 뒤 개발 서버 재시작 |
| 지도가 안 보임 | `NEXT_PUBLIC_NAVER_MAP_CLIENT_ID` 확인, 네이버 콘솔에 `http://localhost:3000` 도메인 등록 |
| 데이터가 사라진 것처럼 보임 | 주소(포트)·브라우저가 달라졌는지 확인 → 5번 항목의 복원 절차 |

## 7. 배포 (참고)

Vercel 등 일반 Next.js 호스팅에 배포한다. 배포본은 브라우저 저장소가 로컬과 분리되므로, 로컬 데이터를 옮길 때는 JSON 내보내기/가져오기를 사용한다.
