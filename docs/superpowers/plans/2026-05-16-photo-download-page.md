# 가정의 달 사진 다운로드 페이지 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** 연천장로교회 성도들이 QR 코드를 스캔하면 바로 가정의 달 사진을 미리보고 다운로드할 수 있는 단일 HTML 페이지를 만들어 GitHub Pages에 배포한다.

**Architecture:** 단일 `index.html` 파일에 HTML/CSS/JS를 모두 담는다. 페이지 로드 시 Google Drive API v3를 호출해 공개 폴더의 사진 목록을 가져오고, 썸네일 그리드를 렌더링한다. 배포는 GitHub Pages (max2guy 계정)로 한다.

**Tech Stack:** Vanilla HTML5, CSS3, JavaScript (ES6+), Google Drive API v3

---

## 파일 구조

```
drive-download-page/
├── index.html          # 전체 앱 (HTML + CSS + JS 단일 파일)
└── docs/
    └── superpowers/
        ├── specs/2026-05-16-photo-download-page-design.md
        └── plans/2026-05-16-photo-download-page.md  ← 이 파일
```

---

## 사전 준비: Google Cloud API 키 발급

이 작업은 브라우저에서 직접 진행한다 (Claude가 대신 할 수 없음).

- [ ] **Step 1: Google Cloud Console 접속**

  https://console.cloud.google.com 로 이동 → 구글 계정 로그인

- [ ] **Step 2: 새 프로젝트 생성**

  상단 프로젝트 선택 → "새 프로젝트" → 이름: `yc-photo-2025` → 만들기

- [ ] **Step 3: Drive API 활성화**

  좌측 메뉴 → "API 및 서비스" → "라이브러리" → "Google Drive API" 검색 → "사용" 클릭

- [ ] **Step 4: API 키 발급**

  "API 및 서비스" → "사용자 인증 정보" → "+ 사용자 인증 정보 만들기" → "API 키"
  → 생성된 키를 복사해 둠

- [ ] **Step 5: API 키 제한 설정 (보안)**

  생성된 API 키 클릭 → "키 제한" →
  - 애플리케이션 제한: "HTTP 리퍼러(웹사이트)" 선택
  - 웹사이트 추가: `https://max2guy.github.io/*`
  - API 제한: "Google Drive API" 선택
  → 저장

- [ ] **Step 6: Drive 폴더 공개 공유 확인**

  https://drive.google.com/drive/folders/1FPFWclBSGZVzzcfurSwe-VaGmm-GHaPp 접속
  → 링크가 있는 모든 사용자에게 공개되어 있는지 확인
  (공개가 아니면 폴더 소유자에게 "링크가 있는 모든 사용자 - 뷰어" 설정 요청)

---

## Task 1: 프로젝트 초기화 및 Git 저장소 생성

**Files:**
- Create: `index.html`

- [ ] **Step 1: 프로젝트 디렉토리로 이동 및 git 초기화**

```bash
cd ~/drive-download-page
git init
```

- [ ] **Step 2: .gitignore 생성**

```bash
cat > .gitignore << 'EOF'
.DS_Store
.superpowers/
EOF
```

- [ ] **Step 3: index.html 뼈대 생성 (API 키 자리 포함)**

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>2025 가정의 달 사진 — 연천장로교회</title>
</head>
<body>
  <p>준비 중...</p>
  <script>
    const FOLDER_ID = '1FPFWclBSGZVzzcfurSwe-VaGmm-GHaPp';
    const API_KEY = 'YOUR_API_KEY_HERE'; // 사전 준비 단계에서 발급한 키로 교체
  </script>
