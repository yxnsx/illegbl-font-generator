# 자소·조합 한글 폰트 제작 웹앱 — 기획서

> 추후 기능 수정/추가 시 참고용 문서. 현재 버전 **v1.0.0** 기준.

- **파일명**: `jaso-hangul-font-maker.html`
- **배포 형태**: 단일 HTML 파일 (약 168KB)
- **외부 의존성**: opentype.js (CDN), Google Fonts (Gowun Batang, Noto Serif KR, JetBrains Mono)
- **저장**: 브라우저 IndexedDB (서버 없음, 모든 작업이 로컬에서 수행)

---

## 1. 제품 개요

### 1.1 목적

웹 브라우저만으로 한글 폰트(TTF)를 제작할 수 있는 도구. FontForge나 Glyphs 같은 데스크탑 폰트 에디터를 설치하기 어려운 사용자가, 11벌식 자소 조합 방식으로 한글 11,172자 폰트를 만들 수 있게 하는 것이 목표.

### 1.2 타겟 사용자

| 사용자군 | 시나리오 |
|---------|---------|
| 손글씨 폰트를 만들고 싶은 일반인 | 종이에 손글씨 → 사진 → 참조 이미지로 깔고 따라 그리기 |
| 폰트 디자인 입문자 | 한글 음절 구조와 자소 조합 원리를 학습하며 시도 |
| 디자이너 | 빠른 프로토타입 (정식 작업은 Glyphs/FontLab으로 옮김) |
| 교육 현장 | 한글 자모 구조 학습 도구 |

### 1.3 핵심 가치 제안

- **설치 불필요** — 단일 HTML, 브라우저만 있으면 됨
- **데이터 로컬 보관** — IndexedDB 자동 저장, 서버 전송 없음
- **태블릿 지원** — iPad/안드로이드 펜으로 작업 가능
- **빠른 시작** — 샘플 자소 30여 개로 즉시 동작 확인

### 1.4 명시적 비목표 (Non-goals)

이 도구는 다음을 하지 않으며, 필요 시 사용자가 다른 도구로 옮겨가야 함:

- 상용 명조·고딕 수준의 정밀한 outline 품질 (단일 stroke 방식이라 구조적 한계)
- GSUB 합자 (옛한글, 첫가끝 NFD 진짜 합자 등)
- 자동 커닝 / 자동 힌팅 (수동 커닝은 지원)
- 다중 굵기/스타일의 일괄 관리 (한 번에 하나의 스타일만)
- 협업 기능 (멀티 유저, 클라우드 동기화)
- 펜 압력 감지 / 서체 스트로크 변형
- 진정한 벡터화 (Potrace 수준의 비트맵→벡터 변환)

---

## 2. 화면 구성 및 기능 명세

전체 **7개 페이지** 탭으로 구성. 상단 네비게이션에서 전환.

```
[01 그리기] → [02 위치] → [03 조합표] → [04 본문] → [05 커닝] → [06 정보] → [07 내보내기]
   작업          조합 조정     검수          가독성      미세조정     메타        출력
```

### 2.1 그리기 (Draw) 페이지

**역할**: 자소 하나하나를 캔버스에서 그리는 핵심 작업 화면.

**3단 레이아웃** (좌·중·우):

#### 좌측 패널: 자소 선택
- 4개 카테고리 탭: 초성 / 중성 / 종성 / 영문·특수문자
- 각 카테고리는 11벌식 분류에 따라 set별로 그리드 표시
  - 초성: 5벌 (V_NJ, V_J, H_NJ, H_J, C)
  - 중성: 2벌 (NJ, J)
  - 종성: 4벌 (V, H, C, S)
- 셀 우상단 빨간 점 = 이미 그려진 자소
- 셀 클릭 → 해당 자소가 캔버스에 로드됨

#### 중앙: 캔버스 작업 영역
- **현재 자소 표시**: 글자 + 분류 메타정보 (예: "ㄱ · 초성 · 세로형 + 받침없음")
- **편집 모드 토글**: 자유 그리기 / 베지어 편집
- **캔버스 (500×500px, EM 1000 기준)**
  - 4중 레이어: 참조 이미지 → 캔버스 → 가이드 SVG → 베지어 핸들 SVG
- **그리기 도구**:
  - 실행취소/재실행 (무한, 최대 100단계)
  - 전체 지우기
  - 두께 슬라이더 (2~80, EM 좌표 기준)
  - **벡터화** 버튼 (자유 그리기를 베지어로 변환)
  - **저장** — 클릭 시 현재 자소를 IndexedDB에 저장한 뒤, **다음 미완성 자소로 자동 이동**(같은 카테고리 · 같은 set 우선, 채워지면 다음 set/카테고리로). 캔버스는 자동으로 비워지고 좌측 자소 탭도 따라서 전환됨. 모든 자소가 채워졌으면 토스트로 "모든 자소 완성!" 알림.
