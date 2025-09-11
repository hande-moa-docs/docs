

# AI 챗봇 API 명세서

-----

## 문서 정보

- **문서명**: AI 챗봇 API 명세서
- **버전**: v1.0.1
- **작성일**: 2025.09.11
- **작성자**: 송민재
- **최종 수정일**: 2025.09.11

-----

## 1\. 개요

* **API 버전**: v1.0
* **기본 URL**: `localhost:9000/api`
* **인증**: JWT 기반 Bearer Token (`Authorization: Bearer {token}`)
* **설명**: Google Gemini API를 활용한 AI 챗봇 기능에 대한 명세서입니다.

-----

## 2\. 챗봇 대화 시작 및 응답 받기

* **API 엔드포인트**: `POST /chatbot/message`
* **설명**: 사용자 질문에 대한 AI 챗봇의 응답을 요청합니다.
* **요청 (Request)**
    * **헤더**:
        * `Content-Type: application/json`
        * `Authorization: Bearer {JWT}` (필수)
    * **바디 (Body)**:
      ```json
      {
        "message": "코스트코에서 소분하기 좋은 상품은 무엇인가요?"
      }
      ```
    * `**message**` (`string`, 필수): 사용자의 질문 메시지.


* **응답 (Response)**
    * **성공 (200 OK)**:
      ```json
      {
        "status": "success",
        "data": {
          "response": "코스트코에서는 바나나, 소고기 등 소분하기 좋은 상품이 많습니다."
        },
        "message": "AI 챗봇 응답이 성공적으로 반환되었습니다."
      }
      ```
    * `**response**` (`string`): AI 챗봇의 답변.
    * **오류 (Error)**:
        * **400 Bad Request**: `code: INVALID_INPUT`, `message: "잘못된 요청입니다. 메시지를 확인해주세요."`
        * **401 Unauthorized**: `code: UNAUTHORIZED`, `message: "인증 토큰이 유효하지 않습니다."`
        * **500 Internal Server Error**: `code: AI_API_ERROR`, `message: "AI 챗봇 API 호출 중 오류가 발생했습니다."`

-----

## 3\. 챗봇 대화 기록 조회

* **API 엔드포인트**: `GET /chatbot/history`
* **설명**: 현재 사용자의 챗봇 대화 기록을 조회합니다. `ChatbotConversation` 엔터티의 필드를 기반으로 응답합니다.
* **요청 (Request)**
    * **헤더**:
        * `Authorization: Bearer {JWT}` (필수)
* **응답 (Response)**
    * **성공 (200 OK)**:
      ```json
      {
        "status": "success",
        "data": [
          {
            "conversationId": 1,
            "userMessage": "코스트코에서 소분하기 좋은 상품은 무엇인가요?",
            "botResponse": "코스트코에서는 바나나, 소고기 등 소분하기 좋은 상품이 많습니다.",
            "intentType": "SERVICE_GUIDE",
            "createdAt": "2025-09-11T12:00:00Z"
          },
          {
            "conversationId": 2,
            "userMessage": "안녕하세요",
            "botResponse": "안녕하세요! 무엇을 도와드릴까요?",
            "intentType": "GENERAL",
            "createdAt": "2025-09-11T12:01:00Z"
          }
        ],
        "message": "대화 기록 조회가 성공했습니다."
      }
      ```
    * `**conversationId**` (`number`): 대화 기록 ID
    * `**userMessage**` (`string`): 사용자 메시지
    * `**botResponse**` (`string`): 챗봇 답변
    * `**intentType**` (`string`): 의도 유형 (`MEETING_SEARCH`, `SERVICE_GUIDE`, `GENERAL`)
    * `**createdAt**` (`string`): 대화 기록 시간
    * **오류 (Error)**:
        * **401 Unauthorized**: `code: UNAUTHORIZED`, `message: "인증 토큰이 유효하지 않습니다."`
        * **500 Internal Server Error**: `code: DATABASE_ERROR`, `message: "데이터베이스 오류가 발생했습니다."`

-----

## 변경 이력

| 버전   | 날짜         | 변경 내용                  | 작성자 |
|--------|--------------|--------------------------|-----|
| v1.0.0 | 2025.09.11   | 초기 문서 작성           | 송민재 |
| v1.0.1 | 2025.09.11   | JPA 엔터티 기반 응답 필드 상세화 | 송민재 |

-----