---
name: "appsshot-code-to-figma-draw"
description: "코드에 구현된 앱 화면을 origin/dev 기준으로 읽어 Figma에 한국어·영어로 그린다. 360×780 기준으로 그린 뒤 폰(966×2093)·패드(1720×2406)로 파생하고, 색·텍스트·컴포넌트를 디자인 시스템에 연결한 채로 만든다. Use when 앱스크린샷 제작, 화면을 피그마에 그릴 때, 코드 기준 디자인 최신화, code to figma."
---

# 코드 → Figma: 화면 그리기 절차

> 원본은 `routinery-v2` 레포의 `.claude/rules/code-to-figma-draw.md`. 여기 사본은 대시보드 공개용이라 Figma 파일키 등 내부 식별자를 플레이스홀더로 바꿨다.

**"이 화면들 그려줘" 요청을 받았을 때 처음부터 끝까지 이대로 한다.**
어떤 Figma 파일에서 시작하든 같은 입력이면 같은 결과가 나오도록 설계돼 있다.

배경·실패 사례는 `routinery-v2 · .claude/rules/code-to-figma-sync.md` 에 있다. 여기는 절차만 둔다.

---

## 입력 — 이 4가지가 있어야 시작한다

| 입력 | 예 | 없으면 |
|---|---|---|
| **① 그릴 화면** | "발견탭 인기 루틴·인기 할 일", "루틴 설정 페이지" | 물어본다 |
| **② 대상 Figma 파일 URL** | `https://figma.com/design/<fileKey>/...?node-id=<page>` | 물어본다 |
| **③ 언어** | 한국어 / 영어 | 기본 **둘 다** |
| **④ 기기** | 폰 / 패드 | 기본 **둘 다** |

②는 **페이지 노드까지** 찍힌 링크를 받는다(`node-id`). 파일만 주면 어디에 그릴지 모른다.

> **이 문서는 한국어·영어까지만 그린다.** 나머지 언어는 한·영 섹션을 원본으로 clone 해서
> 치환하는 별도 절차다 → [앱스샷 다국어 대응 스킬](../appsshot-i18n/SKILL.md).
> 한국어 = 의미 원본, 영어 = 문장 골격 원본이므로 **두 섹션은 사람이 직접 관리**하고 자동화하지 않는다.

---

## 0. 소스 확정 — 매번, 예외 없이

그리기 **직전에** 한다. 하루 전 값도 못 믿는다.

```bash
git remote -v                                       # github.com/Routinery/routinery-v2 인지 확인
git fetch origin dev
git log -1 --format='%h %ad %s' --date=short origin/dev
git rev-list --left-right --count origin/dev...HEAD # 왼쪽 = 내가 뒤처진 커밋 수
```

- **기준 브랜치는 `origin/dev`.** 작업 트리는 시안 코드가 섞여 있어 진실이 아니다.
- **체크아웃하지 않는다.** 전부 `git show origin/dev:<path>` 로 읽는다.
- 이때 나온 **커밋 해시를 기록**한다. 산출물 보고와 Figma 섹션 설명에 같이 적는다.

```
기준: Routinery/routinery-v2 · origin/dev · fc2384ddf (2026-09-17)
```

> 같은 결과를 재현하려면 이 해시가 같아야 한다. 이게 재현성의 1번 조건이다.

---

## 1. 화면 → 파일 목록

### 진입점 찾기

| 화면 종류 | 진입점 |
|---|---|
| 탭 화면 | `src/navigations/TabNavigator.tsx` |
| 푸시/모달 화면 | `src/navigations/index.tsx` |
| 위젯 (iOS) | `ios/routinery-widget/Widgets/<name>/` |

### 의존 트리 펼치기

화면 파일 하나만 읽으면 하위 컴포넌트가 구버전인 걸 못 본다.

```bash
python3 scripts/design-sync/deps.py src/screens/RecommendScreenV2/index.tsx
python3 scripts/design-sync/deps.py <entry> --depth 3
```

