## 일반 API 명세서

-----

### 문서 정보

- **문서명**: 일반 API 명세서
- **버전**: v1.0.1
- **작성일**: 2025.09.11
- **작성자**: 송민재
- **최종 수정일**: 2025.09.11

-----

### 1\. 개요

* **API 버전**: v1.0
* **기본 URL**: `localhost:9000/api`
* **인증**: JWT 기반 Bearer Token (`Authorization: Bearer {token}`)
* **설명**: 서비스의 주요 핵심 기능에 대한 API 명세서입니다.

-----

### 2\. 사용자 프로필 조회

* **API 엔드포인트**: `GET /users/me`
* **설명**: 현재 로그인한 사용자의 프로필 정보를 조회합니다. `User` 엔터티의 필드를 기반으로 응답합니다.
* **요청 (Request)**
    * **헤더**:
        * `Authorization: Bearer {JWT}` (필수)
* **응답 (Response)**
    * **성공 (200 OK)**:
      ```json
      {
        "status": "success",
        "data": {
          "userId": 1,
          "nickname": "반띵러123",
          "profileImageUrl": "https://k.kakaocdn.net/image/...",
          "selfIntroduction": "안녕하세요! 코스트코 소분 모임에 참여하고 싶어요.",
          "trustScore": 300,
          "trustGrade": "BASIC",
          "createdAt": "2025-09-11T10:00:00Z"
        },
        "message": "사용자 프로필 조회가 성공했습니다."
      }
      ```
    * `**userId**` (`number`): 사용자 ID.
    * `**nickname**` (`string`): 사용자 닉네임.
    * `**profileImageUrl**` (`string`): 카카오 프로필 이미지 URL.
    * `**selfIntroduction**` (`string`): 자기소개.
    * `**trustScore**` (`number`): 신뢰도 점수.
    * `**trustGrade**` (`string`): 신뢰 등급 (`WARNING`, `BASIC`, `GOOD`).
    * `**createdAt**` (`string`): 가입 시간.
    * **오류 (Error)**:
        * **401 Unauthorized**: `code: INVALID_TOKEN`
        * **404 Not Found**: `code: USER_NOT_FOUND`

-----

### 3\. 모임 리스트 조회

* **API 엔드포인트**: `GET /groups`
* **설명**: 지도 기반으로 현재 위치 주변의 모임 리스트를 조회합니다. `Meeting` 엔터티의 필드를 기반으로 응답합니다.
* **요청 (Request)**
    * **헤더**: 없음 (비회원도 접근 가능)
    * **파라미터 (URL Params)**:
        * `lat` (`number`, 필수): 현재 위도.
        * `lng` (`number`, 필수): 현재 경도.
        * `radius` (`number`, 선택): 검색 반경 (km). 기본값: 5km.
* **응답 (Response)**
    * **성공 (200 OK)**:
      ```json
      {
        "status": "success",
        "data": [
          {
            "meetingId": 101,
            "title": "코스트코 견과류 소분해요!",
            "thumbnailImageUrl": "https://example.com/images/...",
            "meetingDate": "2025-09-15T14:00:00Z",
            "status": "RECRUITING",
            "currentParticipants": 3,
            "maxParticipants": 4,
            "location": {
              "lat": 37.5665,
              "lng": 126.9780
            }
          }
        ],
        "message": "모임 리스트 조회가 성공했습니다."
      }
      ```
    * `**meetingId**` (`number`): 모임 ID.
    * `**title**` (`string`): 모임 제목.
    * `**thumbnailImageUrl**` (`string`): 모임 썸네일 이미지 URL.
    * `**meetingDate**` (`string`): 모임 시간 (ISO 8601).
    * `**status**` (`string`): 모임 상태 (`RECRUITING`, `FULL`, `ONGOING`, `COMPLETED`, `CANCELLED`).
    * `**currentParticipants**` (`number`): 현재 참여 인원.
    * `**maxParticipants**` (`number`): 최대 참여 인원.
    * `**location**` (`object`): 위도, 경도 정보.
    * **오류 (Error)**:
        * **400 Bad Request**: `code: INVALID_LOCATION`
        * **500 Internal Server Error**: `code: DATABASE_ERROR`

-----

### 4\. 모임 상세 조회

* **API 엔드포인트**: `GET /groups/{groupId}`
* **설명**: 특정 모임의 상세 정보를 조회합니다. `Meeting` 엔터티와 연관 관계에 있는 필드를 모두 포함합니다.
* **요청 (Request)**
    * **헤더**:
        * `Authorization: Bearer {JWT}` (필수, 비회원은 조회 불가)
    * **파라미터 (URL Params)**:
        * `groupId` (`number`, 필수): 조회할 모임의 ID.
