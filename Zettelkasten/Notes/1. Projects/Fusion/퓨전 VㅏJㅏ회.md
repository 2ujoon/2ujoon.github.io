# 퓨전 VㅏJㅏ회
<details>
<summary>스터디 정보</summary>

- 기간 : 2023.03 ~ 2023.11
- 참여자 : 권유진 주임, 김민규 선임, 김민서 주임, 박경현 주임, 서민규 주임, 서보민 책임, 유준혁 주임, 이상원 주임, 이우준 주임
- 강의 정보
  - [자바 ORM 표준 JPA프로그래밍(기본편)](https://www.inflearn.com/course/ORM-JPA-Basic/dashboard)
  - [실전! 스프링 부트와 JPA 활용1 - 웹 애플리케이션 개발](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-%ED%99%9C%EC%9A%A9-1)
  - [실전! 스프링 부트와 JPA 활용2 - API 개발과 성능 최적화](https://www.inflearn.com/course/%EC%8A%A4%ED%94%84%EB%A7%81%EB%B6%80%ED%8A%B8-JPA-API%EA%B0%9C%EB%B0%9C-%EC%84%B1%EB%8A%A5%EC%B5%9C%EC%A0%81%ED%99%94)
  - [프로젝트로 배우는 Vue.js 3](https://www.inflearn.com/course/vue-기초-익히기)

</details>
<br>
 
## 정리
[[자바 ORM 표준 JPA 프로그래밍 - 기본편]]

---
## 2023.03.09 목  1회
<!-- <details> -->
<!-- <summary>보기</summary> -->
<br>

### 진도
- 섹션 0. 강좌 소개
  - 강좌 소개 및 수업자료
  <br>
  
- 섹션 1. JPA 소개
  - SQL 중심적인 개발의 문제점
  - JPA 소개
  <br>
  
- 섹션 2. JPA 시작하기
  - Hello JPA - 프로젝트 생성
  - Hello JPA - 애플리케이션 개발
<br>

</details>
<br>

---
## 2023.03.17 금 2회
<!-- <details> -->
<!-- <summary>보기</summary> -->
<br>

### 진도
- 섹션 3. 영속성 관리 - 내부 동작 방식
  - 영속성 컨텍스트 1
  - 영속성 컨텍스트 2
  - 플러시
  - 준영속 상태
  - 정리
  <br>
  
- 섹션 4. 엔티티 매핑
  - 객체와 테이블 매핑
  - 데이터베이스 스키마 자동 생성
  - 필드와 컬럼 매핑
  - 기본 키 매핑
  - 실전 예제1 - 요구사항 분석과 기본 매핑
  <br>
  
### ???
#### 1. @SequenceGenerator의 allocationSize

- DB에 매번 시퀀스를 호출하지 않음으로써 최적화 하기 위한 속성
- 설정된 allocationSize 값은 앱 내(WAS)에서 공유됨
- 동작 방식(***`!검증 필요`***)
  1. 최초 persist() 호출 시 DB 시퀀스를 두 번 호출 후, 첫 값을 메모리에서 관리할 시작값, 두 번째 값을 끝값으로 지정
  2. 이후 persist() 호출 시 메모리에서 값을 가져옴
  3. 끝 값이 할당되면 다음 persist() 호출 시 DB 시퀀스를 호출
  4. 이 때, 가져온 시퀀스 값이 새로운 끝값이 되고, 시작값은 새로운 끝값 - (allocationSize - 1)을 지정
<br>

#### 2. Camel case -> Snake case
-  Spring legacy 환경에서 컬럼명 매핑 시 Camel case -> Snake case로 자동 변환하여 매핑하는 방법?
<br>

</details>
<br>

---
## 2023.03.23 목 3회
<!-- <details> -->
<!-- <summary>보기</summary> -->

### 진도
- 섹션 5. 연관관계 매핑 기초
  - 단방향 연관관계
  - 양방향 연관관계와 연관관계의 주인 1-기본
  - 양방향 연관관계와 연관관계의 주인 2-주의점, 정리
  - 실전 예제2 - 연관관계 매핑 시작
  <br>
  
- 섹션 6. 다양한 연관관계 매핑
  - 다대일 [N:1]
  - 일대다 [1:N]
  - 일대일 [1:1]
  - 다대다 [N:N]
  - 실전 예제3 - 다양한 연관관계 매핑
  <br>
  
### ???

<br>

</details>
<br>