</body>
</html>
```

- [ ] **Step 4: 브라우저에서 파일 열기 확인**

```bash
open index.html
```

  브라우저에 "준비 중..." 텍스트가 보이면 정상.

- [ ] **Step 5: 초기 커밋**

```bash
git add index.html .gitignore
git commit -m "init: project scaffold"
```

---

## Task 2: HTML 구조 및 CSS 디자인

**Files:**
- Modify: `index.html` (HTML 마크업 + `<style>` 블록 추가)

- [ ] **Step 1: `<head>` 내용 교체 (메타 + 스타일)**

  `index.html`의 `<head>` 전체를 아래로 교체:

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>2025 가정의 달 사진 — 연천장로교회</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: 'Apple SD Gothic Neo', 'Noto Sans KR', sans-serif;
      background: #FAF6F1;
      color: #3D2B1F;
      min-height: 100vh;
    }

    /* ── 헤더 ── */
    .site-header {
      background: linear-gradient(160deg, #F5EDE0 0%, #EDD9C0 100%);
      border-bottom: 1px solid #D4B896;
      padding: 32px 20px 28px;
      text-align: center;
    }
    .eyebrow {
      font-size: 11px;
      letter-spacing: 5px;
      color: #B07D4A;
      text-transform: uppercase;
      margin-bottom: 10px;
    }
    .site-header h1 {
      font-family: 'Georgia', serif;
      font-size: 28px;
      font-weight: 400;
      color: #3D2B1F;
      letter-spacing: 1px;
      margin-bottom: 6px;
    }
    .header-sub {
      font-size: 14px;
      color: #7A5C3C;
      letter-spacing: 1px;
    }
    .divider {
      width: 50px;
      height: 2px;
      background: linear-gradient(90deg, transparent, #B07D4A, transparent);
      margin: 14px auto;
    }

    /* ── 전체 다운로드 버튼 ── */
    .cta-wrap {
      padding: 28px 20px 16px;
      text-align: center;
    }
    .btn-all {
      display: inline-flex;
      align-items: center;
      gap: 10px;
      background: #8B5E3C;
      color: #FAF6F1;
      font-size: 17px;
      font-weight: 600;
      padding: 16px 36px;
      border-radius: 4px;
      border: none;
      letter-spacing: 1px;
      box-shadow: 0 4px 16px rgba(139,94,60,0.3);
      cursor: pointer;
      text-decoration: none;
    }
    .btn-all:active { opacity: 0.85; }
    .cta-hint {
      font-size: 13px;
      color: #9C7A5A;
      margin-top: 10px;
      letter-spacing: 0.5px;
    }

    /* ── 상태 메시지 ── */
    .status {
      text-align: center;
      padding: 16px 20px;
      font-size: 14px;
      color: #B07D4A;
      letter-spacing: 1px;
      min-height: 48px;
    }
    .status.error { color: #C0392B; }

    /* ── 섹션 레이블 ── */
    .section-label {
      font-size: 11px;
      letter-spacing: 4px;
      color: #B07D4A;
      text-transform: uppercase;
      text-align: center;
      padding: 8px 20px 16px;
    }

    /* ── 사진 그리드 ── */
    .photo-grid {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 12px;
      padding: 0 16px 40px;
      max-width: 520px;
      margin: 0 auto;
    }
    .photo-card {
      background: #FFF8F0;
      border: 1px solid #E8D5BF;
      border-radius: 8px;
      overflow: hidden;
      box-shadow: 0 2px 8px rgba(0,0,0,0.06);
    }
    .photo-thumb {
      width: 100%;
      aspect-ratio: 1;
      object-fit: cover;
      display: block;
      background: #EDD9C0;
    }
    .photo-info {
      padding: 10px 12px 12px;
    }
    .photo-name {
      font-size: 12px;
      color: #5C3D1E;
      margin-bottom: 8px;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }
    .btn-dl {
      display: flex;
      align-items: center;
      justify-content: center;
      gap: 6px;
      width: 100%;
      background: #F5EDE0;
      border: 1px solid #C9A87C;
      color: #7A5C3C;
      font-size: 14px;
      font-weight: 600;
      padding: 10px 0;
      border-radius: 4px;
      cursor: pointer;
      text-decoration: none;
    }
    .btn-dl:active { background: #EDD9C0; }

    /* ── 푸터 ── */
    .site-footer {
      text-align: center;
      padding: 20px;
      font-size: 11px;
      color: #C9A87C;
      border-top: 1px solid #E8D5BF;
      letter-spacing: 2px;
    }
  </style>
</head>
```

- [ ] **Step 2: `<body>` 마크업 교체**

  `<body>` 전체를 아래로 교체 (`<script>` 블록은 맨 아래에 유지):

```html
<body>

  <header class="site-header">
    <div class="eyebrow">2025 · May</div>
    <h1>가정의 달 사진</h1>
    <div class="divider"></div>
    <div class="header-sub">소중한 순간을 간직하세요</div>
  </header>

  <div class="cta-wrap">
    <a id="btn-all" class="btn-all" href="#" target="_blank" rel="noopener">
      ⬇ 전체 사진 다운로드
    </a>
    <div class="cta-hint">버튼을 누르면 모든 사진이 저장됩니다</div>
  </div>

  <div id="status" class="status">사진을 불러오는 중...</div>

  <div class="section-label" id="grid-label" style="display:none">개별 사진 선택</div>
  <div class="photo-grid" id="photo-grid"></div>

  <footer class="site-footer">연천장로교회 · 2025 가정의 달</footer>

  <script>
    const FOLDER_ID = '1FPFWclBSGZVzzcfurSwe-VaGmm-GHaPp';
    const API_KEY = 'YOUR_API_KEY_HERE';
  </script>
</body>
```

