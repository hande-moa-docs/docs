# API 명세서 초안

---

### API 명세서 초안

#### **1.1 문서 정보**

* **문서명**: 소분 거래 서비스 API 명세서
* **버전**: v1.0.0 (초안)
* **작성일**: 2025.09.10
* **작성자**: 송민재
* **개요**: 본 문서는 '대용량 상품 소분 거래 서비스'의 백엔드 API 명세서로, 클라이언트(웹/모바일)와의 통신 규약을 정의한다.
---
### **1.2 대상 독자**

* **백엔드 개발자**: API 설계 및 구현, 스펙 검증
*  **프론트엔드 개발자**: API 연동 및 요청/응답 데이터 활용
*  **QA 엔지니어**: API 테스트 케이스 및 시나리오 설계
*  **운영자/PM**: 기능 범위 및 서비스 동작 확인
---

#### **2. 기본 정보**

* **API 종류**: RESTful API
* **요청/응답 형식**: JSON
* **인증**: JWT(JSON Web Token)을 사용한 사용자 인증
* **기본 URL**: `http://localhost:9009/api`

---

### **3. API 목록**

#### **3.1 사용자 관련 API**

| 기능 | URL | HTTP Method | 설명 |
|---|---|---|---|
| **회원가입** | `/users/signup` | `POST` | 이메일, 비밀번호, 닉네임 등을 이용한 회원가입 |
| **로그인** | `/users/login` | `POST` | 이메일과 비밀번호를 이용한 로그인 및 JWT 토큰 발급 |
| **프로필 조회** | `/users/{userId}` | `GET` | 특정 사용자의 프로필 정보 조회 |
| **프로필 수정** | `/users/{userId}` | `PUT` | 사용자의 프로필 정보 수정 |

#### **3.2 파티(소분 거래) 관련 API**

| 기능 | URL | HTTP Method | 설명 |
|---|---|---|---|
| **파티 목록 조회** | `/parties` | `GET` | 필터(지역, 상품 종류 등)에 따른 파티 목록 조회 |
| **파티 상세 조회** | `/parties/{partyId}` | `GET` | 특정 파티의 상세 정보 조회 |
| **파티 생성** | `/parties` | `POST` | 사용자가 새로운 소분 거래 파티 생성 |
| **파티 참여** | `/parties/{partyId}/join` | `POST` | 특정 파티에 참여 신청 |
| **파티 나가기** | `/parties/{partyId}/leave` | `DELETE` | 파티 참여 취소 |
| **파티 수정** | `/parties/{partyId}` | `PUT` | 파티 정보 수정 (파티장만 가능) |
| **파티 삭제** | `/parties/{partyId}` | `DELETE` | 파티 삭제 (파티장만 가능) |

#### **3.3 상품 관련 API**

| 기능 | URL | HTTP Method | 설명 |
|---|---|---|---|
| **상품 목록 조회** | `/items` | `GET` | 모든 상품 목록 또는 검색을 통한 상품 목록 조회 |
| **상품 상세 조회** | `/items/{itemId}` | `GET` | 특정 상품의 상세 정보 조회 |
| **상품 등록** | `/items` | `POST` | 판매할 상품 정보 등록 |

#### **3.4 채팅 관련 API**

| 기능 | URL | HTTP Method | 설명 |
|---|---|---|---|
| **채팅방 목록 조회** | `/chats` | `GET` | 사용자가 참여 중인 채팅방 목록 조회 |
| **채팅 메시지 조회** | `/chats/{chatId}/messages` | `GET` | 특정 채팅방의 메시지 내역 조회 |
| **메시지 전송** | `/chats/{chatId}/messages` | `POST` | 채팅방에 메시지 전송 |

---

### **4. 데이터 모델 (초안)**

#### **4.1 사용자 (User)**

* `userId` (UUID): 사용자 고유 식별자
* `email` (string): 이메일
* `nickname` (string): 닉네임
* `location` (string): 주소 또는 지역

#### **4.2 파티 (Party)**

* `partyId` (UUID): 파티 고유 식별자
* `title` (string): 파티 제목
* `hostId` (UUID): 파티장(사용자) ID
* `itemId` (UUID): 거래 대상 상품 ID
* `currentMembers` (int): 현재 참여 인원 수
* `maxMembers` (int): 최대 모집 인원 수
* `location` (string): 거래 장소
* `status` (string): 파티 상태 (예: `모집중`, `거래예정`, `거래완료`)
* `createdAt` (timestamp): 파티 생성 일시

#### **4.3 상품 (Item)**

* `itemId` (UUID): 상품 고유 식별자
* `name` (string): 상품명
* `description` (string): 상세 설명
* `totalPrice` (number): 총 가격
* `unitPrice` (number): 소분 단위 가격

#### **4.4 채팅 (Chat)**

* `chatId` (UUID): 채팅방 고유 식별자
* `partyId` (UUID): 연결된 파티 ID
* `participants` (array): 참여자 목록 (사용자 ID)

---

이 초안은 추후 논의를 통해 더 구체화되어야 합니다. 특히 `businessLogic.md`와 `requirements.md`를 기반으로 각 API의 요청/응답 형식(Request/Response Body), 성공/실패 코드(Status Code)와 에러 메시지를 명확히 정의하는 작업이 필요합니다.