# GUreport — Prostate MRI Structured Report · PI-RADS v2.1

Single-file HTML tool for structured prostate MRI reporting. No server, no install, no dependencies — open the file in a browser and start reporting.

---

## Tool

### Prostate MRI Structured Report · PI-RADS v2.1
**File:** `prostate_pirads_tool.html`

PI-RADS v2.1 기반 전립선 MRI 판독문 자동 생성 도구.

**Features**
- **PI-RADS v2.1 38-섹터 맵** — Base / Mid / Apex 3패널, SVG axial diagram, 해부학적 배치 (Ant ↑ · Post ↓ · R ← → L), 클릭으로 다중 섹터 선택
- **Zone 색상 구분** — PZ (blue) · TZ (green) · CZ (purple) · AFS (gray)
- **Volume / PSAD 실시간 자동계산** — Ellipsoid formula (L × W × H × 0.52 / 1000)
- **PI-RADS 점수 수동 입력** (드롭다운 1–5), EPE 3단계 (none / possible / definite)
- **Index lesion 자동 산정** — PI-RADS 점수 내림차순 → 동점 시 최대 길이 내림차순
- **판독문 원클릭 생성** — 병원 정상 판독지 형식(`prostate.txt`) 1:1 출력, 클립보드 복사
- **Validation** — 섹터 미선택 / 점수 미입력 / 길이 미입력 오류 다이얼로그
- **localStorage 영속화** (`prostateReport_v1`) · 새로고침 후 전체 복원
- **다크 / 라이트 테마** 토글 (기본: 다크)
- **모바일 대응** — viewport meta, 터치타깃 ≥ 44 px, 수평 스크롤 섹터 맵

**Sections**
| Tab | 섹션 |
|-----|------|
| Prostate | Clinical Information · Techniques · Measurements · PI-RADS Lesions |
| Other findings | SVI · Bladder · Adjacent organ · LN · Bone · Miscellaneous |

**Report format (출력 예시)**
```
== Finding

<Clinical Information>
Previous biopsy history: yes
Prostate cancer: confirmed
 Biopsy results
  - Positive core(s): 3/12
  - Gleason score: 3+4
 Active surveillance: no

<Techniques>
Axial DWI, axial T1WI, T2WI (axial, sagittal, coronal), axial DCE MRI, axial contrast-enhanced T1WI
IV contrast media administration: yes

<Measurements>
PSA (ng/mL): 6.2
Estimated prostate volume (cc): 31.1 (45 × 38 × 35 mm)
PSAD (ng/mL/cc): 0.20
Membranous urethral length (mm): 14

<PIRADS>
Number of suspicious lesions: 1

 Abnormality No.1: located at Rt base PZpm, Rt base PZpl
 PIRADS score: 4
 Maximal tumor length (mm): 12
 Extraprostatic extension: none

<Other findings>
Seminal vesicle invasion: none
...

== Conclusion

Index lesion of PIRADS score 4 at Rt base PZpm, Rt base PZpl with no extra-prostatic extension.
```

---

## PI-RADS v2.1 섹터 맵 구조

```
          ← R            L →
Ant ↑  [ AFS·R  ][ AFS·R  ][ AFS·L  ][ AFS·L  ]
       [ PZa·R  ][ TZa·R  ][ TZa·L  ][ PZa·L  ]
       [ PZpl·R ][ TZp·R  ][ TZp·L  ][ PZpl·L ]
(base) [        ][ CZ·R   ][ CZ·L   ][        ]
Post ↓ [  PZpm·R (×2)     ][  PZpm·L (×2)     ]
```

- **Base**: 14 sectors (PZa, PZpm, PZpl, TZa, TZp, CZ, AFS × R/L)
- **Mid**: 12 sectors (CZ 없음)
- **Apex**: 12 sectors (CZ 없음)
- **Total**: 38 prostate sectors

---

## Usage

```
# 브라우저에서 직접 열기 (네트워크 불필요)
open prostate_pirads_tool.html
```

- 외부 CDN / 프레임워크 **없음** — 순수 HTML + Vanilla JS/CSS
- 오프라인 완전 동작
- PHI(환자 정보) 서버 전송 없음 — 모든 데이터는 로컬 `localStorage`에만 저장

---

## Tech

| 항목 | 내용 |
|------|------|
| 언어 | HTML5 · CSS3 · Vanilla JS (ES6+) |
| 의존성 | 없음 (zero external dependencies) |
| 저장 | localStorage (key: `prostateReport_v1`, `prostateTool_theme`) |
| 지원 브라우저 | Chrome · Edge · Safari · Firefox (최신 버전) |
| 모바일 | iOS Safari · Android Chrome |

---

## Author

**Hunjong Lim, MD**  
Department of Radiology  

---

*Prostate MRI Structured Report · PI-RADS v2.1*
