# 전립선 MRI 구조화 판독 도구 — 빌드 지시문 (Claude Code용)

> 이 문서를 Claude Code에 그대로 붙여넣어 작업한다.
> 함께 둘 파일: ① `Thyroid_Finding_Tool_-_K-TIRADS_2021.html`(디자인/아키텍처 레퍼런스), ② `prostate.txt`(병원 정상 판독 양식 = 출력 템플릿). 두 파일이 없어도 이 문서만으로 빌드 가능하도록 작성됨.

---

## 0. 목표

`prostate.txt`(병원 정상 판독지)를 입력받아, 라디올로지스트가 클릭/입력만으로 **PI-RADS v2.1 기반 전립선 MRI 판독문**을 자동 생성하는 **단일 HTML 도구**를 만든다. 첨부된 thyroid K-TIRADS 도구와 **동일한 아키텍처·디자인 언어**를 따른다.

핵심 정책(확정됨):
- **PI-RADS 점수는 수동 입력**(1~5 드롭다운). 시퀀스 기반 자동 산정 엔진은 만들지 않는다.
- **병변 위치는 PI-RADS v2.1 38-섹터 맵**을 클릭으로 선택.
- **EPE는 3단계**(none / possible / definite).
- **Volume·PSAD는 자동계산**, MUL은 수동 입력.

---

## 1. 기술 제약

- **외부 의존성 0.** 단일 `.html` 파일, vanilla JS/CSS만. CDN·프레임워크·빌드툴 금지. 오프라인(파일 더블클릭)으로 완전 동작.
- **localStorage 영속화.** key: `prostateReport_v1`. 단일 `state` 객체 직렬화. 새로고침 후 복원. `deepMerge`로 기본 state와 병합(필드 추가 시 하위호환).
- **다크/라이트 테마 토글**, 선택값 localStorage(`prostateTool_theme`)에 저장. 기본값: 다크.
- **모바일 대응**: viewport meta, 터치타깃 ≥44px, 입력칸 `inputmode="decimal"`.
- 환자정보/PHI는 저장·전송하지 않는다(전부 클라이언트 로컬).

### 디자인 토큰 (thyroid 도구와 동일 — iOS 스타일)
thyroid HTML이 첨부돼 있으면 그 `:root` / `body.light` 블록을 1:1로 가져온다. 없으면 아래로 복제:

```css
:root{
  --bg0:#000;--bg1:#1C1C1E;--bg2:#2C2C2E;--bg3:#3A3A3C;
  --label1:#FFF;--label2:rgba(235,235,245,.6);--label3:rgba(235,235,245,.3);--label4:rgba(235,235,245,.16);
  --fill1:rgba(120,120,128,.36);--fill2:rgba(120,120,128,.28);--fill3:rgba(118,118,128,.22);--fill4:rgba(116,116,128,.14);
  --sep:rgba(84,84,88,.55);--sep-opaque:#38383A;
  --blue:#0A84FF;--blue-bg:rgba(10,132,255,.18);
  --green:#30D158;--green-bg:rgba(48,209,88,.18);
  --red:#FF453A;--red-bg:rgba(255,69,58,.18);
  --orange:#FF9F0A;--orange-bg:rgba(255,159,10,.14);
  --purple:#BF5AF2;--purple-bg:rgba(191,90,242,.18);
  --gray:#8E8E93;
  --r-sm:10px;--r-md:12px;--r-lg:16px;--r-xl:20px;
  --shadow-card:0 1px 3px rgba(0,0,0,.35),0 4px 16px rgba(0,0,0,.22);
}
body.light{
  --bg0:#F2F2F7;--bg1:#FFF;--bg2:#F2F2F7;--bg3:#E5E5EA;
  --label1:#000;--label2:rgba(60,60,67,.6);--label3:rgba(60,60,67,.3);--label4:rgba(60,60,67,.18);
  --fill1:rgba(120,120,128,.2);--fill2:rgba(120,120,128,.16);--fill3:rgba(118,118,128,.12);--fill4:rgba(116,116,128,.08);
  --sep:rgba(60,60,67,.29);--sep-opaque:#C6C6C8;
  --blue:#007AFF;--blue-bg:rgba(0,122,255,.12);--green:#34C759;--green-bg:rgba(52,199,89,.12);
  --red:#FF3B30;--red-bg:rgba(255,59,48,.1);--orange:#FF9500;--orange-bg:rgba(255,149,0,.1);
  --purple:#AF52DE;--purple-bg:rgba(175,82,222,.12);
  --shadow-card:0 1px 3px rgba(0,0,0,.10),0 4px 16px rgba(0,0,0,.07);
}
```
폰트: `-apple-system, BlinkMacSystemFont, 'SF Pro Text', 'Helvetica Neue', Arial, sans-serif`.
컴포넌트 룩(카드=`--r-lg`+`--shadow-card`, label-cell, radio-group, segmented control, dialog overlay, footer)도 thyroid와 동일 idiom으로.

