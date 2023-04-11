[목록으로 가기](./%EC%9E%90%EB%B0%94%20ORM%20%ED%91%9C%EC%A4%80%20JPA%20%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D%20-%20%EA%B8%B0%EB%B3%B8%ED%8E%B8.html)
[강의자료](../../../../../attachments/jpa_basic/01.%20JPA%20%EC%86%8C%EA%B0%9C.pdf)

## **JPA 소개**
### SQL 중심적인 개발의 문제점
- RDB : 현 DB의 헤게모니
  - RDB에 정보를 저장할 때 객체가 가진 정보를 SQL로 변환하여 RDB에 저장
  - 개발자가 객체와 DB entity를 매핑시킴
  - 객체를 RDB에 저장함에 있어 SQL 의존적 개발을 피하기 어려움
  <br>

- 객체와 RDB의 차이
  1. 상속
    ![](../../../../../attachments/2023-03-13-14-40-43.png)
     - 객체의 상속 관계를 RBD는 슈퍼타입 - 서브타입 관계로 풀어냄
     - insert : DB의 두 테이블에 각각 insert
     - select : join 해서 조회 후 각각의 객체를 생성하게 될 수도🤬
  <br>

  2. 연관관계
     - 객체는 참조, 테이블은 FK를 사용
     - 객체를 DB 테이블에 맞춰 모델링한다면, FK로 사용되는 값을 변수로 가지고 있어야 함
       -> 객체를 다룰 때는 부적절함
         ```
         class Member {
            String id;
            Long teamId;  // TEAM_ID FK컬럼
            String username;
         }
         class Team {
            Long id;
         }
         INSERT INTO MEMBER (MEMBER_ID, TEAM_ID, USERNAME) VALUES...
         ``` 
         <br>

     - 참조 형식으로 모델링한다면, 쿼리 시 파라미터 매핑 등에서 굉장히 번거로워질 수도 있음
         ```
         class Member {
            String id;
            Team team;  // 참조를 통한 연관관계
            String username;
         }
         class Team {
            Long id;
         }
         ``` 
        <br>

     - 엔티티 신뢰 문제
       - 객체는 자유롭게 내/외부의 객체 등으로 탐색할 수 있어야 함
       - 처음 실행하는 SQL에 따라 탐색 범위가 결정됨
         ```
         SELECT M.*, T.*
           FROM MEMBER M
           JOIN TEAM T
             ON M.TEAM_ID = T.TEAM_ID;

         member.getTeam();  // ok
         member.getOrder(); // NPE
         ```
       - 다음 계층의 데이터를 믿을 수 없으면 직접 들어가서 확인해 봐야 함
       - 그렇다고 모든 객체를 미리 로딩할 수는 없음
         ∵ 불필요한 데이터를 조회하며 쿼리 성능을 크게 떨어뜨림
       - 상황에 따른 조회 메서드를 여러 벌 만들게 됨
  <br>

  3. 데이터 타입
  4. 데이터 식별 방법
      ```
      Long memberId = 1;
      Member member1 = dao.getMember(memberId);
      Member member2 = dao.getMember(memberId);
      member1 == member2  // false
      ```
    <br>

- 객체답게 모델링 할수록 매핑 작업이 늘어남
- 객체를 JAVA 컬렉션에 저장하듯이 DB에 저장하면 해결할 수 있음 => JPA
<br>

---
### JPA 소개
- JAVA의 ORM(Object-Relational Mapping) 기술 표준
  - 객체는 객체대로, RDB는 RDB 대로 설계 후 ORM으로 중간에서 매핑
<br>

- 애플리케이션과 JDBC 사이에서 동작
  |      |                                                      |
  | ---- | ---------------------------------------------------- |
  | 동작 | ![](../../../../../attachments/2023-03-13-15-58-18.png) |
  | 저장 | ![](../../../../../attachments/2023-03-13-15-58-39.png) |
  | 조회 | ![](../../../../../attachments/2023-03-13-15-58-57.png) |
<br>

- SQL 중심의 개발에서 객체 중심으로 개발 가능
  - 벤더 독립성
  - 객체 RDB의 패러다임 불일치 해소
    - 상속, 연관관계, 객체 그래프 탐색, 비교 등
    ∵ 개발자가 jpa 메서드를 사용하면, 그에 맞게 JPA가 SQL을 작성하기 때문
<br>

- CRUD
  - 저장 : em.**persist**(member);
  - 조회 : Member member = em.**find**(memberId);
  - 수정 : member.**setName**("memberA");
  - 삭제 : em.**remove**(member);
<br>

- 유지보수에 용이
  - 컬럼이 변경되더라도 객체의 필드만 수정하면 되고, SQL은 따로 처리가 필요하지 않음
<br>

- 신뢰할 수 있는 데이터 계층
  - **지연 로딩**을 이용한 자유로운 객체 그래프 탐색이 가능
<br>

- 동일 트랜잭션 내에서 조회한 엔티티는 같음을 보장 (≒ Java 컬렉션)
- 성능 최적화
  - 1차 캐시 동일성 보장
    - 같은 트랜잭션 내에서 같은 객체 반환(캐싱)
    - 1번째 조회는 쿼리를 날리고, 다음부터는 메모리에서 가져옴
  <br>

  - 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)
    - **트랜잭션 커밋 시점**까지 INSERT SQL을 모아둠
    - JDBC Batch SQL 기능을 사용해 **한 번에 전송**
    - UPDATE, DELETE의 경우, 트랜잭션 커밋 시 SQL을 실행하고 커밋
    - 이 때, 트랜잭션 커밋 전까지 DB에 **Row Lock이 걸리지 않음**
  <br>

  - 지연 로딩(Lazy)과 즉시 로딩(Eager)
    ![](../../../../../attachments/-%20.png)
    - 지연 로딩 : 객체가 **실제 사용될 때** 로딩
    - 즉시 로딩 : JOIN SQL로 연관된 객체까지 한 번에 **미리 조회**
    - 연관된 객체의 사용빈도를 고려하여 선택
    <br>