출력된 목록이 **읽어야 할 파일 전부**다. `~/`·`@components/` 별칭과 `./components/X`
상대경로를 둘 다 따라간다.

### 실제로 렌더되는 것만 고른다

같은 이름의 죽은 코드가 남아 있다. 화면 파일의 JSX에서 **실제로 쓰이는 것**만 그린다.

| 화면 | 쓰는 것 | 죽은 코드 |
|---|---|---|
| 발견탭 인기 루틴 | `<PopularRoutineScreen innerViewMode />` | `RecommendScreenV2/components/PopularRoutine.tsx` |
| 재생 버튼 | `TimerLive`(삼각형) + navy05 원판 | DS `ic_play`(원+삼각형) |
| 할 일 체크 | `RDSCheckBox isCircle` | DS 네모 `ic_check` |

---

## 2. 값 추출 — 화면당 5가지

| 대상 | 어디서 | 규칙 |
|---|---|---|
| **레이아웃** | `createStyles()` 의 `ihp()`/`iwp()` | `ihp`=세로/780, `iwp`=가로/360 · 기준 **360×780** |
| **문자열** | `translate('key')` → 번역 파일 | 아래 |
| **상태** | `{cond && <X/>}` 조건 | 어떤 상태를 그릴지 먼저 정한다 |
| **데이터** | 로컬 json / 서버 | 서버면 아래 |
| **컴포넌트** | 실제 렌더되는 파일 | 위 표 |

### 문자열 — 번역하지 않고 인용한다

스크린샷은 앱을 찍은 것이다. 새로 번역하면 스토어 화면과 앱이 다른 말을 한다.

```bash
git show origin/dev:src/assets/translations/ko/ko.json        > /tmp/ko.json
git show origin/dev:src/assets/translations/en/en.json        > /tmp/en.json
git show origin/dev:src/assets/translations/en/en_habits.json > /tmp/en_habits.json
```

우선순위: `<lang>.json` → `<lang>_habits.json`(ko 와 **인덱스 동일**) →
`recommend_todolist.json`·`recommend_sample.json`·`taglist.json`(12개 언어 인라인).

**앱 번역값이 어색해도 그대로 넣는다.** 고칠 거면 스크린샷이 아니라 코드 티켓으로 올린다.

### 코드에 없는 문자열 = 서버 데이터

| 종류 | 출처 |
|---|---|
| 인기 루틴 이름·설명 | 콘텐츠 DB(서버) |
| 발견탭 카테고리 탭 라벨 | 서버 `tagName` |
| 루틴 썸네일 | `{CLOUD_FRONT_URL}/images/{routineThumb}.png` |

→ **실기기 스크린샷이 있으면 그 문구를 그대로 쓴다.** 없으면 직접 쓰고 **지어낸 목록을 보고한다.**

---

## 3. 대상 파일 준비 — 새 파일에서도 그대로

### DS 라이브러리는 키로 들어온다 — 사전 작업 없음

**검증됨(2026-09-17): 별도 라이브러리 활성화 없이 다른 파일에서 그대로 import 된다.**

```js
await figma.variables.importVariableByKeyAsync(VAR_KEY);   // → Navy/navy05
await figma.importStyleByKeyAsync(TEXT_STYLE_KEY);         // → Body / *B14
await figma.importComponentSetByKeyAsync(COMPONENT_KEY);   // → Navigation/Bottom Navigation
```

키 전체 표 →
`routinery-v2 · .claude/skills/code-to-figma-design-system/references/project-routinery.md`

### 구조 — 이 이름 규칙을 지키면 파일이 달라도 결과가 같다

```
SECTION  <국가코드>_<앱스샷 사이즈>_<프로젝트명>        예: KR_5.5~6.5_뉴티너리 · US_iPad_뉴티너리
  FRAME  <번호> <화면명>                                예: 03 발견탭 (Discover)
  FRAME  <번호> <화면명> — iPad
```