- **가이드 토글** (4종):
  - 격자 (50px 단위)
  - 베이스라인 + x-height
  - 한글 비례 가이드 (자소 타입별 권장 영역)
  - 참조 자소 (같은 글자의 다른 set 자소를 흐리게 표시)
- **참조 이미지 툴바**:
  - PNG 업로드 (PNG/JPG/WebP/SVG 가능, 5MB 권장)
  - 자동 맞춤 (비율 유지)
  - 제거
  - 투명도 슬라이더 (5~100%, 기본 35%)
  - 이동/크기조정 토글 (켜면 그리기 잠시 중단)
  - 비율 고정 토글 (기본 ON, 모서리 드래그 시 비율 유지)

#### 우측 패널: 미리보기 & 히스토리
- **미리보기**: 사용자 입력 글자(기본 "가나다")가 실시간으로 합성된 SVG 표시
- **히스토리 패널**: 단계별 시점으로 되돌리기 가능
- **단축키 안내**: Ctrl+Z/Y/S, Delete

### 2.2 위치 (Arrange) 페이지

**역할**: 11벌식 케이스별로 자소가 합쳐질 때 위치/크기를 미세조정.

**기본 케이스 6개**: 가, 각, 고, 곡, 과, 광 (각각 다른 choSet/jungSet/jongSet 조합 대표)

- 캔버스에 음절이 합성되어 표시됨
- 자소(초성/중성/종성)를 클릭하면 선택 → 빨간 바운딩 박스 + 모서리 핸들
- **드래그**로 자소 이동, **모서리 핸들**로 크기 조정
- 우측 패널에서 X/Y/가로/세로 슬라이더로 정밀 조정
- 케이스 간 복사/붙여넣기 (한 케이스에서 조정한 값을 다른 케이스로)
- 기본값 복원 버튼

### 2.3 조합표 (Grid) 페이지

**역할**: 11,172자 전체를 한눈에 보고 어색한 글자를 발견.

- **필터**: 전체 / KS1001자 / 최빈500자 / 미완성만 / 완성만
- **정렬**: 유니코드순 / 빈도순 / 미완성 먼저
- **검색**: 글자 직접 입력으로 필터
- **셀 크기 조정** (40~120px 슬라이더)
- 성능 보호: 한 번에 최대 2,000셀 표시
- 셀 클릭 시 그 글자의 초성으로 그리기 페이지 이동
- 우측 통계: 전체/KS1001/초성/중성/종성 진행도

### 2.4 본문 (Typeset) 페이지

**역할**: 실제 문장으로 가독성 테스트.

- 6개 샘플 문장 (한글 자모 노래, 훈민정음, 진달래꽃, 헌법, 판그램 등)
- 직접 입력 textarea
- 크기 (14~80px), 행간 (1.0~2.5), 자간 (-30~+30) 조정
- 미완성 글자는 회색 fallback 표시

### 2.5 커닝 (Kerning) 페이지

**역할**: 글자쌍별로 시각적 간격을 미세 조정. 사용자가 직접 페어를 추가하고 슬라이더로 보정값을 정함.

**좌측 패널**:
- 페어 추가 입력 (왼쪽 글자 / 오른쪽 글자, Enter로 추가)
- 자주 쓰이는 페어 추천 버튼 (한글 흔한 조합, 받침+모음 시작, 한영 경계, 영문 페어, 숫자-한글 등)
  - 추가된 페어는 추천 목록에서 회색 처리되어 중복 추가 방지
- 통계 (총 페어 수, 조정값 합계)

**중앙 영역**:
- 페어별 행: [실시간 미리보기] [슬라이더 -200~+200] [수치] [삭제]
- 미리보기는 두 글자 사이 spacer로 즉시 반영
- 슬라이더 조작 → 디바운스(300ms) → IndexedDB 저장
- 음수값은 빨간색으로 표시 (글자 좁히기)

**단위**: EM의 1/1000 (예: -50 = EM 1000인 폰트에서 글자가 5% 정도 가까워짐)

**저장 위치**: IndexedDB `kerning` store, key=`${left}|${right}`, value=숫자

**출력 시**: 내보내기 페이지의 "사용자 정의 커닝 페어 포함" 옵션이 켜져 있으면 TTF의 GPOS 테이블에 포함됨 (opentype.js의 `font.kerningPairs` 사용)

### 2.6 정보 (Meta) 페이지

**역할**: 폰트 메타데이터 입력.

- **기본 정보**: 패밀리명, 스타일(Regular/Light/Medium/Bold/Thin), 버전
- **저작자**: 디자이너, 제작사, 웹사이트
- **라이선스**: 종류 선택 시 (OFL/MIT/CC0/CC-BY/Proprietary/Custom) 자동 템플릿 채움
- **저작권 표시**, **설명**

### 2.7 내보내기 (Export) 페이지

**역할**: TTF 다운로드 및 프로젝트 백업.