* **응답 (Response)**
    * **성공 (200 OK)**:
      ```json
      {
        "status": "success",
        "data": {
          "meetingId": 101,
          "title": "코스트코 견과류 소분해요!",
          "description": "아몬드, 호두 등 견과류를 4명이서 나눠 가져요. 개인 용기 꼭 가져오세요!",
          "thumbnailImageUrl": "https://example.com/images/...",
          "meetingDate": "2025-09-15T14:00:00Z",
          "status": "RECRUITING",
          "currentParticipants": 3,
          "maxParticipants": 4,
          "hostUser": {
            "userId": 1,
            "nickname": "호스트123"
          },
          "mart": {
            "martId": 1,
            "martName": "코스트코 양재점",
            "address": "서울 서초구 양재대로 123",
            "latitude": 37.5665,
            "longitude": 126.9780
          },
          "participants": [
            {
              "userId": 1,
              "nickname": "호스트123",
              "participantType": "HOST"
            },
            {
              "userId": 2,
              "nickname": "참여자22",
              "participantType": "PARTICIPANT"
            }
          ]
        },
        "message": "모임 상세 조회가 성공했습니다."
      }
      ```
    * `**hostUser**` (`object`): 호스트 사용자 정보.
    * `**mart**` (`object`): 마트 정보.
    * `**participants**` (`array`): 참여자 목록 (`MeetingParticipant` 엔터티를 기반으로 구성).
    * **오류 (Error)**:
        * **401 Unauthorized**: `code: INVALID_TOKEN`
        * **404 Not Found**: `code: GROUP_NOT_FOUND`

-----

### 5\. 피드백 관리

#### 5.1 피드백 작성

* **API 엔드포인트**: `POST /feedbacks`
* **설명**: 모임 종료 후 참여자에 대한 피드백을 작성합니다. `Feedback` 엔터티의 필드를 기반으로 합니다.
* **요청 (Request)**
    * **헤더**:
        * `Content-Type: application/json`
        * `Authorization: Bearer {JWT}` (필수)
    * **바디 (Body)**:
      ```json
      {
        "meetingId": 101,
        "receiverUserId": 2,
        "isPositive": true
      }
      ```
    * `**meetingId**` (`number`, 필수): 피드백 대상 모임 ID.
    * `**receiverUserId**` (`number`, 필수): 피드백을 받을 사용자 ID.
    * `**isPositive**` (`boolean`, 필수): 피드백 내용 (긍정/부정).
* **응답 (Response)**
    * **성공 (201 Created)**:
      ```json
      {
        "status": "success",
        "message": "피드백이 성공적으로 등록되었습니다."
      }
      ```
    * **오류 (Error)**:
        * **400 Bad Request**: `code: INVALID_FEEDBACK_REQUEST`
        * **401 Unauthorized**: `code: UNAUTHORIZED`
        * **404 Not Found**: `code: MEETING_NOT_FOUND` or `USER_NOT_FOUND`

#### 5.2 사용자 피드백 조회

* **API 엔드포인트**: `GET /users/{userId}/feedbacks`
* **설명**: 특정 사용자가 받은 피드백 리스트를 조회합니다.
* **요청 (Request)**
    * **헤더**:
        * `Authorization: Bearer {JWT}` (선택)
    * **파라미터 (URL Params)**:
        * `userId` (`number`, 필수): 피드백을 조회할 사용자 ID.
* **응답 (Response)**
    * **성공 (200 OK)**:
      ```json
      {
        "status": "success",
        "data": [
          {
            "feedbackId": 1,
            "giverUser": {
              "userId": 1,
              "nickname": "호스트123"
            },
            "isPositive": true,
            "createdAt": "2025-09-17T18:00:00Z"
          }
        ],
        "message": "피드백 리스트 조회가 성공했습니다."
      }
      ```
    * `**feedbackId**` (`number`): 피드백 ID.
    * `**giverUser**` (`object`): 피드백을 준 사용자 정보.
    * `**isPositive**` (`boolean`): 피드백 내용.
    * `**createdAt**` (`string`): 피드백 작성 시간.
    * **오류 (Error)**:
        * **404 Not Found**: `code: USER_NOT_FOUND`

-----

### 변경 이력

| 버전 | 날짜 | 변경 내용 | 작성자 |
| :--- | :--- | :--- | :--- |
| v1.0.0 | 2025.09.11 | 초기 문서 작성 | 박현수 |
| v1.0.1 | 2025.09.11 | JPA 엔터티 기반 응답 필드 및 신규 API 추가 | 박현수 |