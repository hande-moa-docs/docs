# Moa Life 프로젝트 - 개요 및 기능 정리

본 문서는 Moa Life 프로젝트의 **전반적인 개요, 팀 정체성, 핵심 기능, 기술 스택, 우선순위 기능 정의(MoSCoW), 유사 플랫폼 분석 및 차별성**을 정리한 문서입니다.  
Moa가 지향하는 서비스 방향성과 구현 목표를 팀 전체가 공유할 수 있도록 작성되었습니다.

**작성자**:  

**문서 버전**: v1.0

**대상 독자:**
- **팀원 전원 (개발자/디자이너/QA)**: 프로젝트 목표와 기능 이해
- **PM/기획자**: 기능 정의와 범위 확인, 일정 관리 참고
- **운영자**: 서비스 방향성과 핵심 기능 파악
- **신규 합류자**: 프로젝트 전반 구조와 목표를 빠르게 이해하기 위한 온보딩 자료

---

## 프로젝트 개요

### 팀명: **hande (한데)**



---

### 프로젝트명: **Moa Life (모아 라이프)**


---

### 플랫폼명: **Moa (모아)**



---

### 서비스 개요



---

### 프로젝트 핵심 지향점



---

### 주요 특징


---

## 프론트엔드 페이지 구성

1. **메인 페이지**
   

2. **상세 페이지**
    

3. **로그인/ 회원가입**
    

4. **AI 챗봇**
    - 소설 챕터 리스트
    - 챕터 별 소설 내용 -> 본문 내용 모달
    - 마지막 에피소드 위에 다음화 이어쓰기 참여 안내

5. **결제페이지**

---

## 기술 스택

| 구분            | 기술 및 버전                                                                           |
| ------------- |-----------------------------------------------------------------------------------|
| **언어**        | Java 17                                                                           |
| **프레임워크**     | Spring Boot **3.5.4**                                                             |
| **프론트엔드**     | HTML5, CSS3, JavaScript (ES6+), react, react-router                               |
| **보안/인증**     | Spring Security, JWT (JJWT **0.12.3**)                                            |
| **AI 연동**     | Google Gemini API                                                                 |
| **데이터베이스**    | MariaDB (JDBC 드라이버), Spring Data JPA, Hibernate ORM, QueryDSL **5.0.0 (Jakarta)** |
| **빌드/의존성 관리** | Gradle (Spring Dependency Management Plugin **1.1.7**)                            |
| **테스트 프레임워크** | JUnit 5, Mockito (mockito-core, mockito-junit-jupiter), H2 Database               |
| **개발 편의 도구**  | Lombok, Spring Boot DevTools                                                      |
| **로깅**        | Spring Boot Logging, Hibernate SQL Debug/Trace                                    |
| **파일 업로드**    | 로컬 파일 업로드 (`~/Moa/uploads/`)                                                      |
| **협업 도구**     | Git, GitHub, Discord                                                              |
| **개발 환경**     | IntelliJ IDEA, Windows 11                                                         |
| **테스트 환경**    | Chrome, Postman                                                                   |

---

## 기능 목록 – MoSCoW 정리

### Must Have (핵심 기능 - MVP 버전)



---

### Should Have (되도록 구현할 것)



---

### Could Have (시간 여유 시 고려)



### Won’t Have (1차 출시 제외)



---

## 유사 플랫폼 사례 분석

### 당근마켓

* **특징**:  
* **장점**:  
* **한계**:  


---

## Moa의 차별성