- 프레임 번호는 **정렬 키**다. `01`·`02`·`03`·`03b`·`04`… 로 붙인다.
- 섹션 간격 **3000 이상.** 좁으면 옆 섹션 자식으로 빨려 들어간다.
- 프레임 가로 간격: 폰 240 / 패드 300.

### 작업용 기준 섹션

`TMP_360_base` 섹션을 만들어 **360×780 원본**을 그리고, 거기서 폰·패드를 파생한다.
파생이 끝나도 지우지 말 것 — 다음 최신화 때 여기서 다시 파생한다.

---

## 4. 그리기 — 360×780 기준으로만

966에서 직접 그리면 DS 텍스트 스타일(14px 등)과 크기가 어긋난다. **반드시 360에서 그린다.**

### 4-1. RDS 매핑 — 노드를 만드는 그 줄에서 바로 연결한다

| 코드 | Figma | API |
|---|---|---|
| `colors.navy05` | 색 **변수** | `importVariableByKeyAsync` → `setBoundVariableForPaint` |
| `Typo size={n} bold` | **텍스트 스타일** | `importStyleByKeyAsync` → `setTextStyleIdAsync` |
| `<RDSIcon icon="X"/>` | DS 아이콘 · 없으면 레포 SVG | `src/components/Icon/icons/assets/<X>.svg` |
| `images.ic[...]` | 실제 에셋 | CloudFront webp → **PNG 변환** 후 업로드 |
| 하단 탭·버튼·필드 | DS 컴포넌트 인스턴스 | `importComponentSetByKeyAsync` |

```js
// 색: 리터럴 hex 를 베이스로 깔고 변수를 묶는다 (검정 렌더 방지)
n.fills = [figma.variables.setBoundVariableForPaint(
  {type:'SOLID', color: rgb('#3B3D4A')}, 'color', await V('navy05'))];

// 텍스트: 폰트 로드 → characters → 스타일
await figma.loadFontAsync({family:'Montserrat', style:'Bold'});
t.characters = '인기 루틴';
await t.setTextStyleIdAsync((await S('b16')).id);
```

- 폰트는 **Montserrat**. Pretendard 는 MCP 환경에 없다.
- `Body/*B18` 은 램프에 없다 → 코드의 18px 은 `Body/*B16~17` 로 연결하고 보고에 적는다.
- 이미지는 **PNG 로 변환해서 업로드**한다. WebP 는 업로드가 200 이어도 Figma 가 디코드 못 한다.
  ```bash
  sips -s format png in.webp --out out.png
  ```
- 아이콘 틴트는 **`VECTOR` 만.** `FRAME/INSTANCE/GROUP` 까지 칠하면 검은 사각형이 된다.
- SVG·인스턴스 크기는 `resize()` 말고 **`rescale()`** — `resize()` 는 stroke-width 를 안 줄인다.

### 4-2. 오토레이아웃 — 코드의 `numberOfLines` 를 그대로 잠근다

**만들 때 잠근다.** 한국어로 맞아 보여도 영어는 +15~40% 늘어나 카드를 뚫는다.

| 코드 | Figma |
|---|---|
| `flex: 1` | `layoutSizingHorizontal = 'FILL'` |
| `numberOfLines={n}` | `maxLines = n` + `textTruncation = 'ENDING'` |
| `flexShrink: 1` (형제가 옆에 붙어야 함) | HUG + `maxWidth = 부모폭 − 형제폭 − gap` |
| 가로 스크롤 리스트 | 컨테이너 `clipsContent = true` |

```js
for (const s of t.getStyledTextSegments(['fontName'])) await figma.loadFontAsync(s.fontName);
t.layoutSizingHorizontal = 'FILL';
t.textAutoResize = 'HEIGHT';
t.maxLines = 2;
t.textTruncation = 'ENDING';
```