- 요약 정보 표시 (패밀리, 한글 음절 개수, 영문 개수, 총 글리프 수)
- **파일 형식**: TTF 단일 출력 (TrueType, 대부분의 OS·워드프로세서 호환)
  - OTF/WOFF/WOFF2가 필요한 경우 별도 변환기(예: FontSquirrel Webfont Generator) 사용 권장
- **옵션**:
  - NFD 자모 입력 지원 (호환자모 영역 매핑)
  - 사용자 정의 커닝 페어 포함 (커닝 페이지에서 편집한 값 적용)
  - 미완성 글자 제외 (subset)
- **프로젝트 백업/복원**: JSON 파일로 전체 데이터 export/import (자소·위치·메타·커닝 모두 포함)

---

## 3. 11벌식 분류 체계

한글 음절은 (초성, 중성, 종성?)의 조합. 각 자소는 주변 자모 환경에 따라 **여러 형태**가 필요. 본 도구는 **11벌식**을 채택.

### 3.1 분류 기준

| 카테고리 | set 수 | 기준 |
|---------|--------|------|
| **초성** | 5 | (세로형 V × 받침유무) + (가로형 H × 받침유무) + 복합형 C(통합) |
| **중성** | 2 | 받침 유무 |
| **종성** | 4 | 앞 중성의 형태 (3종 + 간략형 1종) |

**합계 11벌** (5 + 2 + 4).

> **왜 초성이 6벌이 아니라 5벌인가?**
> 단순 곱하기로는 V/H/C × NJ/J = 6벌이 나오지만, **복합형(C)은 받침 유무 구분 없이 1벌로 통합**됩니다. 복합형 중성(ㅘ, ㅝ, ㅢ 등)은 그 자체가 가로+세로 결합이라 초성이 차지할 공간이 V형보다 좁고 압축적이며, 받침 유무에 따른 공간 변화가 두드러지지 않기 때문입니다. 한컴/Sandoll 등 업계 표준도 동일하게 5벌입니다.

### 3.2 set 코드와 의미

```
초성:
  V_NJ  vertical jung, no jong   (예: 가)
  V_J   vertical jung, with jong  (예: 각)
  H_NJ  horizontal jung, no jong  (예: 고)
  H_J   horizontal jung, with jong (예: 곡)
  C     complex jung (받침 유무 통합) (예: 과, 광, 괴, 괵)

중성:
  NJ    no jong  (예: 가)
  J     with jong (예: 각)

종성:
  V     after vertical jung   (예: 각)
  H     after horizontal jung (예: 곡)
  C     after complex jung    (예: 광)
  S     simplified            (예비, 현재 미사용)
```

### 3.3 분류 함수 (구현)

```javascript
VERTICAL_JUNG  = {ㅏ ㅐ ㅑ ㅒ ㅓ ㅔ ㅕ ㅖ ㅣ}
HORIZONTAL_JUNG = {ㅗ ㅛ ㅜ ㅠ ㅡ}
COMPLEX_JUNG    = {ㅘ ㅙ ㅚ ㅝ ㅞ ㅟ ㅢ}

getJungType(j)         → 'V' | 'H' | 'C'
getChoSet(j, hasJong)  → 'V_NJ' | 'V_J' | 'H_NJ' | 'H_J' | 'C'
getJungSet(hasJong)    → 'NJ' | 'J'
getJongSet(j)          → 'V' | 'H' | 'C'
```

### 3.4 벌식 비교: 11벌식이 sweet spot인 이유

| 항목 | 8벌식 | **11벌식 (현재)** | 16벌식 | 24벌식 |
|------|------|----------|--------|--------|
| 초성 | 2벌 (V/H) | 5벌 | 8벌 | 12벌 |
| 중성 | 2벌 | 2벌 | 4벌 | 8벌 |
| 종성 | 4벌 | 4벌 | 4벌 | 4벌 |
| **자모 조합 수** | 19×2 + 21×2 + 27×4 = **188** | 19×5 + 21×2 + 27×4 = **245** | 19×8 + 21×4 + 27×4 = **344** | 19×12 + 21×8 + 27×4 = **504** |
| 작업 시간 (자소당 5분) | ~16시간 | ~20시간 | ~29시간 | ~42시간 |
| 품질 | 학습용·캐주얼 폰트 한정 | **본문체로 충분** | 고급 본문체 | 상용 명조·고딕 수준 |
| 대표 폰트 | 손글씨 폰트 다수 | 나눔고딕, 본고딕 KR | 산돌 명조네오, 마루부리 | SM 신신명조, AG 최제우체 |
| 한계 | 받침 압축이 부자연스러움 | 같은 set 내 자모별 미묘한 차이 무시 | 거의 모든 케이스 자연스러움 | 한자 손글씨 수준 정밀도 |

**11벌식 채택 근거**: 8벌식 대비 자소 수가 30% 정도만 늘어나면서 본문 품질에 도달함. 24벌식까지 가면 작업량은 두 배지만 본문에서의 차이를 일반 사용자가 체감하기는 어려움. 본 도구의 타겟 사용자(손글씨 폰트, 입문자)에게는 11벌식이 작업 부담과 결과 품질의 균형점.