---

## 2. 화면 구조

상단 고정바: 탭 버튼 + 우측에 `[판독문]`(primary, blue), `[전체 초기화]`(red text), `[테마토글]`.

**탭 2개:**
1. **Prostate** — Clinical Information / Techniques / Measurements / PI-RADS Lesions
2. **Other findings** — SVI / bladder / adjacent organ / LN / bone / miscellaneous

각 탭 상단에 해당 섹션 `초기화` 버튼.

---

## 3. 섹션 상세

### 3-1. Clinical Information (카드)
- **Previous biopsy history**: 라디오 `yes / no`
- **Prostate cancer**: 라디오 `confirmed / not confirmed`
- **Biopsy results**: `Positive core(s):` 텍스트, `Gleason score:` 텍스트
- **Active surveillance**: 텍스트(또는 yes/no 라디오 + 메모)

### 3-2. Techniques (카드)
- 고정 기본문(편집 가능 textarea), 기본값:
  `Axial DWI, axial T1WI, T2WI (axial, sagittal, coronal), axial DCE MRI, axial contrast-enhanced T1WI`
- **IV contrast media administration**: 토글 `yes / no` (기본 yes)

### 3-3. Measurements (카드) — ★ 자동계산 핵심
입력 필드:
- **Prostate dimensions**: `L` × `W` × `H` 세 칸, **단위 mm** (`inputmode="decimal"`)
- **PSA (ng/mL)**: 숫자 입력
- **Membranous urethral length (mm)**: 숫자 직접 입력

자동계산(읽기전용 표시, 입력 시 실시간 갱신):
- **Estimated prostate volume (cc)**
  - ellipsoid: `volume_cc = (L_mm * W_mm * H_mm * 0.52) / 1000`
  - 0.52 ≈ π/6. **단위 주의**: mm³ 결과를 cc로 만들려고 `/1000`. (= L,W,H를 cm로 환산 후 ×0.52 와 동일)
  - 표시는 소수 1자리. 예) L 45 · W 38 · H 35 → `volume = 45*38*35*0.52/1000 = 31.1 cc`
- **PSAD (ng/mL/cc)**
  - `PSAD = PSA / volume_cc`, 소수 **2자리**. 예) PSA 6.2, vol 31.1 → `0.20`
  - PSA 또는 volume 미입력 시 빈칸/`—`.

> 표시 정책: 리포트의 volume 줄에는 산출값과 함께 입력치수를 괄호로 부기한다 — `Estimated prostate volume (cc): 31.1 (45 × 38 × 35 mm)`. (불필요하면 괄호 제거 옵션)

### 3-4. PI-RADS Lesions (병변 N개, +추가/삭제)
thyroid의 nodule 모듈 패턴 재사용. 각 병변 카드 필드:
- **Location**: PI-RADS v2.1 **38-섹터 맵**에서 선택(아래 §4). 선택된 섹터는 카드에 chip으로 표시.
- **PIRADS score**: 드롭다운 `1 / 2 / 3 / 4 / 5` (수동)
- **Maximal tumor length (mm)**: 숫자 입력
- **Extraprostatic extension**: 라디오 `none / possible / definite`
- **Comment**: textarea(선택)

병변 헤더에 `Abnormality No.N`과 선택된 PI-RADS 점수 배지 표시(색: 5=red,4=orange,3=green,2/1=blue/gray).
**Index lesion**은 자동 산정(§5) — 카드에 `Index` 표식.

### 3-5. Other findings (탭2)
6개 항목, 각각 텍스트 입력이며 **기본값 `none`** 으로 프리필(이상 시 편집):
- Seminal vesicle invasion
- Abnormality of the urinary bladder
- Invasion to adjacent organ
- Significant lymph node enlargement
- Bone metastases
- Miscellaneous

