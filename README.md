# PocketMuse GitHub Pages

PocketMuse Android 앱의 Civitai OAuth 로그인에 필요한 Android App Links 검증 파일과 정적 콜백 안내 페이지를 제공한다. HTML/CSS와 JSON만 사용하며 서버, 빌드 시스템, 외부 스크립트는 없다.

## 파일과 URL

```text
PocketMuse.github.io/
├── .well-known/
│   ├── assetlinks.json
│   └── assetlinks.example.json
├── oauth/
│   └── callback/
│       └── index.html
├── .nojekyll
├── index.html
└── README.md
```

- 소개: https://pocketmuse.github.io/
- OAuth Redirect URI: `https://pocketmuse.github.io/oauth/callback/` (마지막 `/` 포함)
- App Links 검증: `https://pocketmuse.github.io/.well-known/assetlinks.json` (Debug 인증서 설정 완료, 배포 후 활성화)

콜백은 앱으로 연결되지 않았을 때 재시도를 안내하는 정적 fallback이다. Authorization Code, state 또는 URL 쿼리를 읽거나 출력·전송·저장하지 않는다. JavaScript와 외부 리소스 없이 `noindex`, `no-referrer`를 설정한다. 앱이 App Link를 정상 처리하면 브라우저에서 이 안내를 표시할 필요가 없다.

## GitHub Pages 배포

1. PocketMuse 조직의 `pocketmuse.github.io` 저장소에 이 디렉터리의 파일을 배치한다. `.well-known`과 빈 `.nojekyll`도 반드시 포함한다.
2. 사용자 승인 후 변경 파일을 커밋하고 배포 브랜치(예: `main`)로 푸시한다.
3. 저장소 **Settings → Pages → Build and deployment**에서 **Deploy from a branch**, 해당 브랜치와 **/(root)**를 선택해 저장한다.
4. **Enforce HTTPS**를 활성화하고 배포 완료 후 아래 URL을 검증한다.

조직 사이트이므로 URL에 저장소 이름을 추가하지 않는다. `.nojekyll`은 Jekyll 처리를 비활성화해 정적 파일을 그대로 제공하도록 유지한다. 별도의 워크플로 파일은 필요하지 않다.

## Android 인증서 등록 상태

- Application ID: `com.example.pocketmuseprototype`
- 현재 `.well-known/assetlinks.json`에는 **Debug 서명 인증서 하나만 등록**되어 있다. Android Agent가 실제 Debug APK를 `apksigner verify --print-certs`로 검사해 확인한 정보로 전달받았다.
- Release 서명은 아직 설정되지 않았다. **Release 배포 시 실제 Release 인증서의 SHA-256 지문을 추가해야 한다.** Release의 `applicationId`가 다르면 해당 패키지명과 인증서 대응 관계도 Android Agent에게 받는다.
- Play App Signing 사용 시 배포 앱의 **앱 서명 인증서** SHA-256 지문을 받는다 (업로드 인증서와 구별).

인증서 지문은 콜론으로 구분한 32바이트, 즉 `AA:BB:…` 형식의 64자리 16진수 값으로 받는다. 값은 추측하거나 생성하지 않는다. 개인키, keystore, 비밀번호, OAuth 토큰은 요청하거나 저장소에 포함하지 않는다.

## assetlinks.json 갱신

실제 배포 파일은 `.well-known/assetlinks.json`이다. `.well-known/assetlinks.example.json`은 재사용할 템플릿으로 유지한다. 템플릿은 유효한 JSON이지만 자리표시자가 있으므로 App Links 검증에는 사용할 수 없다.

1. 기존 `assetlinks.json`을 갱신한다. 새로 생성해야 할 경우 템플릿을 같은 디렉터리의 `assetlinks.json`으로 복사한다.
2. `package_name`에 전달받은 실제 `applicationId`를 설정한다. 템플릿에서 복사한 경우 자리표시자를 교체한다.
3. `sha256_cert_fingerprints`에 해당 패키지의 실제 지문을 등록한다. Release 지문이 없으면 Debug 지문만 유지한다. 템플릿에서 복사한 경우 모든 자리표시자를 교체하거나 삭제한다. 빈 값이나 자리표시자를 남기지 않는다.
4. 패키지명이 같으면 하나의 지문 배열에 여러 인증서 지문을 넣을 수 있다. Debug/Release의 패키지명이 다르면 최상위 배열에 statement를 각각 추가하고 각 패키지에 맞는 지문을 넣는다.
5. `relation`의 `delegate_permission/common.handle_all_urls`와 `namespace`의 `android_app`은 유지한다. UTF-8 JSON으로 저장하고 모든 값과 JSON 문법을 확인한 뒤 사용자 승인 후 커밋·푸시한다.

## 배포 후 검증

PowerShell에서는 `curl.exe`를 사용한다. 쿼리나 인증 정보를 붙이지 않은 URL만 검사한다.

```powershell
curl.exe --proto '=https' --max-redirs 0 -sS -D - -o NUL https://pocketmuse.github.io/
curl.exe --proto '=https' --max-redirs 0 -sS -D - -o NUL https://pocketmuse.github.io/oauth/callback/
```

두 URL 모두 인증서 오류 없이 HTTP `200`, `Content-Type: text/html`로 응답하는지 확인한다. 콜백 화면의 한국어 안내와 모바일 레이아웃도 확인한다. TLS 확인을 우회하는 `-k` 옵션은 사용하지 않는다.

실제 `assetlinks.json`을 완성하고 배포한 뒤에만 아래를 검사한다.

```powershell
curl.exe --proto '=https' --max-redirs 0 -sS -D - -o NUL https://pocketmuse.github.io/.well-known/assetlinks.json
$response = Invoke-WebRequest -Uri 'https://pocketmuse.github.io/.well-known/assetlinks.json' -MaximumRedirection 0
$response.Content | ConvertFrom-Json
```

HTTPS에서 리다이렉트 없이 HTTP `200`, `Content-Type: application/json`을 반환해야 한다. 응답 본문이 HTML 오류 페이지가 아닌 JSON 배열인지, 실제 패키지명·지문이 전달받은 값과 일치하는지 확인한다. 로컬 문법 확인은 `Get-Content -Raw -Encoding UTF8 .well-known/assetlinks.json | ConvertFrom-Json`으로 할 수 있다. 템플릿은 파일명을 `assetlinks.example.json`으로 바꿔 별도로 검사한다.

로컬 JSON 문법·Digital Asset Links 구조·전달받은 Application ID 및 Debug 지문 일치 여부는 확인했다. 이번 작업에서 배포하지 않았으므로 배포된 HTTPS `200`/JSON 응답과 Android 기기의 App Links 검증·앱 연결 확인은 아직 미완료다. Android 코드·Manifest·PKCE·토큰 관리 구현은 별도 Android 프로젝트의 책임이다.

## 공식 참고 문서

- [Android Digital Asset Links 설정](https://developer.android.com/training/app-links/configure-assetlinks)
- [GitHub Pages 사이트 생성](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site)
- [GitHub Pages 배포 소스 설정](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)
- [Civitai OAuth 개발자 문서](https://github.com/civitai/civitai/blob/main/docs/auth/oauth-developer-docs.md)
