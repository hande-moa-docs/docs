## 카카오 SSO API 명세서

-----

### 문서 정보

- **문서명**: 카카오 SSO API 명세서
- **버전**: v1.0.1
- **작성일**: 2025.09.11
- **작성자**: 송민재
- **최종 수정일**: 2025.09.11

-----

### 1\. 개요

* **API 버전**: v1.0
* **기본 URL**: `localhost:9000/api`
* **설명**: 카카오톡을 통한 **Single Sign-On (SSO)** 인증 및 회원가입에 대한 명세서입니다. 모든 보안 관련 작업(인가 코드 교환, 토큰 발급)은 백엔드에서만 처리됩니다.

-----

### 2\. 카카오 로그인 리다이렉트

* **API 엔드포인트**: `GET /auth/kakao`
* **설명**: 사용자를 카카오 로그인 페이지로 리다이렉트합니다.
* **프론트엔드 흐름**:
    1.  사용자가 '카카오 로그인' 버튼 클릭.
    2.  `window.location.href = 'localhost:9000/api/v1/auth/kakao';`
    3.  백엔드가 카카오 서버로 리다이렉트.
    4.  로그인 성공 시, 카카오 서버는 **인가 코드**를 백엔드 서버의 콜백 URL로 전달.
* **요청/응답**: 별도의 명세는 없으며, 백엔드 내부 로직으로 처리됩니다.

-----

### 3\. 인가 코드 콜백 및 JWT 발급

* **API 엔드포인트**: `GET /auth/kakao/callback`
* **설명**: 카카오로부터 인가 코드를 받아 **JWT**를 발급하는 백엔드 콜백 엔드포인트입니다.
* **프론트엔드 흐름**:
    1.  카카오 로그인 후 백엔드 콜백 URL로 자동 리다이렉트됩니다.
    2.  이때, 백엔드가 프론트엔드 URL에 JWT 토큰을 포함하여 리다이렉트합니다.
    3.  프론트엔드는 URL 쿼리 파라미터에서 토큰을 추출하여 로컬 스토리지 또는 상태 관리(Zustand)에 저장합니다.
* **요청 (Request)**
    * **파라미터 (URL Params)**:
        * `code` (`string`, 필수): 카카오로부터 받은 인가 코드.
* **응답 (Response)**
    * **성공 (200 OK) - JWT 토큰이 URL에 포함**:
      ```text
      리다이렉트 URL: localhost:5173/login-success?token={JWT_TOKEN}
      ```
    * `**token**` (`string`): 백엔드에서 생성한 JWT. 이 토큰을 프론트엔드가 저장하여 모든 API 호출에 사용합니다.
    * **오류 (Error) - JWT 토큰 없이 리다이렉트**:
      ```text
      리다이렉트 URL: localhost:5173/login-fail?error={ERROR_CODE}
      ```
    * `**ERROR_CODE**`는 백엔드에서 정의한 오류 코드입니다. (예: `KAKAO_AUTH_FAILED`)

-----

### 4\. JWT 토큰 재발급

* **API 엔드포인트**: `POST /auth/reissue`
* **설명**: 만료된 JWT 토큰을 재발급합니다. **Refresh Token** 방식을 사용합니다.
* **요청 (Request)**
    * **헤더**:
        * `Authorization: Bearer {expired_token}` (만료된 토큰)
        * `Refresh-Token: {refresh_token}` (Refresh Token)
    * **바디 (Body)**: 없음.
* **응답 (Response)**
    * **성공 (200 OK)**:
        * **헤더**:
            * `Authorization: Bearer {new_jwt}`
            * `Refresh-Token: {new_refresh_token}`
        * 새로운 토큰을 프론트엔드에 전달합니다.
    * **오류 (Error)**:
        * **401 Unauthorized**:
            * `code`: `EXPIRED_REFRESH_TOKEN`
            * `message`: `"리프레시 토큰이 만료되었습니다. 다시 로그인해주세요."`

-----

### 변경 이력

| 버전 | 날짜 | 변경 내용 | 작성자 |
| :--- | :--- | :--- | :--- |
| v1.0.0 | 2025.09.11 | 초기 문서 작성 (백엔드 인가 코드 처리 방식 명시) | 송민재 |
| v1.0.1 | 2025.09.11 | JPA 엔터티 기반 명세 재확인 및 수정 | 송민재 |