---

## 4. PI-RADS v2.1 섹터 맵 (38 섹터)

> 주의: "39 섹터"는 PI-RADS **v2(2015)** 숫자. **v2.1은 전립선 38 섹터**(+ SV 2, 막성요도 1 = 총 41). v2.1에서 base PZ에 좌·우 posterior-medial PZ가 추가됨. 이 도구는 전립선 **38 섹터**만 lesion localizer로 사용(SV·요도는 별도 섹션에서 처리).

### 레벨·region 구성
- **Base (14)**: 각 side(R/L)에 `PZa, PZpm, PZpl, TZa, TZp, CZ, AFS`
- **Mid (12)**: 각 side에 `PZa, PZpm, PZpl, TZa, TZp, AFS`  (CZ 없음)
- **Apex (12)**: 각 side에 `PZa, PZpm, PZpl, TZa, TZp, AFS`  (CZ 없음)

> CZ는 base에만 존재. 약어: PZa=anterior, PZpm=postero-medial, PZpl=postero-lateral, TZa/TZp=transition anterior/posterior, AFS=anterior fibromuscular stroma.

### 데이터 모델
- 섹터 key: `` `${level}_${region}_${side}` `` (예: `base_PZpm_R`)
- 가독 라벨: `Rt base PZpm`, `Lt mid TZa` (side: R→Rt / L→Lt)
- 출력/정렬 순서: level(base→mid→apex) → side(R→L) → region(PZa,PZpm,PZpl,TZa,TZp,CZ,AFS) 고정 순.

### 맵 UI (반드시 해부학적 배치)
- **3개 axial 패널**(Base / Mid / Apex)을 가로로 나란히. 모바일에서는 세로 스택.
- **anterior = 위, posterior = 아래**. **환자 Right = 화면 왼쪽**, Left = 오른쪽(영상 판독 관례). 각 패널에 `R | L`, `Ant↑/Post↓` 표시.
- 각 패널은 4열 CSS grid, **grid-template-areas**로 다음 배치(위=anterior, 아래=posterior):

```
행1(anterior):  AFS-R  AFS-R  AFS-L  AFS-L
행2:            PZa-R  TZa-R  TZa-L  PZa-L
행3:            PZpl-R TZp-R  TZp-L  PZpl-L
행4(base만):     .     CZ-R   CZ-L    .
행5(posterior): PZpm-R PZpm-R PZpm-L PZpm-L
```
- 즉 외측열(1,4)=PZ 측방, 내측열(2,3)=TZ/CZ, 최상단=AFS, 최하단 중앙=PZpm. mid/apex는 행4(CZ) 제거.
- **zone 색상 코딩**: PZ=blue-bg, TZ=green-bg, CZ=purple-bg, AFS=fill(gray) 톤. 선택 시 해당 zone accent로 채우고 체크 표시.
- 셀 클릭 = **현재 편집 중인 병변**의 섹터 토글. 한 병변이 여러 섹터 가질 수 있음(다중 선택).
- 각 셀에 region 약어 텍스트 표기(라벨이 안전망).

> 구현은 SVG path보다 **HTML+CSS grid**(rect형 클릭 셀) 권장 — 히트테스트 안정적, 모바일·테마 대응 쉬움. 추후 v2에서 정식 axial 다이어그램(SVG)으로 업그레이드 여지만 남겨둘 것.

---

## 5. Index lesion 자동 산정

- 후보: 섹터 또는 PIRADS 점수가 입력된 병변.
- 규칙: **PIRADS 점수 내림차순** → 동점 시 **Maximal tumor length 내림차순**. 최상위 1개가 index lesion.
- 병변이 0개(또는 후보 없음)면 conclusion은 "No suspicious intraprostatic lesion."

---

## 6. 판독문 출력 (★ `prostate.txt` 구조 1:1)

`[판독문]` 클릭 → 다이얼로그에 아래 형식 텍스트 생성 + `복사` 버튼(클립보드, 성공 토스트). 빈 항목 처리 규칙은 §7.

