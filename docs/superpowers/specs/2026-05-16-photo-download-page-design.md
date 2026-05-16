# 가정의 달 사진 다운로드 페이지 — 디자인 스펙

## 개요

교회 성도들이 QR 코드를 스캔하면 접근할 수 있는 공개 사진 다운로드 페이지.
Google Drive에 올라온 가정의 달 사진을 어르신 포함 누구나 쉽게 받을 수 있도록 한다.

---

## 요건

### 사용자
- 교회 성도 (전 연령대, 어르신 포함)
- QR 코드 스캔으로 접근 (모바일 우선)
- 인증 없음, 완전 공개

### 콘텐츠
- 2025 가정의 달 사진 (가족, 개인, 구역, 친구 등)
- Google Drive 공개 폴더: `1FPFWclBSGZVzzcfurSwe-VaGmm-GHaPp`
- 폴더 소유자와 무관하게 공개 공유된 상태

---

## 기술 스택

| 항목 | 선택 |
|------|------|
| 구현 방식 | 정적 HTML + Vanilla JS (단일 파일) |
| 호스팅 | GitHub Pages |
| 데이터 소스 | Google Drive API v3 (API 키 방식) |
| 프레임워크 | 없음 (순수 HTML/CSS/JS) |

---

## 아키텍처

```
브라우저
  └─ index.html (HTML + CSS + JS 단일 파일)
       ├─ 페이지 로드 시 → Google Drive API 호출
       │    GET /drive/v3/files?q='{FOLDER_ID}' in parents&key={API_KEY}
       │    → 파일 목록(ID, 이름) 반환
       ├─ 썸네일 표시
       │    https://drive.google.com/thumbnail?id={FILE_ID}&sz=w400
       ├─ 개별 다운로드
       │    https://drive.google.com/uc?export=download&id={FILE_ID}
       └─ 전체 다운로드
            구글 드라이브 폴더 ZIP 다운로드 링크
            (파일 수/용량 제한 있으면 Drive 폴더로 fallback)
```

---

## 디자인 시스템

### 색상
| 역할 | 값 |
|------|----|
| 배경 | `#FAF6F1` (크림) |
| 헤더 배경 | `#F5EDE0 → #EDD9C0` (그라디언트) |
| 기본 텍스트 | `#3D2B1F` (딥브라운) |
| 서브 텍스트 | `#7A5C3C` |
| 강조 (골드) | `#B07D4A` |
| 주 버튼 배경 | `#8B5E3C` |
| 주 버튼 텍스트 | `#FAF6F1` |
| 보조 버튼 배경 | `#F5EDE0` |
| 테두리 | `#E8D5BF`, `#C9A87C` |

### 타이포그래피
- 제목: Georgia (serif), 28px, weight 400
- 소제목·레이블: letter-spacing 4–5px, uppercase, 11px
- 본문: Apple SD Gothic Neo / Noto Sans KR
- 버튼: 17px (전체), 13px (개별)

---

## 페이지 구조

### 1. 헤더
- eyebrow: `2025 · May`
- 제목: `가정의 달 사진`
- 장식 divider (골드 그라디언트 선)
- 부제: `소중한 순간을 간직하세요`

### 2. 전체 다운로드 버튼 (CTA)
- 크고 명확한 버튼: `⬇ 전체 사진 다운로드`
- 버튼 아래 안내 문구: `버튼을 누르면 모든 사진이 저장됩니다`
- 어르신을 위한 심리적 불안 제거용 설명

### 3. 개별 사진 그리드
- 섹션 레이블: `개별 사진 선택`
- 2열 그리드 (모바일 기준)
- 각 카드: 썸네일 + 파일명 + `⬇ 받기` 버튼
- 로딩 중: 점 3개 애니메이션 + `사진을 불러오는 중...`
- 에러 시: `사진을 불러오지 못했습니다. 다시 시도해 주세요.`

### 4. 푸터
- `연천장로교회 · 2025 가정의 달`

---

## UX 원칙 (어르신 친화)

1. **버튼 크기**: 전체 다운로드 버튼 padding 16px 이상
2. **설명 문구**: 버튼마다 무슨 일이 일어나는지 설명
3. **로딩 피드백**: 사진을 불러오는 동안 명확한 상태 표시
4. **에러 메시지**: 기술 용어 없이 쉬운 한국어
5. **단순 구조**: 스크롤 한 번이면 모든 기능 접근 가능

---

## 구현 단계

1. **Google Cloud 설정**
   - Google Cloud Console에서 프로젝트 생성
   - Google Drive API 활성화
   - API 키 발급 (브라우저 제한: GitHub Pages 도메인만 허용)

2. **HTML 파일 작성** (`index.html`)
   - 헤더 / 버튼 / 그리드 마크업
   - CSS: 크림·브라운 테마
   - JS: Drive API 호출 → 파일 목록 → 썸네일 렌더링
   - 파일 상단 상수: `FOLDER_ID = '1FPFWclBSGZVzzcfurSwe-VaGmm-GHaPp'`, `API_KEY = '...'`

3. **GitHub Pages 배포**
   - GitHub 계정: https://github.com/max2guy
   - 저장소 생성 (public): `max2guy/ycpc-photo-2025` 또는 유사
   - `index.html` 푸시
   - Pages 설정 → 자동 배포
   - 배포 URL: `https://max2guy.github.io/<repo명>/`
   - 생성된 URL로 QR 코드 생성/업데이트

---

## 제약 사항

- Google Drive 폴더 ZIP 다운로드는 파일 수 > 500개 또는 용량 > 2GB이면 불가. 이 경우 Drive 폴더 직접 링크로 fallback.
- API 키는 GitHub Pages 도메인으로 제한하여 남용 방지.
- Drive API 무료 할당량: 하루 1,000,000,000 쿼리 (실질적 제한 없음).
