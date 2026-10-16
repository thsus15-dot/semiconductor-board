# 휴게실 반도체 현황판

75인치 전자칠판용 채용·뉴스 대시보드.
**Claude가 매일 아침 직접 검색해서 `data.json`을 갱신·커밋합니다. API 키 없음.**

```
index.html   화면 (데이터는 data.json 에서만 읽음)
data.json    유일한 데이터 소스 — Claude 루틴이 매일 아침 갱신
README.md    이 문서
```

수집 스크립트도, GitHub Actions 워크플로도, API 키도 없습니다.

## 1. 로컬에서 보기

`index.html` 더블클릭은 안 됩니다 (file:// 에서는 data.json fetch 가 차단됨). 폴더에서:

```
python -m http.server 8000
```
→ http://localhost:8000

## 2. 올리기 (사용자가 해야 하는 단계)

GitHub 에 빈 저장소를 만들고:

```
cd C:\Users\thsus\semiconductor-board
git init && git add . && git commit -m "init" && git branch -M main
git remote add origin https://github.com/<아이디>/<저장소>.git
git push -u origin main
```

그다음 [vercel.com](https://vercel.com/new) 에서 Import → 프레임워크 **Other** → Deploy.
(정적 파일이라 GitHub Pages 를 켜도 동일하게 동작합니다.)

## 3. 매일 자동 갱신 (Claude 루틴)

저장소가 생긴 뒤 Claude Code 에서 `/schedule` 로 루틴을 만듭니다.
루틴은 Anthropic 클라우드에서 돌며 **이 저장소를 clone → 검색 → data.json 수정 → commit & push** 합니다.
push 가 되면 Vercel 이 자동 재배포하고, 화면은 10분 안에 새 데이터를 집어옵니다.

- 스케줄: 매일 KST 07:00 (= UTC 22:00, cron `0 22 * * *`)
- 루틴 관리/삭제: https://claude.ai/code/routines

루틴이 하는 일(프롬프트 요지):

1. 최근 48시간 한국 반도체 뉴스 3건을 검색 — 주제가 겹치지 않게, 실제 기사 URL만
2. 잡코리아 검색 페이지에서 반도체 엔지니어 공고 8건을 수집 — 직무 5종 균형, 신입 가능 우선
3. 반도체와 무관한 공고(인테리어·화장품·단순생산 등)는 제외
4. 확인 못 한 값은 지어내지 말고 `마감일 확인` / `공고 참조` 로 둘 것
5. `data.json` 을 쓰고 커밋·푸시

## 4. 전자칠판 띄우기

Chrome 바로가기를 만들어 대상 뒤에 플래그를 붙입니다:

```
chrome.exe --kiosk --noerrdialogs --disable-session-crashed-bubble https://내주소.vercel.app
```

- Wake Lock 으로 화면 꺼짐을 막지만, **Windows 전원 옵션의 "디스플레이 끄기"는 안 함으로** 따로 설정해야 합니다.
- 6시간마다 자동 새로고침, 10분마다 data.json 재조회.
- 마감일이 지난 공고는 화면에서 자동으로 사라집니다.
- 데이터가 30시간 넘게 갱신되지 않으면 제목 옆에 빨간 **"데이터 갱신 지연"** 배지가 뜹니다 — 루틴이 멈췄다는 신호입니다.

## 데이터 형식 (손으로 고칠 때)

```json
{
  "company": "회사명",
  "title": "공고 제목",
  "category": "공정/소자 | 첨단패키징 | 회로설계 | 장비/설비 | 품질/테스트",
  "location": "근무지",
  "experience": "신입 지원 가능",
  "deadline": "2026-10-25",   // 빈 문자열이면 화면에 '마감일 확인'
  "url": "공고 원문 URL"
}
```

## 알아둘 것

- **검색으로 공고의 마감일·근무지까지 정확히 긁어오긴 어렵습니다.** 공식 채용 사이트(SK커리어스 등)는
  자바스크립트로만 그려져 읽히지 않고, 잡코리아 목록에는 마감일이 없습니다. 그래서 모르는 값은
  비워두고 화면에 `마감일 확인` / `공고 참조` 로 띄웁니다 — QR 로 원문을 보게 하는 쪽이 안전합니다.
- 공고 정확도가 중요하면 담당자가 `data.json` 의 `jobs` 배열을 직접 손보면 됩니다. 루틴에게
  "jobs 는 건드리지 말고 news 만 갱신하라"고 프롬프트를 바꿔도 됩니다.
- 화면은 Tailwind CDN, Pretendard 폰트, QR 생성 API(api.qrserver.com)를 외부에서 받습니다.
  사내망이 이 세 도메인을 막으면 레이아웃이 깨집니다.
- 지수·가격 같은 숫자는 일부러 뺐습니다. 자동으로 못 받아오는 값을 고정 숫자로 띄우면
  틀린 정보가 몇 주씩 벽에 걸려 있게 됩니다.