- [ ] **Step 3: 브라우저 새로고침으로 디자인 확인**

  - 크림·브라운 헤더, 전체 다운로드 버튼, "사진을 불러오는 중..." 상태 메시지가 보이면 정상.
  - 모바일 크기로 창을 줄여서 레이아웃 확인.

- [ ] **Step 4: 커밋**

```bash
git add index.html
git commit -m "feat: add HTML structure and cream-brown CSS theme"
```

---

## Task 3: Google Drive API 연동 — 파일 목록 로딩

**Files:**
- Modify: `index.html` (JS 추가)

> **사전 조건:** 사전 준비 단계에서 발급한 API 키를 `API_KEY` 상수에 교체한 후 진행.

- [ ] **Step 1: `API_KEY` 상수에 실제 키 입력**

  `index.html`에서 아래 줄을 찾아 실제 API 키로 교체:
  ```js
  const API_KEY = 'YOUR_API_KEY_HERE';
  // ↓
  const API_KEY = 'AIzaSy...실제키...';
  ```

- [ ] **Step 2: Drive API 호출 함수 추가**

  `</script>` 바로 위에 아래 JS를 추가:

```js
async function loadPhotos() {
  const statusEl = document.getElementById('status');
  const gridEl = document.getElementById('photo-grid');
  const gridLabel = document.getElementById('grid-label');

  try {
    let files = [];
    let pageToken = null;

    // 페이지 단위로 전체 파일 목록 수집 (최대 1000장)
    do {
      const params = new URLSearchParams({
        q: `'${FOLDER_ID}' in parents and mimeType contains 'image/' and trashed = false`,
        fields: 'nextPageToken, files(id, name)',
        pageSize: 100,
        key: API_KEY,
        ...(pageToken ? { pageToken } : {})
      });
      const res = await fetch(`https://www.googleapis.com/drive/v3/files?${params}`);
      if (!res.ok) throw new Error(`API 오류: ${res.status}`);
      const data = await res.json();
      files = files.concat(data.files || []);
      pageToken = data.nextPageToken;
    } while (pageToken);

    if (files.length === 0) {
      statusEl.textContent = '사진이 없습니다.';
      return;
    }

    statusEl.textContent = '';
    gridLabel.style.display = 'block';
    renderPhotos(files, gridEl);

  } catch (err) {
    console.error(err);
    statusEl.textContent = '사진을 불러오지 못했습니다. 잠시 후 다시 시도해 주세요.';
    statusEl.classList.add('error');
  }
}

function renderPhotos(files, gridEl) {
  gridEl.innerHTML = '';
  files.forEach(file => {
    const thumbUrl = `https://drive.google.com/thumbnail?id=${file.id}&sz=w400`;
    const downloadUrl = `https://drive.google.com/uc?export=download&id=${file.id}`;

    const card = document.createElement('div');
    card.className = 'photo-card';
    card.innerHTML = `
      <img class="photo-thumb" src="${thumbUrl}" alt="${file.name}" loading="lazy">
      <div class="photo-info">
        <div class="photo-name">${file.name}</div>
        <a class="btn-dl" href="${downloadUrl}" download="${file.name}" target="_blank" rel="noopener">
          ⬇ 받기
        </a>
      </div>
    `;
    gridEl.appendChild(card);
  });
}

// 전체 다운로드 버튼: Drive 폴더 ZIP 링크
document.getElementById('btn-all').href =
  `https://drive.google.com/drive/folders/${FOLDER_ID}`;

loadPhotos();
```

- [ ] **Step 3: 브라우저에서 동작 확인**

  ```bash
  open index.html
  ```

  확인 항목:
  - 사진 썸네일이 그리드에 나타나는지
  - "개별 사진 선택" 레이블이 나타나는지
  - 각 카드의 "⬇ 받기" 버튼이 실제 파일로 연결되는지 (클릭해서 다운로드 시도)
  - "전체 사진 다운로드" 버튼이 Drive 폴더로 이동하는지

  > **API 키 제한 때문에 로컬에서 오류가 날 경우:** API 키 제한에서 `localhost`를 임시로 허용하거나, 제한 없이 테스트 후 배포 전에 다시 제한 설정.

- [ ] **Step 4: 커밋**

```bash
git add index.html
git commit -m "feat: fetch and render photos from Google Drive API"
```

---

## Task 4: 에러 처리 및 UX 개선

**Files:**
- Modify: `index.html` (이미지 로드 실패 처리 + 로딩 인디케이터 개선)

- [ ] **Step 1: 이미지 로드 실패 처리 추가**

  `renderPhotos` 함수의 `card.innerHTML` 안에서 `<img>` 태그를 아래로 교체:

```html
<img class="photo-thumb"
     src="${thumbUrl}"
     alt="${file.name}"
     loading="lazy"
     onerror="this.style.background='#EDD9C0';this.removeAttribute('src')">
