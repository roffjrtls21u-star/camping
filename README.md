# 캠핑 준비 & 기록

캠핑 준비물 체크리스트와 캠핑 기록을 관리하는 웹앱입니다. Firebase(Auth + Firestore)로 데이터를 저장하고, GitHub Pages로 배포합니다.

## 1. Firebase 프로젝트 만들기

1. https://console.firebase.google.com 접속 → **프로젝트 추가**
2. 프로젝트 이름 입력 후 생성 (구글 애널리틱스는 꺼도 무방)
3. 왼쪽 메뉴 **빌드 > Authentication** → **시작하기** → **로그인 방법** 탭 → **Google** 활성화
4. 왼쪽 메뉴 **빌드 > Firestore Database** → **데이터베이스 만들기** → **프로덕션 모드**로 시작 (리전은 asia-northeast3 등 가까운 곳 선택)
5. **프로젝트 개요 옆 톱니바퀴 > 프로젝트 설정** → 아래로 스크롤해서 **내 앱** → **웹 앱 추가**(`</>` 아이콘) → 앱 닉네임 입력 후 등록
6. 화면에 나오는 `firebaseConfig` 객체 값을 복사

## 2. 이 프로젝트에 설정 값 붙여넣기

`firebase-config.js` 파일을 열어서 복사한 값으로 교체하세요.

```js
export const firebaseConfig = {
  apiKey: "...",
  authDomain: "...",
  projectId: "...",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "..."
};
```

이 값들은 브라우저에 그대로 노출되는 공개 값이라 GitHub에 커밋해도 안전합니다. 실제 데이터 보호는 아래 Firestore 보안 규칙과 로그인이 담당합니다.

## 3. Firestore 보안 규칙 설정

Firebase 콘솔 **Firestore Database > 규칙** 탭에서 아래 내용으로 교체 후 **게시**하세요. 로그인한 본인만 자신의 데이터를 읽고 쓸 수 있게 하는 규칙입니다.

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{userId}/{document=**} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
  }
}
```

## 4. GitHub Pages로 배포하기

1. GitHub에서 새 저장소 생성 (예: `camping-app`)
2. 이 폴더(`index.html`, `firebase-config.js`, `README.md`)를 저장소에 업로드
   ```
   git init
   git add .
   git commit -m "캠핑 준비 웹앱 초기 배포"
   git branch -M main
   git remote add origin https://github.com/사용자명/camping-app.git
   git push -u origin main
   ```
3. GitHub 저장소 **Settings > Pages** → **Source**를 `main` 브랜치 / `/ (root)`로 설정 후 저장
4. 잠시 후 `https://사용자명.github.io/camping-app/` 에서 접속 가능

## 5. 배포한 도메인을 Firebase에 허용하기

Google 로그인이 배포된 사이트에서도 동작하도록, Firebase 콘솔 **Authentication > 설정 > 승인된 도메인**에 `사용자명.github.io`를 추가하세요. (localhost는 기본 등록되어 있어 로컬 테스트는 바로 됩니다.)

## 데이터 구조

- `users/{내 uid}/trips/{캠핑id}` — 캠핑 1건당 문서 하나. 제목, 장소, 날짜, 체크리스트(카테고리/음식 타임라인), 기록(날씨·별점·메모)이 모두 들어있습니다.
- `users/{내 uid}/meta/template` — "템플릿으로 저장" 시 저장되는 준비물 템플릿. 새 캠핑을 만들 때 불러와 재사용합니다.

## 로컬에서 미리 보기

Firebase Auth 팝업 로그인은 `file://`로 열면 동작하지 않을 수 있어, 간단한 로컬 서버로 열어보세요.

```
npx serve .
```

또는 Python이 있다면:

```
python3 -m http.server 8000
```

이후 `http://localhost:8000` 접속.