**확장 방법**: 16/24벌식으로 확장하려면 다음만 수정하면 됨:
- `CHO_SETS`, `JUNG_SETS`, `JONG_SETS` 상수 배열
- `CHO_SET_NAMES`, `JUNG_SET_NAMES`, `JONG_SET_NAMES` 한글명
- `getChoSet()`, `getJungSet()`, `getJongSet()` 분류 함수
- `DEFAULT_TRANSFORMS` 기본 위치 변환값 추가

데이터 호환성: 기존 11벌식 자소는 그대로 보존되므로 점진적 전환 가능.

---

## 4. 데이터 모델

### 4.1 IndexedDB 스키마

데이터베이스명: `jasoFontMaker`, 버전 2.

| Object Store | Key | Value | 용도 |
|-------------|-----|-------|------|
| `jaso` | `${type}_${char}_${set}` | `{key, strokes, updated}` | 한글 자소 (초/중/종) |
| `jaso` | `__refimg__${jasoKey}` | `{key, isRefImg, src, x, y, w, h, opacity, lockRatio, naturalRatio}` | 자소별 참조 이미지 (같은 store 사용, 키 접두사로 구분) |
| `latin` | `char` | `{char, strokes}` | 영문/숫자/특수문자 |
| `arrange` | `${choSet}\|${jungSet}\|${jongSet}` | `{key, cho?, jung?, jong?}` | 케이스별 위치 조정값 |
| `meta` | `${field}` | `{key, value}` | 폰트 메타데이터 (필드별 1행) |
| `kerning` | `${left}\|${right}` | `{pair, value}` | 사용자 커닝 페어 (value는 EM/1000 단위, -200~+200) |
| `history` | autoIncrement | (현재 미사용, 향후 자소별 변경 이력 저장용) | 예약 |

### 4.2 자소 데이터 구조

```javascript
jasoStore[key] = {
  strokes: [
    {
      width: 60,           // EM 좌표 기준 두께
      type: 'free' | 'bezier',
      points: [[x, y], ...],  // EM 좌표 (0~1000)
      bezier?: [
        {
          x, y,             // anchor
          cp1?: {x, y},     // 이전 segment의 cp2와 짝
          cp2?: {x, y}
        }
      ]
    }
  ],
  updated: timestamp
}
```

### 4.3 위치 조정 데이터

```javascript
arrangeStore[`${choSet}|${jungSet}|${jongSet || 'none'}`] = {
  cho?:  {x, y, sx, sy},  // EM 좌표, scale은 0.1~1.5
  jung?: {x, y, sx, sy},
  jong?: {x, y, sx, sy}
}
```

기본값은 `DEFAULT_TRANSFORMS` 테이블에 정의. arrangeStore에 항목 있으면 우선.

### 4.4 좌표계

- **캔버스 좌표**: 0~500 (UI 표시용)
- **EM 좌표**: 0~1000 (저장 및 폰트 출력용)
- 변환 시 `ratio = EM_SIZE / CANVAS_SIZE = 2`
- 폰트 메트릭: ASCENT 800, DESCENT -200, EM 1000

---

## 5. 기술 스택과 핵심 알고리즘

### 5.1 외부 의존성

| 라이브러리 | 버전 | 용도 |
|-----------|------|------|
| opentype.js | 1.3.4 | TTF 생성 (Glyph, Path, Font 클래스) |
| Google Fonts | - | UI 폰트 (Gowun Batang, Noto Serif KR, JetBrains Mono) |

직접 구현한 알고리즘은 다음과 같음:

### 5.2 자유 그리기 → 베지어 변환

`pointsToBezier(points)` 함수:

1. **Douglas-Peucker** 단순화 (tolerance=5)로 점 개수 축소
2. **Catmull-Rom 스플라인**으로 부드러운 곡선 생성
3. 각 점에 anchor + cp1/cp2 (이전·다음 점 차이의 1/6) 추가
4. 결과: 사용자가 베지어 모드에서 핸들로 직접 편집 가능

### 5.3 음절 합성 (SVG)

`composeSyllableSVG(char)`:

1. 유니코드에서 초/중/종 인덱스 분리
2. 각 자소의 set 결정
3. `transformPath(strokeData, t)`로 EM 좌표 + transform 적용한 SVG path 생성
4. 3개 자소를 하나의 `<svg>`에 합침

### 5.4 Stroke → Outline 변환 (TTF용)

`strokesToOpenTypePath(strokeData, t)` — **stroke offset 방식**

각 stroke를 다음 단계로 단일 닫힌 path로 변환:

