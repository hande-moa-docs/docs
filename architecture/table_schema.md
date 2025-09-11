# 반띵 데이터베이스 테이블 스키마

## 문서 정보
- **문서명**: 반띵 데이터베이스 테이블 스키마
- **버전**: v1.0.3
- **작성일**: 2025.09.11
- **데이터베이스**: 김경민
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
    nickname VARCHAR(50) NOT NULL UNIQUE COMMENT '사용자 닉네임 (고유)',
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

## 더미 데이터 설명

### 테스트 시나리오
- **마트**: 10개 지점 (코스트코 5개, 트레이더스 4개, 롯데마트 1개)
- **사용자**: 8명 (다양한 신뢰도 등급)
- **모임**: 8개 (진행중 7개, 완료 1개)
- **피드백**: 완료된 모임에서 상호 평가 시스템 테스트 가능

### 신뢰도 등급별 사용자
- **GOOD**: 최절약 (520점)
- **BASIC**: 이소분, 정합리, 윤공유, 강커뮤 등
- **WARNING**: 한경제 (80점)

---

## 더미 데이터 (시연 및 개발용)

### 마트 데이터

```sql
INSERT INTO marts (mart_name, mart_brand, address, latitude, longitude) VALUES
-- 코스트코
('코스트코 양재점', 'COSTCO', '서울특별시 서초구 양재대로 275', 37.4833917, 127.0341544),
('코스트코 상봉점', 'COSTCO', '서울특별시 중랑구 망우로 354', 37.5955556, 127.0952778),
('코스트코 송도점', 'COSTCO', '인천광역시 연수구 센트럴로 123', 37.3838889, 126.6597222),
('코스트코 광명점', 'COSTCO', '경기도 광명시 일직로 17', 37.4167778, 126.8597222),
('코스트코 의정부점', 'COSTCO', '경기도 의정부시 평화로 525', 37.7361111, 127.0469444),

-- 이마트 트레이더스
('이마트 트레이더스 월계점', 'TRADERS', '서울특별시 노원구 덕릉로 515', 37.6127778, 127.0580556),
('이마트 트레이더스 킨텍스점', 'TRADERS', '경기도 고양시 일산서구 킨텍스로 217-6', 37.6686111, 126.7430556),
('이마트 트레이더스 영등포점', 'TRADERS', '서울특별시 영등포구 영중로 15', 37.5147222, 126.9075000),
('이마트 트레이더스 하남점', 'TRADERS', '경기도 하남시 미사대로 750', 37.5444444, 127.2225000),

-- 롯데마트
('롯데마트 서울역점', 'LOTTE_MART', '서울특별시 중구 한강대로 405', 37.5547222, 126.9705556);
```

### 사용자 데이터

```sql
INSERT INTO users (nickname, provider, provider_id, trust_score, trust_grade, agree) VALUES
('김코스트', 'kakao', 'kakao_123456', 300, 'BASIC', TRUE),
('이소분', 'kakao', 'kakao_789012', 350, 'BASIC', TRUE),
('박나눔', 'kakao', 'kakao_345678', 280, 'BASIC', TRUE),
('최절약', 'kakao', 'kakao_111222', 520, 'GOOD', TRUE),
('정합리', 'kakao', 'kakao_333444', 450, 'BASIC', TRUE),
('한경제', 'kakao', 'kakao_555666', 80, 'WARNING', TRUE),
('윤공유', 'kakao', 'kakao_777888', 380, 'BASIC', TRUE),
('강커뮤', 'kakao', 'kakao_999000', 420, 'BASIC', TRUE);
```

### 모임 데이터

```sql
INSERT INTO meetings (host_user_id, mart_id, title, description, meeting_date, max_participants) VALUES
-- 양재 코스트코 모임들
(1, 1, '코스트코 견과류 소분해요!', '아몬드, 호두 등 견과류를 4명이서 나눠 가져요. 개인 용기 꼭 가져오세요!', '2025-09-15 14:00:00', 4),
(2, 1, '세제 대용량 소분 모임', '다우니 4L를 3명이서 나누어 가져가실 분!', '2025-09-16 16:00:00', 3),
(4, 1, '베이커리 빵 소분', '코스트코 머핀과 베이글을 함께 나눠요', '2025-09-18 10:00:00', 5),

-- 상봉 코스트코 모임들
(3, 2, '상봉 코스트코 냉동식품 소분', '냉동만두와 냉동과일을 함께 소분해요. 아이스박스 준비됩니다.', '2025-09-17 11:00:00', 4),
(5, 2, '육류 소분 모임', '소고기, 돼지고기 대용량 소분합니다', '2025-09-19 15:00:00', 5),

-- 트레이더스 모임들
(6, 6, '트레이더스 생활용품 소분', '화장지, 세제 등 생활용품 함께 구매해요', '2025-09-20 13:00:00', 4),
(7, 7, '킨텍스 트레이더스 과일 소분', '사과, 배 등 과일 박스 소분', '2025-09-21 16:00:00', 3),

-- 종료된 모임 (피드백 테스트용)
(8, 1, '완료된 모임 - 쌀 소분', '20kg 쌀을 5명이서 나눠가졌습니다', '2025-09-10 14:00:00', 5);

-- 마지막 모임 상태를 COMPLETED로 변경
UPDATE meetings SET status = 'COMPLETED' WHERE meeting_id = 8;
```