- **`resize()` 는 sizing mode 를 FIXED 로 되돌린다** → `resize()` **다음에** FILL 을 준다.
- **미리 `…` 로 잘라 넣지 않는다.** 전문을 넣고 잘림은 `maxLines` 에 맡긴다.

### 4-3. 화면 뼈대

```
FRAME 360×780 · VERTICAL · clipsContent · fill white01
  ├ status bar space (44)          ← 상태바는 제거하되 같은 높이의 빈 프레임을 남긴다
  ├ NavBar (48)                     ← RDSNavBar: height ihp(48), padH iwp(18)
  ├ … 본문 구좌 …                    ← 780 을 넘으면 잘린다 = 실제 스크롤 상태
  └ RDSBottomTab (70)               ← ABSOLUTE · y = 780 − 70
```

하단 탭바:

```js
nav.rescale(1/2.6833333);                 // ① rescale 먼저
root.appendChild(nav);                    // ② 그다음 append (순서 바꾸면 폭이 뭉개진다)
nav.layoutPositioning = 'ABSOLUTE';
nav.x = 0; nav.y = root.height - nav.height;
nav.constraints = {horizontal:'STRETCH', vertical:'MAX'};
```

---

## 5. 폰 · 패드 파생

| 용도 | 크기 | 배율 |
|---|---|---|
| 기준 | 360 × 780 | 1 |
| 폰 | **966 × 2093** | `×2.6833` · 상태바 자리 118 |
| 패드 | **1720 × 2406** | 세로 `×3.0846` · 가로 `×4.7778` |

```js
const PH = 2.6833333, PAD = 3.0846154, W = 1720, H = 2406;
const STRETCH = W / (360 * PAD);            // 1.5489 — 패드는 ihp≠iwp 라 가로만 더 늘린다

// 폰
const a = base.clone(); a.rescale(PH);

// 패드: 균일 확대 → 폭 맞춤 → 가로 패딩/간격만 보정
const b = base.clone();
b.rescale(PAD);
b.resize(W, H);
for (const n of b.findAll(x => x.layoutMode && x.layoutMode !== 'NONE')) {
  n.paddingLeft  *= STRETCH;
  n.paddingRight *= STRETCH;
  if (n.layoutMode === 'HORIZONTAL') n.itemSpacing *= STRETCH;
}
```

파생 후 **폰·패드 각각 스크린샷을 떠서 눈으로 본다.** 고정 높이 박스는 넘쳐도 높이가 안 변해
수치로는 안 잡힌다.

---

## 6. 검증 — 프레임 하나를 끝낼 때마다 돌린다

`use_figma` 로 실행. **방금 그린 프레임 단위**로 본다(섹션 전체로 돌리면 예전 프레임의
미연결까지 섞여 신호가 죽는다).

```js
const KO = /[가-힣]/;
const f = /* 방금 그린 FRAME — 이름으로 찾는다 */;

// DS 인스턴스 내부와 import 한 SVG 하위는 내 책임이 아니다 → 제외
function mine(n) {
  for (let p = n; p; p = p.parent) {
    if (p.type === 'INSTANCE') return false;
    if (p !== n && (p.type === 'GROUP' || (p.type === 'FRAME' && p.name.startsWith('ic/')))) return false;
  }
  return true;
}

const r = {ko: 0, unstyled: [], rawColor: [], overflow: []};
for (const n of f.findAll(() => true)) {
  if (!mine(n)) continue;
  if (n.type === 'TEXT') {
    if (KO.test(n.characters)) r.ko++;                                   // 타언어 프레임은 0
    if (!n.textStyleId) r.unstyled.push(n.characters.slice(0, 16));      // DS 텍스트 스타일 미연결
    if (n.parent && 'width' in n.parent && !n.parent.clipsContent
        && n.x + n.width > n.parent.width + 1) r.overflow.push(n.characters.slice(0, 16));
  }
  if (['FRAME', 'TEXT', 'ELLIPSE', 'RECTANGLE'].includes(n.type) && Array.isArray(n.fills))
    for (const x of n.fills)
      if (x.type === 'SOLID' && !(x.boundVariables && x.boundVariables.color))
        r.rawColor.push(n.name);                                          // DS 색 변수 미연결
}
return JSON.stringify({ko: r.ko,
  unstyled: r.unstyled.length, unstyledSample: r.unstyled.slice(0, 5),
  rawColor: r.rawColor.length, rawColorSample: r.rawColor.slice(0, 5),
  overflow: r.overflow.length, overflowSample: r.overflow.slice(0, 5)});
```

