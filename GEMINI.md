# GEMINI.md — 반도체 현황판 인수인계

Gemini CLI 가 이 폴더에서 시작하면 자동으로 읽는 문서. 데이터 형식·화면 사용법은 README.md 참고.

## 구조
- `index.html` 화면 전부 (Tailwind CDN, 빌드 없음). `data.json` 만 읽는다.
- `data.json` 유일한 데이터. **매일 아침 자동 루틴이 덮어쓰고 커밋·푸시한다 — 손으로 고치면 다음 실행 때 사라질 수 있다.**
- `companies.txt`(면접후기 기업), `top100.txt`(연봉 TOP100) — 공고 검색 순환용 목록.
- 배포: GitHub `main` 푸시 → Vercel 자동 재배포 (semiconductor-board.vercel.app).

## 자동 수집 (이 저장소 밖)
- `C:\Users\thsus\board-cron\` : `run-jobs.ps1`(실행기), `news-prompt.md`, `jobs-prompt.md`, `settings.json`(접속 허용 도메인).
- Windows 작업 `SemiconductorBoard-Jobs` 월~금 08:27, 숨김 창, 배터리에서도 실행. Claude Code 헤드리스(`claude -p`)로 돈다.
- 실패하면 09:05 Claude 클라우드 루틴이 뉴스만 대신 갱신, 09:35 감시 루틴이 공고 미갱신 알림.
- 결과: `board-cron\last-status.txt` 에 두 줄(news/jobs exit=0)이면 성공. 로그는 `board-cron\log\` (UTF-16).
- 수집 기준(건수·출처·필터)을 바꾸려면 위 프롬프트 md 를 고친다. index.html 이 아니다.

## 화면 규칙 (index.html)
- 큰 화면(`lg:` 1024px 이상)=75인치 전자칠판 키오스크: 고정 높이, 자동 롤링, QR.
- 1024px 미만=폰: 뉴스 위·채용 아래로 반씩, 각자 스크롤, 자동롤링·QR 없음(`isMobile`).
  폰용 변경은 접두사 없는 클래스, 키오스크는 `lg:` 로 — 한쪽 고칠 때 다른 쪽 깨지지 않게.
- D-day 는 화면이 `deadline` 으로 매번 계산. 지난 공고는 자동으로 숨는다.
- `added`(KST 날짜): NEW 배지 3일, 첫날만 맨 앞 정렬 (`liveJobs`).
- 주간 뉴스 표시 범위 7일 (`newsAge(n.time) <= 7`).

## 작업 방식
- 확인: `python -m http.server 8000` → localhost:8000 (file:// 로는 data.json 못 읽음). 폰 폭(390px)과 큰 화면 둘 다 볼 것.
- 커밋 전 `git pull` — 루틴이 매일 data.json 을 푸시하므로 충돌 주의.