1. **샘플링**: 베지어는 곡률 적응적으로 샘플링 (직선부 12점, 곡선부 최대 40점). 자유 그리기는 이동 평균으로 평활화.
2. **법선 벡터 계산**: 각 샘플 점에서 양옆 점의 평균 방향에 수직인 단위 벡터를 구함. 부드러운 코너에서 자연스러운 폭 유지.
3. **양쪽 평행 곡선**: 법선 방향 +r, -r로 두 점씩 생성 → 왼쪽/오른쪽 outline.
4. **인접 중복 제거**: 좁은 코너에서의 self-intersection 일부 방지.
5. **닫힌 path 조립**: 왼쪽 곡선(Catmull-Rom 베지어) → 끝점 반원 cap (4분할 베지어) → 오른쪽 곡선 역순 → 시작점 반원 cap → close.

Glyphs/FontForge에서 쓰는 stroke-to-outline 알고리즘과 본질적으로 같은 방식. 결과 outline이 매끄럽고, 작은 폰트 크기에서도 자연스럽게 렌더링됨.

**남은 한계**:
- 매우 좁은 코너(180도 가까이 꺾임)에서 self-intersection 가능
- 두 stroke가 교차할 때 boolean union 미수행 (시각적으로는 OK이나 path가 겹침)
- 진짜 정밀한 구현은 paper.js 또는 bezier-js의 `Bezier.offset()` 통합 필요 (50~200KB 추가)

함수 목록은 10장 코드 구조 가이드 참조.

### 5.5 자동 저장 (디바운스)

- `scheduleAutoSave()`: 1.5초 디바운스
- 그리기/베지어 편집/위치 조정 등 모든 변경에서 호출
- 저장 중 표시 → 완료 시 "자동 저장됨"
- `beforeunload`에서 저장 중이면 경고

### 5.6 호환성 체크 (`checkCompatibility`)

페이지 로드 시 다음을 검사하고 부족한 경우 상단 노란 배너 표시:
- IndexedDB 지원
- opentype.js 로드 성공
- Blob API 지원
- `navigator.deviceMemory < 2` 시 메모리 경고

---

## 6. 사용자 흐름 (User Flow)

### 6.1 첫 사용 흐름 (5분 안에 결과물)

1. 파일 더블클릭 → 브라우저에서 열림
2. 우상단 **샘플** 버튼 → 미리 그려둔 자소 30개 로드
3. **[03 조합표]** → 11,172자 중 일부가 이미 채워진 것 확인
4. **[04 본문]** → 샘플 문장으로 가독성 확인
5. **[07 내보내기]** → TTF 다운로드 → OS에 설치

### 6.2 정식 폰트 제작 흐름

페이지 번호와 매칭하여 진행:

1. **[01 그리기]** 한 자소부터 시작 (보통 ㄱ V_NJ)
2. 손글씨 사진을 **참조 이미지**로 업로드 → 따라 그리기
3. 같은 글자의 5벌(또는 4벌)을 모두 작업 (V_NJ → V_J → H_NJ → H_J → C)
4. 다음 자소로 이동, 11벌 모두 채울 때까지 반복
5. **[02 위치]** 6개 케이스 미세조정
6. **[03 조합표]** 어색한 글자 발견 → 다시 [01 그리기]로 돌아가 수정
7. **[04 본문]** 실제 문장 가독성 점검
8. **[05 커닝]** 어색한 글자쌍 페어 추가 → 슬라이더로 간격 조정 (선택사항)
9. **[06 정보]** 메타데이터 작성
10. **[07 내보내기]** TTF 다운로드

### 6.3 백업/복원

- **프로젝트 백업**: [07 내보내기] → JSON 파일로 다운로드
- **다른 기기로 이동**: 그 기기에서 같은 HTML 파일을 열고 JSON 복원

---

## 7. 디자인 시스템

### 7.1 컨셉

일본 활자 작업실 분위기 — 종이 질감, 검정 잉크, 주황(vermilion) 강조.

### 7.2 색상 토큰

```css
--ink:           #1a1410   /* 기본 텍스트 / 선 */
--paper:         #f4ede1   /* 배경 */
--paper-deep:    #ebe2d2   /* 배경 강조 */
--paper-soft:    #efe7d6   /* 배경 부드러움 */
--vermilion:     #c2412e   /* 주황 강조, 활성 상태 */
--vermilion-soft: #d97a6a
--indigo:        #2c3e60   /* 보조 강조 (드물게) */
--rule:          #d4c8b0   /* 구분선 */
--rule-soft:     #e2d8c2
--muted:         #8a7e68   /* 보조 텍스트 */
```

### 7.3 타이포그래피

```css
--serif:   'Noto Serif KR', 'Gowun Batang', serif    /* 본문 */
--display: 'Gowun Batang', serif                      /* 제목, 자소 */
--mono:    'JetBrains Mono', monospace                /* UI 라벨, 좌표값 */
```

### 7.4 미디어 쿼리 브레이크포인트

- 1100px 이하: 우측 패널 숨김 (2단 레이아웃)
- 768px 이하: 1단 세로 레이아웃 (모바일/태블릿 세로)
- `pointer: coarse`: 터치 환경 — 슬라이더 thumb·버튼 패딩 확대