| 항목 | 기대값 |
|---|---|
| `ko` | 타언어 프레임 **0** |
| `unstyled` | **0** — 걸리면 텍스트 스타일을 안 묶은 것 |
| `rawColor` | **0** — 걸리면 색 변수를 안 묶은 것 |
| `overflow` | **0** — 걸리면 4-2 의 `maxLines`/`FILL` 을 안 준 것 |

그리고 **섹션 단위로** 두 가지를 더 센다.

```js
// ① 프레임 개수 — 시작 전 숫자와 같아야 한다 (겹침으로 인한 유실 확인)
// ② 섹션 간 겹침 — 겹치면 프레임이 옆 섹션으로 빨려 들어가 사라진다
const rows = parent.children.filter(c => c.type === 'SECTION')
  .map(s => ({n: s.name, x: s.x, y: s.y, w: s.width, h: s.height, f: s.children.length}));
const ov = [];
for (let i = 0; i < rows.length; i++) for (let j = i + 1; j < rows.length; j++) {
  const a = rows[i], b = rows[j];
  if (a.x < b.x + b.w && b.x < a.x + a.w && a.y < b.y + b.h && b.y < a.y + a.h) ov.push(a.n + ' ∩ ' + b.n);
}
return JSON.stringify({rows, overlaps: ov});   // overlaps 는 빈 배열이어야 한다
```

---

## 7. 보고 — 이 4가지를 반드시 적는다

1. **기준 커밋** — `Routinery/routinery-v2 · origin/dev · <해시> (<날짜>)`
2. **만든 프레임** — 섹션 / 프레임명 / 노드 ID / 크기
3. **직접 지어낸 문구** — 코드 번역 파일에 없어 새로 쓴 것 전부
4. **DS 공백** — 대응 컴포넌트가 없어 직접 그린 것 (`Body/*B18` 부재 등)

---

## 8. 실행 중 지켜야 할 것

- **에러가 나면 그 호출의 변경이 전부 롤백된다.** "이어서 채우기" 금지, 처음부터 다시.
- **응답이 ~20KB 넘으면 SSE 가 깨진다.** 40~60개 단위로 잘라 반환. 이모지는 이스케이프.
- `findAllWithCriteria` 는 **인스턴스 내부와 방금 clone 한 서브트리를 놓친다** → 6장 검증을 반드시 돌린다.
- **노드 ID 를 캐시하지 않는다.** 디자이너가 동시 편집 중이면 ID 가 바뀐다. 매 호출마다 **이름으로 다시 찾는다.**
- 시작 전과 끝난 뒤 **섹션별 프레임 개수를 세어 비교**한다.

---

## 관련

- 배경·실패 기록 → `routinery-v2 · .claude/rules/code-to-figma-sync.md`
- 타언어 대응(한·영 확정 이후) → [앱스샷 다국어 대응 스킬](../appsshot-i18n/SKILL.md) · `routinery-v2 · .claude/rules/i18n-principles.md`
- Figma MCP 일반 규칙 → `routinery-v2 · .claude/rules/mcp-figma.md`
- DS 키·프레임 규격 실측표 → `routinery-v2 · .claude/skills/code-to-figma-design-system/references/project-routinery.md`
- 의존 트리 스크립트 → `scripts/design-sync/deps.py`