```

- [ ] **Step 2: 로딩 중 점 애니메이션 추가**

  `<style>` 블록 안에 추가:

```css
@keyframes blink { 0%,80%,100%{opacity:0.2} 40%{opacity:1} }
.dot {
  display: inline-block;
  width: 7px; height: 7px;
  background: #C9A87C;
  border-radius: 50%;
  margin: 0 2px;
  animation: blink 1.2s ease-in-out infinite;
}
.dot:nth-child(2) { animation-delay: 0.2s; }
.dot:nth-child(3) { animation-delay: 0.4s; }
```

  `<div id="status">` 내용을 교체:

```html
<div id="status" class="status">
  <span class="dot"></span><span class="dot"></span><span class="dot"></span>
  &nbsp;사진을 불러오는 중...
</div>
```

- [ ] **Step 3: 브라우저에서 에러 케이스 확인**

  API 키를 일부러 잘못된 값으로 바꿔 테스트:
  - 빨간 에러 메시지 "사진을 불러오지 못했습니다. 잠시 후 다시 시도해 주세요." 표시 확인
  - 이후 올바른 키로 복원

- [ ] **Step 4: 커밋**

```bash
git add index.html
git commit -m "feat: add loading animation and image error handling"
```

---

## Task 5: GitHub Pages 배포

**Files:**
- GitHub 저장소 생성 (브라우저 작업)
- `index.html` 푸시

- [ ] **Step 1: GitHub에서 새 저장소 생성**

  https://github.com/new 접속:
  - Repository name: `yc-photo-2025`
  - Visibility: **Public**
  - Initialize: 체크 해제 (이미 로컬에 파일 있음)
  → "Create repository" 클릭

- [ ] **Step 2: 원격 저장소 연결 및 푸시**

```bash
git remote add origin https://github.com/max2guy/yc-photo-2025.git
git branch -M main
git push -u origin main
```

- [ ] **Step 3: GitHub Pages 활성화**

  저장소 → Settings → Pages →
  - Source: "Deploy from a branch"
  - Branch: `main` / `/ (root)`
  → Save

  약 1분 후 배포 URL 확인: `https://max2guy.github.io/yc-photo-2025/`

- [ ] **Step 4: API 키 제한 도메인 업데이트 확인**

  Google Cloud Console → API 키 설정에서
  `https://max2guy.github.io/*` 가 허용 도메인에 있는지 재확인

- [ ] **Step 5: 배포된 URL에서 전체 동작 테스트**

  `https://max2guy.github.io/yc-photo-2025/` 접속:
  - [ ] 사진 그리드 정상 로딩
  - [ ] "⬇ 받기" 버튼 클릭 → 다운로드 시작
  - [ ] "⬇ 전체 사진 다운로드" 버튼 → Drive 폴더 이동
  - [ ] 모바일(또는 개발자도구 모바일 에뮬레이션)에서 레이아웃 확인
  - [ ] 어르신이 쓰기 쉬운지 버튼 크기/간격 점검

- [ ] **Step 6: QR 코드 URL 업데이트**

  기존 QR 코드가 다른 URL을 가리키고 있다면,
  `https://max2guy.github.io/yc-photo-2025/` 로 새 QR 코드 생성:
  - 무료 QR 생성: https://qr.io 또는 https://www.qr-code-generator.com

---

## 완료 체크리스트

- [ ] 사진이 그리드에 자동으로 로딩됨
- [ ] 개별 "⬇ 받기" 버튼으로 사진 다운로드 가능
- [ ] 전체 다운로드 버튼이 Drive 폴더로 연결됨
- [ ] 로딩 중 애니메이션 표시됨
- [ ] API 오류 시 한국어 안내 메시지 표시됨
- [ ] 모바일에서 2열 그리드 정상 표시됨
- [ ] 푸터에 "연천장로교회 · 2025 가정의 달" 표시됨
- [ ] GitHub Pages URL로 QR 코드 생성 완료
