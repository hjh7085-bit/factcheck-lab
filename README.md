# 마운자로·영양제 중복·궁합 10초 체크기

팩트체크 건강연구소의 첫 번째 웹툴. 먹는 약·영양제를 체크하면 **겹치는 성분(🔴)**, **흡수 경쟁으로 시간을 나눠야 하는 조합(🟡)**, **같이 먹으면 좋은 조합(🟢)** 과 **하루 복용 시간표**를 즉시 만들어 줍니다.

- 성분 29종 · 조합 규칙 22개 · 의존성 0개(바닐라 JS) · GitHub Pages 정적 배포
- 문헌·허가사항 근거를 각 카드에 표기 (`근거 · ...`)
- 모든 결과에 "참고용 · 처방약 병용은 의사·약사 상담" 고지

## 구조

```
factcheck-lab/
├── index.html          # 마크업 · SEO 메타 · JSON-LD(WebApplication, FAQPage)
├── styles.css          # 디자인 시스템(라이트/다크), 반응형
├── app.js              # 성분 데이터 + 규칙 엔진 + 렌더링
├── assets/
│   ├── favicon.svg
│   └── og-image.png    # 1200×630 공유 이미지 (/workspace/make_og_image.py로 생성)
├── .nojekyll
└── robots.txt
```

## 로컬 실행

```bash
cd factcheck-lab
python3 -m http.server 8231
# http://localhost:8231
```

## 기능

| 기능 | 설명 |
| --- | --- |
| 체크리스트 | 4개 카테고리(비만·혈당 / 간·위·장 / 비타민·미네랄 / 혈행·항산화·기타) 29종 |
| 빠른 예시 | 마운자로 다이어트 세트·직장인 기본 루틴·간·위장 케어 세트·미네랄 풀세트 |
| 분석 엔진 | 🔴 8 · 🟡 5 · 🟢 9 규칙, 약 이름이 바뀌면 문구도 동적 생성 |
| 맞춤 시간표 | 아침 공복 / 아침·점심·저녁 식후 / 취침 전 5개 시간대 + 주 1회 주사 슬롯 |
| 간격 자동 분리 | 철분·칼슘·아연·마그네슘·식이섬유는 충돌 시 자동으로 다른 시간대에 배치 |
| 공유 | 선택 항목이 `#i=mounjaro,zinc` 형태로 URL에 남아 링크 공유·복원 가능 |
| 결과 복사 | 결과 요약 + 시간표를 텍스트로 복사 |
| 테마 | 시스템 설정 따름 + 라이트/다크 수동 전환(localStorage) |
| SEO | 메타·OG·JSON-LD(FAQ 5문항)·시맨틱 마크업 |

## 콘텐츠 수정 방법

### 성분 추가/수정 — `app.js`의 `ITEMS`

```js
{
  id: "zinc",                 // 규칙에서 참조하는 키
  cat: "vitamin",             // metabolic | gut | vitamin | etc
  name: "아연",               // 화면 표기
  sub: "15~30mg",             // 카드 부제
  prefer: ["morning", "dinner"], // 시간표 우선순위(앞이 1순위)
  why: "공복에 먹으면 메스꺼울 수 있어 식후로 두세요.",
}
```

시간대 키: `weekly` · `morning-empty` · `morning` · `lunch` · `dinner` · `bedtime`

### 규칙 추가 — `app.js`의 `RULES`

```js
{
  id: "rule-id",
  level: "red",            // red | amber | green
  badge: "주의",           // 카드 좌측 배지 문구
  title: "제목",
  when: (s) => s.has("zinc") && !s.has("copper"),   // s = Set<item id>
  body: (s) => "<p class='rule__body'>설명</p>",
  source: "IOM DRI · 아연 UL 35mg/일",
  link: "zinc",            // COLUMNS의 키 → '관련 칼럼 보기' 링크
}
```

### 간격(2시간) 규칙 — `SPACING`

배열에 `["fiber", "iron"]`처럼 추가하면 시간표에서 두 항목이 같은/이웃 시간대에 배치되지 않습니다.

## 팩트체크 원칙

- 브랜드 컨셉이 "팩트 신뢰"이므로, 규칙을 추가할 때는 **허가사항·공개 라벨·1차 문헌**을 확인하고 `source`에 근거 수준을 정확히 적습니다(근거가 약하면 "근거 제한적"이라고 표기).
- 칼럼 링크(`COLUMNS`)는 실제 발행된 글 ID만 넣습니다. 2026-11-15 확인 결과, 초기 요청서에 있던 `224475021341`(오메가3)·`224475185348`(소화효소)는 존재하지 않는 ID여서 실제 글인 `224474981968`·`224474930562`로 교체했습니다.

## 배포 (GitHub Pages)

```bash
cd factcheck-lab
git init -b main
git add -A
git commit -m "feat: 마운자로·영양제 중복·궁합 10초 체크기"
git remote add origin https://github.com/hjh7085-bit/factcheck-lab.git
git push -u origin main
```

저장소 **Settings → Pages → Source: Deploy from a branch → main / (root)** 로 지정하면 `https://hjh7085-bit.github.io/factcheck-lab/` 에서 열립니다.

## 다음 단계 아이디어

- 애드센스 승인 후 결과 하단에 광고 슬롯(`<!-- ADSENSE -->` 자리) 삽입
- 결과에 나온 성분별 제휴 제품 CTA(쿠팡 파트너스 · 아이허브 `?rcode=PPU0869`) 연결
- 체크 결과를 이미지 카드로 저장해 인스타·스레드 공유