```
== Finding

<Clinical Information>
Previous biopsy history: [yes/no]
Prostate cancer: [confirmed/not confirmed]
 Biopsy results
  - Positive core(s): [값]
  - Gleason score: [값]
 Active surveillance: [값]

<Techniques>
[techniques 기본문]
IV contrast media administration: [yes/no]

<Measurements>
PSA (ng/mL): [값]
Estimated prostate volume (cc): [산출값] ([L] × [W] × [H] mm)
PSAD (ng/mL/cc): [산출값]
Membranous urethral length (mm): [값]

<PIRADS>
Number of suspicious lesions: [N]

 Abnormality No.1: located at [가독 섹터 목록]
 PIRADS score: [n]
 Maximal tumor length (mm): [값]
 Extraprostatic extension: [none/possible/definite]
 [Comment: 값]            ← comment 있을 때만

 Abnormality No.2: ...     ← 병변 수만큼 반복

<Other findings>
Seminal vesicle invasion: [none/값]
Abnormality of the urinary bladder: [none/값]
Invasion to adjacent organ: [none/값]
Significant lymph node enlargement: [none/값]
Bone metastases: [none/값]
Miscellaneous: [none/값]


== Conclusion

Index lesion of PIRADS score [X] at [가독 섹터 목록] with [no/possible/definite] extra-prostatic extension.
```

- 병변이 없으면 `<PIRADS>` 본문 `Number of suspicious lesions: 0` + Conclusion 은:
  ```
  == Conclusion

  No suspicious intraprostatic lesion.
  ```
- Conclusion의 EPE 어구 매핑: `none → no`, `possible → possible`, `definite → definite` ("...with no/possible/definite extra-prostatic extension.").
- `located at`의 섹터 목록은 §4 가독 라벨을 §4 정렬 순서로 `, ` 조인.

---

## 7. 동작 규칙

- **실시간 자동저장**: 모든 입력 change/input 시 state 갱신 → `saveState()`.
- **자동계산 반응성**: L/W/H/PSA 변경 시 volume·PSAD 즉시 재계산·표시.
- **Validation**(판독문 생성 전): 점수 또는 섹터가 입력된 병변에 대해 ─ 섹터 미선택 시 "Abnormality N: location required", Maximal length 미입력 시 경고. 에러 있으면 다이얼로그로 목록 표시하고 생성 중단.
- **초기화**: 섹션별 초기화 + `전체 초기화`(확인 다이얼로그 후 state 리셋·localStorage 클리어).
- **빈 값 출력 정책**: 미입력 텍스트 필드는 빈칸 또는 라벨만 출력(앱이 멋대로 값 추정 금지). Other findings는 기본 `none` 유지.

---

## 8. 수용 기준 (테스트 케이스)

1. **Volume**: L=45, W=38, H=35(mm) → `31.1 cc` 표시 & 리포트 `31.1 (45 × 38 × 35 mm)`.
2. **PSAD**: PSA=6.2, 위 volume → `0.20`. PSA만 있고 volume 없으면 `—`.
3. **MUL**: 14 입력 → 리포트 그대로 `14`.
4. **섹터 맵**: base 패널에서 `PZpm-R`, `PZpl-R` 클릭 → 병변에 `Rt base PZpm, Rt base PZpl` chip & 리포트 located at에 동일.
5. **Index**: 병변A(PIRADS 4, 12mm), 병변B(PIRADS 5, 9mm) → Conclusion의 index = 병변B(점수 우선). 동점이면 길이 큰 쪽.
6. **무병변**: 병변 0 → Conclusion "No suspicious intraprostatic lesion."
7. **영속성**: 입력 후 새로고침 → 전부 복원. 테마 토글 후 새로고침 → 테마 유지.
8. **오프라인**: 파일 더블클릭(네트워크 없음)으로 전 기능 동작, 클립보드 복사 동작.

---

## 9. 산출물

- `prostate_pirads_tool.html` 단일 파일.
- 푸터: 작성자명 + `Prostate MRI Structured Report · PI-RADS v2.1` 표기.
- 코드 주석으로 섹터 데이터 모델·자동계산 공식 명시(추후 유지보수용).

---

### 빌드 순서 권장
1) thyroid 셸 복제(디자인토큰·탭·state·dialog·theme 골격) → 내용 제거
2) Clinical / Techniques / Measurements(자동계산 포함)
3) PI-RADS lesion 모듈 + 38-섹터 맵 ← 가장 공들일 부분
4) Other findings
5) 판독문 생성(§6) + validation
6) §8 케이스로 검증
