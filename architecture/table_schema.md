# 반띵 데이터베이스 테이블 스키마

## 문서 정보
- **문서명**: 반띵 데이터베이스 테이블 스키마
- **버전**: v1.1.0
- **작성일**: 2025.09.11
- **작성자**: 김경민
- **최종 수정일**: 2025.09.11
- **데이터베이스**: MariaDB

---

## 개요

본 문서는 반띵 프로젝트의 데이터베이스 테이블 구조를 정의합니다. AI 챗봇, 메인페이지, 상세페이지, 사용자 관리, 피드백 시스템을 포함한 통합 스키마입니다.

---

## 테이블 목록

1. [marts](#1-marts) - 마트 정보
2. [users](#2-users) - 사용자 관리
3. [meetings](#3-meetings) - 모임 정보
4. [meeting_participants](#4-meeting_participants) - 모임 참여자
5. [chatbot_conversations](#5-chatbot_conversations) - 챗봇 대화 이력
6. [chatbot_meeting_suggestions](#6-chatbot_meeting_suggestions) - 챗봇 모임 추천
7. [feedbacks](#7-feedbacks) - 피드백 평가

---

## 1. marts
**마트 지점 정보 (지도 API 연동용)**

```sql
CREATE TABLE marts (
    mart_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    mart_name VARCHAR(100) NOT NULL COMMENT '마트명 (예: 코스트코 양재점)',
    mart_brand ENUM('COSTCO', 'TRADERS', 'LOTTE_MART') NOT NULL COMMENT '마트 브랜드',
    address VARCHAR(200) NOT NULL COMMENT '주소',
    latitude DECIMAL(10, 8) NOT NULL COMMENT '위도 (카카오 지도 API용)',
    longitude DECIMAL(11, 8) NOT NULL COMMENT '경도 (카카오 지도 API용)',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    INDEX idx_brand (mart_brand),
    INDEX idx_location (latitude, longitude)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
``` 

---

## 2. users
**사용자 정보 (소셜 로그인 + 신뢰도 시스템)**

```sql
CREATE TABLE users (
    user_id BIGINT AUTO_INCREMENT NOT NULL PRIMARY KEY,
    nickname VARCHAR(50) NOT NULL COMMENT '사용자 닉네임 (고유)',
    profile_image_url VARCHAR(255) COMMENT '프로필 사진 URL',
    self_introduction VARCHAR(255) COMMENT '자기소개',
    provider VARCHAR(50) NOT NULL COMMENT '소셜 로그인 제공자 (kakao)',
    provider_id VARCHAR(255) NOT NULL COMMENT '소셜 로그인 플랫폼 고유 ID',
    trust_score INT NOT NULL DEFAULT 300 COMMENT '신뢰도 점수 (초기값 300)',
    trust_grade ENUM('WARNING', 'BASIC', 'GOOD') NOT NULL DEFAULT 'BASIC' COMMENT '신뢰도 등급',
    no_show_count INT DEFAULT 0 COMMENT '노쇼 횟수',
    agree BOOLEAN NOT NULL DEFAULT FALSE COMMENT '약관 동의 여부',
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '회원 가입 시점',
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '정보 수정 시점',
    deleted_at TIMESTAMP NULL COMMENT '논리적 삭제 시점 (NULL이면 활성 상태)',

    UNIQUE KEY uk_provider_id (provider, provider_id),
    INDEX idx_nickname (nickname),
    INDEX idx_trust_score (trust_score),
    INDEX idx_trust_grade (trust_grade),
    INDEX idx_deleted_at (deleted_at),
    INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```
 
---

## 3. meetings
**모임 정보 (메인 목록 및 상세페이지)**

```sql
CREATE TABLE meetings (
    meeting_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    host_user_id BIGINT NOT NULL COMMENT '모임 호스트 사용자 ID',
    mart_id BIGINT NOT NULL COMMENT '모임 장소 (마트 지점)',
    title VARCHAR(100) NOT NULL COMMENT '모임 제목',
    description TEXT COMMENT '모임 상세 설명',
    meeting_date TIMESTAMP NOT NULL COMMENT '모임 일시',
    max_participants INT NOT NULL DEFAULT 5 COMMENT '최대 인원 (고정 5명)',
    current_participants INT DEFAULT 1 COMMENT '현재 참여자 수 (호스트 포함)',
    status ENUM('RECRUITING', 'FULL', 'ONGOING', 'COMPLETED', 'CANCELLED') DEFAULT 'RECRUITING' COMMENT '모임 상태',
    thumbnail_image_url VARCHAR(500) COMMENT '썸네일 이미지 URL',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    deleted_at TIMESTAMP NULL COMMENT '논리 삭제 시간',

    INDEX idx_host_user (host_user_id),
    INDEX idx_mart (mart_id),
    INDEX idx_status (status),
    INDEX idx_meeting_date (meeting_date),
    INDEX idx_recruiting (status, meeting_date)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

## 4. meeting_participants
**모임 참여자 정보**

```sql
CREATE TABLE meeting_participants (
    participant_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    meeting_id BIGINT NOT NULL COMMENT '모임 ID',
    user_id BIGINT NOT NULL COMMENT '참여자 사용자 ID',
    participant_type ENUM('HOST', 'PARTICIPANT') NOT NULL COMMENT '참여자 유형',
    application_status ENUM('PENDING', 'APPROVED', 'REJECTED') DEFAULT 'PENDING' COMMENT '신청 상태',
    joined_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP COMMENT '참여 신청 시간',
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    UNIQUE KEY uk_meeting_user (meeting_id, user_id),
    INDEX idx_meeting_id (meeting_id),
    INDEX idx_user_id (user_id),
    INDEX idx_status (application_status)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

## 5. chatbot_conversations
**챗봇 대화 이력 (사용자 질문과 AI 응답)**

```sql
CREATE TABLE chatbot_conversations (
    conversation_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    user_id BIGINT NOT NULL COMMENT '사용자 ID (로그인 필수)',
    user_message TEXT NOT NULL COMMENT '사용자 질문',
    bot_response TEXT NOT NULL COMMENT '챗봇 응답',
    intent_type ENUM('MEETING_SEARCH', 'SERVICE_GUIDE', 'GENERAL') DEFAULT 'GENERAL' COMMENT '질문 의도 분류',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    INDEX idx_user_id (user_id),
    INDEX idx_intent_type (intent_type),
    INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
``` 

---

## 6. chatbot_meeting_suggestions
**챗봇이 추천한 모임 목록**

```sql
CREATE TABLE chatbot_meeting_suggestions (
    suggestion_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    conversation_id BIGINT NOT NULL COMMENT '대화 ID',
    meeting_id BIGINT NOT NULL COMMENT '추천된 모임 ID',
    suggestion_reason VARCHAR(200) COMMENT '추천 이유 (간단한 설명)',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    INDEX idx_conversation_id (conversation_id),
    INDEX idx_meeting_id (meeting_id)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
``` 

---

## 7. feedbacks
**모임 참여자 간 피드백 평가 (자기 피드백 방지)**

```sql
CREATE TABLE feedbacks (
    feedback_id BIGINT AUTO_INCREMENT PRIMARY KEY,
    giver_user_id BIGINT NOT NULL COMMENT '피드백을 제공한 사용자 ID',
    receiver_user_id BIGINT NOT NULL COMMENT '피드백을 받은 사용자 ID',
    meeting_id BIGINT NOT NULL COMMENT '피드백이 속한 모임 ID',
    is_positive BOOLEAN NOT NULL COMMENT '긍정적 피드백(true) 또는 부정적 피드백(false)',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,

    UNIQUE KEY uk_feedback_unique (giver_user_id, receiver_user_id, meeting_id),
    CONSTRAINT chk_no_self_feedback CHECK (giver_user_id != receiver_user_id),

    INDEX idx_giver_user (giver_user_id),
    INDEX idx_receiver_user (receiver_user_id),
    INDEX idx_meeting_id (meeting_id),
    INDEX idx_created_at (created_at)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```

---

## 외래키 제약조건

```sql
-- meetings 테이블 외래키
ALTER TABLE meetings ADD FOREIGN KEY (mart_id) REFERENCES marts(mart_id);
ALTER TABLE meetings ADD FOREIGN KEY (host_user_id) REFERENCES users(user_id) ON DELETE CASCADE;

-- meeting_participants 테이블 외래키
ALTER TABLE meeting_participants ADD FOREIGN KEY (meeting_id) REFERENCES meetings(meeting_id) ON DELETE CASCADE;
ALTER TABLE meeting_participants ADD FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE;

-- chatbot_conversations 테이블 외래키
ALTER TABLE chatbot_conversations ADD FOREIGN KEY (user_id) REFERENCES users(user_id) ON DELETE CASCADE;

-- chatbot_meeting_suggestions 테이블 외래키
ALTER TABLE chatbot_meeting_suggestions ADD FOREIGN KEY (conversation_id) REFERENCES chatbot_conversations(conversation_id) ON DELETE CASCADE;
ALTER TABLE chatbot_meeting_suggestions ADD FOREIGN KEY (meeting_id) REFERENCES meetings(meeting_id) ON DELETE CASCADE;

-- feedbacks 테이블 외래키
ALTER TABLE feedbacks ADD FOREIGN KEY (giver_user_id) REFERENCES users(user_id) ON DELETE CASCADE;
ALTER TABLE feedbacks ADD FOREIGN KEY (receiver_user_id) REFERENCES users(user_id) ON DELETE CASCADE;
ALTER TABLE feedbacks ADD FOREIGN KEY (meeting_id) REFERENCES meetings(meeting_id) ON DELETE CASCADE;
```

---

## 더미 데이터 (시연 및 개발용)

### 마트 데이터

```sql
INSERT INTO marts (mart_name, mart_brand, address, latitude, longitude, created_at, updated_at) VALUES
-- 코스트코
('코스트코 양재점', 'COSTCO', '서울특별시 서초구 양재대로 275', 37.4833917, 127.0341544, NOW(), NOW()),
('코스트코 상봉점', 'COSTCO', '서울특별시 중랑구 망우로 354', 37.5955556, 127.0952778, NOW(), NOW()),
('코스트코 송도점', 'COSTCO', '인천광역시 연수구 센트럴로 123', 37.3838889, 126.6597222, NOW(), NOW()),
('코스트코 광명점', 'COSTCO', '경기도 광명시 일직로 17', 37.4167778, 126.8597222, NOW(), NOW()),
('코스트코 의정부점', 'COSTCO', '경기도 의정부시 평화로 525', 37.7361111, 127.0469444, NOW(), NOW()),

-- 이마트 트레이더스
('이마트 트레이더스 월계점', 'TRADERS', '서울특별시 노원구 덕릉로 515', 37.6127778, 127.0580556, NOW(), NOW()),
('이마트 트레이더스 킨텍스점', 'TRADERS', '경기도 고양시 일산서구 킨텍스로 217-6', 37.6686111, 126.7430556, NOW(), NOW()),
('이마트 트레이더스 영등포점', 'TRADERS', '서울특별시 영등포구 영중로 15', 37.5147222, 126.9075000, NOW(), NOW()),
('이마트 트레이더스 하남점', 'TRADERS', '경기도 하남시 미사대로 750', 37.5444444, 127.2225000, NOW(), NOW()),

-- 롯데마트
('롯데마트 서울역점', 'LOTTE_MART', '서울특별시 중구 한강대로 405', 37.5547222, 126.9705556, NOW(), NOW());
```

### 사용자 데이터

```sql
INSERT INTO users (nickname, provider, provider_id, trust_score, trust_grade, agree, created_at, updated_at) VALUES
('김코스트', 'kakao', 'kakao_123456', 300, 'BASIC', TRUE, NOW(), NOW()),
('이소분', 'kakao', 'kakao_789012', 350, 'BASIC', TRUE, NOW(), NOW()),
('박나눔', 'kakao', 'kakao_345678', 280, 'BASIC', TRUE, NOW(), NOW()),
('최절약', 'kakao', 'kakao_111222', 520, 'GOOD', TRUE, NOW(), NOW()),
('정합리', 'kakao', 'kakao_333444', 450, 'BASIC', TRUE, NOW(), NOW()),
('한경제', 'kakao', 'kakao_555666', 80, 'WARNING', TRUE, NOW(), NOW()),
('윤공유', 'kakao', 'kakao_777888', 380, 'BASIC', TRUE, NOW(), NOW()),
('강커뮤', 'kakao', 'kakao_999000', 420, 'BASIC', TRUE, NOW(), NOW());
```

### 모임 데이터

```sql
INSERT INTO meetings (host_user_id, mart_id, title, description, meeting_date, max_participants, created_at, updated_at) VALUES
-- 양재 코스트코 모임들
(1, 1, '코스트코 견과류 소분해요!', '아몬드, 호두 등 견과류를 4명이서 나눠 가져요. 개인 용기 꼭 가져오세요!', '2025-09-15 14:00:00', 4, NOW(), NOW()),
(2, 1, '세제 대용량 소분 모임', '다우니 4L를 3명이서 나누어 가져가실 분!', '2025-09-16 16:00:00', 3, NOW(), NOW()),
(4, 1, '베이커리 빵 소분', '코스트코 머핀과 베이글을 함께 나눠요', '2025-09-18 10:00:00', 5, NOW(), NOW()),

-- 상봉 코스트코 모임들
(3, 2, '상봉 코스트코 냉동식품 소분', '냉동만두와 냉동과일을 함께 소분해요. 아이스박스 준비됩니다.', '2025-09-17 11:00:00', 4, NOW(), NOW()),
(5, 2, '육류 소분 모임', '소고기, 돼지고기 대용량 소분합니다', '2025-09-19 15:00:00', 5, NOW(), NOW()),

-- 트레이더스 모임들
(6, 6, '트레이더스 생활용품 소분', '화장지, 세제 등 생활용품 함께 구매해요', '2025-09-20 13:00:00', 4, NOW(), NOW()),
(7, 7, '킨텍스 트레이더스 과일 소분', '사과, 배 등 과일 박스 소분', '2025-09-21 16:00:00', 3, NOW(), NOW()),

-- 종료된 모임 (피드백 테스트용)
(1, 1, '완료된 모임 - 쌀 소분', '20kg 쌀을 5명이서 나눠가졌습니다', '2025-09-10 14:00:00', 5, NOW(), NOW());

-- 마지막 모임 상태를 COMPLETED로 변경
UPDATE meetings SET status = 'COMPLETED' WHERE meeting_id = 8;
```

### 모임 참여자 데이터

```sql
-- 호스트들 (자동으로 APPROVED 상태)
INSERT INTO meeting_participants (meeting_id, user_id, participant_type, application_status, joined_at, updated_at) VALUES
(1, 1, 'HOST', 'APPROVED', NOW(), NOW()),
(2, 2, 'HOST', 'APPROVED', NOW(), NOW()),
(3, 3, 'HOST', 'APPROVED', NOW(), NOW()),
(4, 4, 'HOST', 'APPROVED', NOW(), NOW()),
(5, 5, 'HOST', 'APPROVED', NOW(), NOW()),
(6, 6, 'HOST', 'APPROVED', NOW(), NOW()),
(7, 7, 'HOST', 'APPROVED', NOW(), NOW()),
(8, 1, 'HOST', 'APPROVED', NOW(), NOW());

-- 일반 참여자들
INSERT INTO meeting_participants (meeting_id, user_id, participant_type, application_status, joined_at, updated_at) VALUES
-- 모임 1 참여자들
(1, 2, 'PARTICIPANT', 'APPROVED', NOW(), NOW()),
(1, 3, 'PARTICIPANT', 'PENDING', NOW(), NOW()),

-- 모임 2 참여자들
(2, 1, 'PARTICIPANT', 'APPROVED', NOW(), NOW()),
(2, 4, 'PARTICIPANT', 'PENDING', NOW(), NOW()),

-- 모임 3 참여자들
(3, 5, 'PARTICIPANT', 'APPROVED', NOW(), NOW()),
(3, 6, 'PARTICIPANT', 'APPROVED', NOW(), NOW()),

-- 완료된 모임 8의 참여자들 (피드백 테스트용)
(8, 2, 'PARTICIPANT', 'APPROVED', NOW(), NOW()),
(8, 3, 'PARTICIPANT', 'APPROVED', NOW(), NOW()),
(8, 4, 'PARTICIPANT', 'APPROVED', NOW(), NOW()),
(8, 5, 'PARTICIPANT', 'APPROVED', NOW(), NOW());
```

### 챗봇 대화 이력 데이터

```sql
INSERT INTO chatbot_conversations (user_id, user_message, bot_response, intent_type, created_at, updated_at) VALUES
(1, '양재 코스트코 근처에 견과류 소분 모임 있나요?', '네, 현재 양재 코스트코에서 견과류 소분 모임이 진행중입니다. 9월 15일 오후 2시에 진행되는 모임에 참여하실 수 있습니다.', 'MEETING_SEARCH', NOW(), NOW()),
(2, '소분 모임은 어떻게 참여하나요?', '소분 모임 참여는 간단합니다. 원하는 모임을 선택하고 참여 신청을 하시면, 호스트의 승인 후 참여가 확정됩니다.', 'SERVICE_GUIDE', NOW(), NOW()),
(3, '냉동식품 소분할 때 주의사항이 있나요?', '냉동식품 소분 시에는 아이스박스나 보냉백을 꼭 준비하시고, 소분 후 빠른 시간 내에 냉동보관하시기 바랍니다.', 'SERVICE_GUIDE', NOW(), NOW()),
(4, '안녕하세요', '안녕하세요! 반띵 챗봇입니다. 소분 모임에 대해 궁금한 것이 있으시면 언제든 물어보세요.', 'GENERAL', NOW(), NOW());
```

### 챗봇 모임 추천 데이터

```sql
INSERT INTO chatbot_meeting_suggestions (conversation_id, meeting_id, suggestion_reason, created_at, updated_at) VALUES
(1, 1, '사용자가 찾던 양재 코스트코 견과류 소분 모임', NOW(), NOW()),
(3, 3, '냉동식품 관련 질문에 대한 관련 모임 추천', NOW(), NOW());
```

### 피드백 데이터

```sql
INSERT INTO feedbacks (giver_user_id, receiver_user_id, meeting_id, is_positive, created_at, updated_at) VALUES
-- 모임 8에서의 상호 피드백들 (호스트: 1번, 참여자: 2,3,4,5번)
(1, 2, 8, TRUE, NOW(), NOW()),   -- 호스트 -> 참여자2 긍정
(1, 3, 8, TRUE, NOW(), NOW()),   -- 호스트 -> 참여자3 긍정
(1, 4, 8, FALSE, NOW(), NOW()),  -- 호스트 -> 참여자4 부정 (노쇼)
(1, 5, 8, TRUE, NOW(), NOW()),   -- 호스트 -> 참여자5 긍정

(2, 1, 8, TRUE, NOW(), NOW()),   -- 참여자2 -> 호스트 긍정
(3, 1, 8, TRUE, NOW(), NOW()),   -- 참여자3 -> 호스트 긍정
(5, 1, 8, TRUE, NOW(), NOW()),   -- 참여자5 -> 호스트 긍정

-- 참여자들 간 상호 피드백
(2, 3, 8, TRUE, NOW(), NOW()),   -- 참여자2 -> 참여자3 긍정
(3, 2, 8, TRUE, NOW(), NOW()),   -- 참여자3 -> 참여자2 긍정
(2, 5, 8, TRUE, NOW(), NOW()),   -- 참여자2 -> 참여자5 긍정
(5, 2, 8, TRUE, NOW(), NOW());   -- 참여자5 -> 참여자2 긍정
```

### 현재 참여자 수 업데이트

```sql
UPDATE meetings SET current_participants = (
    SELECT COUNT(*) 
    FROM meeting_participants 
    WHERE meeting_participants.meeting_id = meetings.meeting_id 
    AND application_status = 'APPROVED'
);
```

---

## 테스트 시나리오 설명

### 마트 지점 (10개)
- **코스트코**: 양재점, 상봉점, 송도점, 광명점, 의정부점 (5개)
- **이마트 트레이더스**: 월계점, 킨텍스점, 영등포점, 하남점 (4개)
- **롯데마트**: 서울역점 (1개)

### 사용자 (8명)
- **김코스트** (300점, BASIC) - 1번 사용자, 모임 1, 8 호스트
- **이소분** (350점, BASIC) - 2번 사용자, 모임 2 호스트
- **박나눔** (280점, BASIC) - 3번 사용자, 모임 3 호스트
- **최절약** (520점, GOOD) - 4번 사용자, 신뢰도 최고 등급
- **정합리** (450점, BASIC) - 5번 사용자, 모임 5 호스트
- **한경제** (80점, WARNING) - 6번 사용자, 신뢰도 경고 등급
- **윤공유** (380점, BASIC) - 7번 사용자, 모임 7 호스트
- **강커뮤** (420점, BASIC) - 8번 사용자

### 모임 현황 (8개)
- **진행중 모임**: 7개 (RECRUITING 상태)
- **완료된 모임**: 1개 (COMPLETED 상태, 피드백 테스트용)
- **참여 현황**: 호스트와 참여자들의 다양한 상태 조합

### 피드백 시스템 테스트
- **완료된 모임 8번**에서 상호 피드백 예시
- **긍정 피드백**: 대부분의 참여자 간 긍정적 평가
- **부정 피드백**: 4번 사용자에 대한 노쇼 피드백 1건
- **자기 피드백 방지**: CHECK 제약조건으로 구현

---

## 주요 특징

1. **타임스탬프 처리**: `created_at`, `updated_at`을 `NOW()`로 명시적 설정
2. **참여자 상태 관리**: PENDING, APPROVED, REJECTED 상태 관리
3. **신뢰도 시스템**: 점수 기반 등급 시스템 (WARNING/BASIC/GOOD)
4. **피드백 시스템**: 완료된 모임에서 상호 평가 가능
5. **챗봇 연동**: AI 추천 모임과 대화 이력 연결

---

## 버전 이력

| 버전     | 날짜         | 변경 내용 | 작성자 |
|--------|------------|-----------|-----|
| v1.0.0 | 2025.09.10 | 초기 스키마 작성 | 김경민 |
| v1.0.1 | 2025.09.11 | 디폴트 값 수정 | 김경민 |
| v1.0.2 | 2025.09.11 | 피드백 시스템 추가 | 김경민 |
| v1.0.3 | 2025.09.11 | 제약조건 수정, 문서화 | 김경민 |
| v1.0.4 | 2025.09.11 | 더미 데이터 추가 | 김경민 |
| v1.1.0 | 2025.09.11 | data.sql 동기화된 더미 데이터로 업데이트 | 김경민 |