---

## 8. 현재 한계 및 알려진 이슈

### 8.1 폰트 출력 품질

- **outline 알고리즘**: stroke offset 방식으로 매끄러운 outline 생성. 매우 좁은 코너(180도 가까이 꺾임)에서 self-intersection 가능. 두 stroke가 교차할 때 boolean union 미수행 (시각적으로는 OK이나 path가 겹침). 진짜 정밀한 구현은 paper.js/bezier-js 통합 필요.
- **출력 형식**: TTF만 지원. OTF/WOFF/WOFF2는 별도 변환기 사용 필요 (opentype.js 한계).
- **커닝**: 사용자 정의 페어는 GPOS 커닝 테이블로 정상 출력됨. 자동 커닝(글자쌍 자동 분석으로 보정값 제안)은 미구현.
- **힌팅**: 미구현. 현대 OS·브라우저는 그레이스케일·서브픽셀 렌더링·고해상도 디스플레이를 지원해서 힌팅 없이도 깔끔하게 보임. 정밀한 자동 힌팅은 ttfautohint(C 도구) 수준의 별도 작업 필요.
- **NFD 자모 합자**: 진짜 GSUB 합자가 아니라, 호환자모(U+3131~) 영역에 자소 글리프를 단순 매핑.

### 8.2 입력 한계

- **펜 압력 미감지**: Apple Pencil 사용 시에도 모든 선이 동일 두께
- **핀치 줌 미지원**: 캔버스 확대/축소 불가
- **벡터 import 미지원**: SVG 파일을 자소로 import 불가 (참조 이미지로만 사용 가능)

### 8.3 데이터 한계

- **단일 프로젝트**: 동시에 하나의 폰트 패밀리만 작업 가능
- **단일 스타일**: Regular/Bold 등을 동시에 다룰 수 없음
- **참조 이미지 용량**: 5MB 초과 PNG는 IndexedDB 저장 실패 가능

### 8.4 협업

- **로컬 전용**: 서버 없음, 다른 사람과 동시 작업 불가
- **브라우저 격리**: 같은 파일이라도 Chrome에서 만든 폰트가 Firefox 같은 파일에서는 안 보임 (각 브라우저가 별도 IndexedDB)

### 8.5 알려진 버그/미세한 이슈

- 위치 조정 페이지에서 자소 클릭 영역(바운딩박스)이 자소가 매우 얇을 때 잡기 어려움
- 그리드 페이지에서 2,000셀 초과 시 자동 잘림 → 검색·필터 사용 안내만 표시
- 베지어 편집 모드에서 자유 그리기와 베지어가 섞여 있을 때, 자유 그리기는 핸들이 안 보임 (벡터화 후 편집해야 함)

---

## 9. 향후 발전 방향

### 9.1 단기 개선 (소규모, 단일 PR로 가능)

| 항목 | 난이도 | 가치 |
|------|--------|------|
| 캔버스 핀치 줌/패닝 | 중 | 태블릿 작업성 향상 |
| 펜 압력 감지 (Pointer Events `pressure`) | 중 | 손글씨 폰트 자연스러움 |
| 자소 복사/붙여넣기 (다른 자소를 베이스로) | 하 | 작업 속도 향상 |
| Ctrl+클릭 다중 선택 (베지어 핸들) | 중 | 편집 효율 |
| SVG export (현재 자소 1개) | 하 | Illustrator 워크플로 연계 |
| 그리드 페이지 가상 스크롤 | 중 | 11,172자 모두 표시 |
| 다크 모드 | 하 | 야간 작업 |
| 다국어 UI (영어) | 중 | 해외 사용자 |
| 커닝 페어 일괄 import (CSV/TSV) | 하 | 기존 폰트 커닝 표 재사용 |

### 9.2 중기 발전 (별도 모듈 추가)

- **진정한 벡터화**: Potrace.js 통합 → PNG 손글씨를 자동으로 stroke가 아닌 outline으로 변환
- **outline 알고리즘 정밀화**: paper.js 또는 bezier-js의 `Bezier.offset()` 통합 → self-intersection·stroke 교차 boolean union 완전 처리
- **GSUB 합자**: NFD 자모 입력 시 진짜 합자 (예: ᄀ+ᅡ → 가)
- **자동 커닝**: 글자쌍 시각적 간격 자동 분석 → 커닝 페이지에 추천값 제안
- **다중 스타일 관리**: Light/Regular/Bold를 한 프로젝트에서
- **참조 이미지 갤러리**: 자소별 다중 참조 이미지
- **자소 일관성 검사**: ㄱ의 시작점이 ㄲ과 다르면 경고

### 9.3 장기 발전 (구조 변경 필요)