### 모임 참여자 데이터

```sql
-- 호스트들 (자동으로 APPROVED 상태)
INSERT INTO meeting_participants (meeting_id, user_id, participant_type, application_status) VALUES
(1, 1, 'HOST', 'APPROVED'),
(2, 2, 'HOST', 'APPROVED'),
(3, 3, 'HOST', 'APPROVED'),
(4, 4, 'HOST', 'APPROVED'),
(5, 5, 'HOST', 'APPROVED'),
(6, 6, 'HOST', 'APPROVED'),
(7, 7, 'HOST', 'APPROVED'),
(8, 8, 'HOST', 'APPROVED');

-- 일반 참여자들
INSERT INTO meeting_participants (meeting_id, user_id, participant_type, application_status) VALUES
-- 모임 1 참여자들
(1, 2, 'PARTICIPANT', 'APPROVED'),
(1, 3, 'PARTICIPANT', 'PENDING'),

-- 모임 2 참여자들
(2, 1, 'PARTICIPANT', 'APPROVED'),
(2, 4, 'PARTICIPANT', 'PENDING'),

-- 모임 3 참여자들
(3, 5, 'PARTICIPANT', 'APPROVED'),
(3, 6, 'PARTICIPANT', 'APPROVED'),

-- 완료된 모임 8의 참여자들 (피드백 테스트용)
(8, 1, 'PARTICIPANT', 'APPROVED'),
(8, 2, 'PARTICIPANT', 'APPROVED'),
(8, 3, 'PARTICIPANT', 'APPROVED'),
(8, 4, 'PARTICIPANT', 'APPROVED');
```

### 챗봇 대화 이력 데이터

```sql
INSERT INTO chatbot_conversations (user_id, user_message, bot_response, intent_type) VALUES
(1, '양재 코스트코 근처에 견과류 소분 모임 있나요?', '네, 현재 양재 코스트코에서 견과류 소분 모임이 진행중입니다. 9월 15일 오후 2시에 진행되는 모임에 참여하실 수 있습니다.', 'MEETING_SEARCH'),
(2, '소분 모임은 어떻게 참여하나요?', '소분 모임 참여는 간단합니다. 원하는 모임을 선택하고 참여 신청을 하시면, 호스트의 승인 후 참여가 확정됩니다.', 'SERVICE_GUIDE'),
(3, '냉동식품 소분할 때 주의사항이 있나요?', '냉동식품 소분 시에는 아이스박스나 보냉백을 꼭 준비하시고, 소분 후 빠른 시간 내에 냉동보관하시기 바랍니다.', 'SERVICE_GUIDE'),
(4, '안녕하세요', '안녕하세요! 반띵 챗봇입니다. 소분 모임에 대해 궁금한 것이 있으시면 언제든 물어보세요.', 'GENERAL');
```

### 챗봇 모임 추천 데이터

```sql
INSERT INTO chatbot_meeting_suggestions (conversation_id, meeting_id, suggestion_reason) VALUES
(1, 1, '사용자가 찾던 양재 코스트코 견과류 소분 모임'),
(3, 3, '냉동식품 관련 질문에 대한 관련 모임 추천');
```

### 피드백 데이터

```sql
INSERT INTO feedbacks (giver_user_id, receiver_user_id, meeting_id, is_positive) VALUES
-- 모임 8에서의 상호 피드백들
(8, 1, 8, TRUE),   -- 호스트 -> 참여자1 긍정
(8, 2, 8, TRUE),   -- 호스트 -> 참여자2 긍정
(8, 3, 8, FALSE),  -- 호스트 -> 참여자3 부정 (노쇼)
(8, 4, 8, TRUE),   -- 호스트 -> 참여자4 긍정

(1, 8, 8, TRUE),   -- 참여자1 -> 호스트 긍정
(2, 8, 8, TRUE),   -- 참여자2 -> 호스트 긍정
(4, 8, 8, TRUE),   -- 참여자4 -> 호스트 긍정

-- 참여자들 간 상호 피드백
(1, 2, 8, TRUE),   -- 참여자1 -> 참여자2 긍정
(2, 1, 8, TRUE),   -- 참여자2 -> 참여자1 긍정
(1, 4, 8, TRUE),   -- 참여자1 -> 참여자4 긍정
(4, 1, 8, TRUE);   -- 참여자4 -> 참여자1 긍정
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

## 주요 변경사항

1. **meetings 테이블**: 불필요한 UNIQUE 제약조건 제거 (host_user_id는 FK만 유지)
2. **meeting_participants 테이블**: application_status 기본값을 'PENDING'으로 변경
3. **feedbacks 테이블**: CHECK 제약조건으로 자기 피드백 방지 (UNIQUE 제약조건 제거)
4. **max_participants**: 기본값 5로 설정하여 고정값 명시

---

## 버전 이력

| 버전     | 날짜         | 변경 내용 | 작성자 |
|--------|------------|-----------|-----|
| v1.0.0 | 2025.09.10 | 초기 스키마 작성 | 김경민 |
| v1.0.1 | 2025.09.11 | 디폴트 값 수정 | 김경민 |
| v1.0.2 | 2025.09.11 | 피드백 시스템 추가 | 김경민 |
| v1.0.3 | 2025.09.11 | 제약조건 수정, 문서화 | 김경민 |