- **백엔드 추가**: 클라우드 저장, 계정, 협업
- **OTF 출력**: opentype.js를 CFF 지원 버전으로 교체하거나 별도 라이브러리 추가
- **WOFF/WOFF2 출력**: WASM 기반 변환기 추가 (TTF→WOFF2)
- **자동 힌팅**: ttfautohint를 WASM으로 포팅한 라이브러리 통합 (파일 크기 1MB+ 증가)
- **옛한글 지원**: 첫가끝(NFD) 입력 + GSUB 옛 자모 합자
- **다중 언어 글리프**: 일본어 가나, 한자 일부, 키릴 등
- **벡터 import**: SVG/AI 파일을 자소로 직접 import (Glyphs 워크플로)
- **16벌식/24벌식 확장**: 분류 함수 교체로 가능하나 기존 자소와의 마이그레이션 UI 필요

### 9.4 코드 구조 개선 (리팩토링)

현재는 단일 HTML 파일 안에 모든 JS가 인라인. 코드가 더 커지면 다음을 고려:

- ES Modules로 분리 (개발 시 분리, 배포 시 번들링)
- TypeScript 도입 (자소 데이터 구조가 복잡해지면 유용)
- Web Components로 캔버스/조합표 컴포넌트화
- 상태 관리 라이브러리 도입 (현재는 전역 변수 + DOM 직접 조작)
- 단위 테스트 (특히 음절 합성·좌표 변환 로직)

단, **단일 HTML 배포 가능성을 잃지 않는 것이 중요**. 사용자가 다운로드해서 더블클릭만으로 쓸 수 있어야 한다는 원칙.

---

## 10. 코드 구조 가이드 (수정 시 참조)

현재 `jaso-hangul-font-maker.html` 단일 파일 안에 모든 코드가 들어 있음. JS 부분은 다음 섹션으로 주석 구분되어 있음:

| 섹션 | 역할 | 주요 함수/변수 |
|------|------|---------------|
| 0 | 한글 자모 데이터 상수 | CHO, JUNG, JONG, *_SETS, getChoSet 등 |
| 1 | KS X 1001 1001자 + 빈도 데이터 | KS1001_TEXT, KS1001_SET |
| 2 | IndexedDB 래퍼 | openDB, dbGet/Put/Delete/All/Clear |
| 3 | 인메모리 저장소 | jasoStore, arrangeStore, latinStore, metaStore, kerningStore |
| 4 | 기본 위치 변환 테이블 | DEFAULT_TRANSFORMS, getTransform |
| 5 | UI 상태 변수 | currentSelection, currentPage, currentMode |
| 6 | 캔버스 자유 그리기 | strokes, history, redrawCanvas, onDrawStart 등 |
| 7 | 베지어 편집 모드 | drawBezierHandles, startBezierDrag |
| 8 | 벡터화 (점→베지어) | vectorizeStrokes, pointsToBezier, simplifyPoints |
| 9 | 자소 저장/로드 | saveCurrentJaso, loadJaso, strokesToEm/FromEm |
| 10 | 자소 셀 UI 빌더 | buildJasoSets, buildLatinSet, updateJasoCellStates |
| 11 | 음절 합성 (SVG) | transformPath, composeSyllableSVG |
| 12 | 미리보기 (디바운스) | schedulePreviewUpdate, updateLivePreview |
| 13 | 자동 저장 | scheduleAutoSave, markUnsaved/Saved |
| 14 | 페이지 전환 | switchPage |
| 15 | 위치 조정 페이지 | initArrangePage, drawArrangeCanvas, arrangeDragMove 등 |
| 16 | 조합표 페이지 | refreshGridPage, canCompose, countJasoByType |
| 17 | 본문 시뮬레이터 | refreshTypesetPage, renderTypeset, SAMPLE_TEXTS |
| 17-A | 커닝 페이지 | refreshKerningPage, addKernPair, setupKerningControls, COMMON_KERN_PAIRS, pairKey |
| 18 | 메타데이터 | loadMeta, applyLicenseTemplate, setupMetaControls |
| 19 | 내보내기 (TTF + outline 알고리즘) | exportFont, strokesToOpenTypePath, addStrokeOutline, drawSmoothPath, arcCap, sampleBezier, smoothSamples, addCircle, removeNearbyDuplicates, composeSyllableOTPath |
| 20 | 프로젝트 백업/복원 | exportProject, importProject (kerning 포함) |
| 21 | 토스트/모달/호환성 | showToast, showModal, checkCompatibility |
| 22 | 샘플 데이터 | loadSampleData, generateSampleJaso |
| 23 | 키보드 단축키 | setupKeyboard |
| 23-A | 참조 이미지 | refImg, setupRefImage, applyRefImg, drawRefImgHandles, refImgDragMove, save/loadRefImgPerJaso |
| 24 | 초기화 | loadFromDB, init |

**수정 시 주의**:

- 데이터 구조 변경 시 다음 5곳을 모두 같이 수정 — `loadFromDB`, `loadRefImgForCurrent`, `exportProject`, `importProject`, reset 핸들러
- 새 페이지 추가 시 다음 4곳을 모두 추가 — `switchPage()`의 분기, nav-tab DOM, page section DOM, `init()`의 setup 호출
- 새 자소 카테고리 추가 시 `composeSyllableSVG`와 `composeSyllableOTPath` 모두 수정 필요 (SVG 미리보기와 폰트 출력이 별개)
- 좌표 변환 시 캔버스(500)/EM(1000) 비율 주의
- DB 스키마 변경 시 `DB_VERSION` 증가 + `onupgradeneeded`에 store 추가 + 모든 dbGetAll/Put/Delete 호출에 try-catch (구버전 사용자 보호)
- outline 알고리즘 수정 시 self-intersection 처리에 특히 주의 — 좁은 코너에서 path가 꼬일 수 있음

---

## 11. 라이선스 및 사용 조건

- **본 도구로 만든 폰트**의 라이선스는 사용자가 정함 ([06 정보] 페이지에서 설정)
- **외부 의존성** 라이선스 준수 필요:
  - opentype.js — MIT
  - Google Fonts (Gowun Batang, Noto Serif KR, JetBrains Mono) — OFL

---

## 부록 A: 11벌식 케이스별 기본 변환값

`DEFAULT_TRANSFORMS` (EM 좌표):

| 자소 | set | x | y | sx | sy | 비고 |
|------|-----|---|---|-----|-----|------|
| cho | V_NJ | 0 | 0 | 0.95 | 0.95 | 가 (받침없음) |
| cho | V_J | 0 | -50 | 0.85 | 0.70 | 각 (받침있음, 위로 압축) |
| cho | H_NJ | 75 | -10 | 0.85 | 0.55 | 고 (가로형 위에) |
| cho | H_J | 100 | -30 | 0.80 | 0.45 | 곡 |
| cho | C | 75 | -10 | 0.85 | 0.55 | 과 |
| jung | NJ | 0 | 0 | 1.0 | 1.0 | 기본 |
| jung | J | 0 | 0 | 1.0 | 0.85 | 받침있음 (위로 압축) |
| jong | V | 0 | 600 | 1.0 | 0.40 | 각 받침 |
| jong | H | 0 | 600 | 1.0 | 0.40 | 곡 받침 |
| jong | C | 0 | 600 | 1.0 | 0.40 | 광 받침 |
| jong | S | 50 | 650 | 0.9 | 0.35 | 간략형 (예약) |

이 값들은 사용자가 [02 위치] 페이지에서 케이스별로 오버라이드 가능.

---

## 부록 B: 자주 묻는 질문 (FAQ)

**Q. 데이터는 어디 저장되나요?**
A. 브라우저의 IndexedDB(로컬). 다른 기기로 옮기려면 [07 내보내기] 페이지에서 프로젝트 백업(JSON) 사용.

**Q. 같은 HTML 파일을 다른 컴퓨터에서 열면 작업물이 보이나요?**
A. 안 보입니다. IndexedDB는 기기·브라우저 단위로 격리됨. JSON 백업·복원으로 옮겨야 함.

**Q. 만든 폰트를 상업적으로 써도 되나요?**
A. 사용자가 [06 정보] 페이지에서 설정한 라이선스에 따름. 본 도구는 폰트 생성만 지원하고, 라이선스 결정은 사용자 책임.

**Q. 11,172자를 다 그려야 하나요?**
A. 아니요. 자소 단위로 그리면 자동 조합됨. 19+21+27 = 67개 자모를 11벌식으로 만들면 약 200~300개 자소만 그려도 11,172자 모두 합성됨.

**Q. 손글씨가 아닌 디지털 폰트도 만들 수 있나요?**
A. 가능하나 베지어 편집 도구가 Glyphs 수준이 아니라서 정밀한 디지털 디자인은 한계가 있음. 그런 작업은 Glyphs/FontLab 권장.

**Q. 커닝은 꼭 해야 하나요?**
A. 안 해도 됩니다. 한글은 모든 음절이 정사각 박스에 들어가도록 설계되어 글자간 간격이 거의 일정합니다. 영문/한영 경계나 받침+모음 시작(예: "을 아") 같은 특수 케이스에서만 시각적으로 도움이 됩니다. 본문체의 완성도를 더 끌어올리고 싶을 때만 사용하세요.

**Q. 커닝 단위 EM/1000은 무슨 뜻인가요?**
A. 폰트의 한 글자 너비(EM = 1000)에 대한 보정 비율입니다. -50이면 글자가 5% 정도 가까워지고, +50이면 5% 멀어집니다. 일반적으로 ±50 이내가 자연스럽고, ±100 넘어가면 부자연스럽게 느껴질 수 있습니다.

**Q. PNG 참조 이미지가 너무 많아지면 어떻게 되나요?**
A. 자소 하나당 이미지 하나가 IndexedDB에 저장됩니다. 한 자소당 1MB 이하면 245개 자소 모두 채워도 250MB 이내라 대부분의 브라우저가 감당합니다. 5MB 초과 이미지는 경고가 뜨며 저장 실패할 수 있습니다.
