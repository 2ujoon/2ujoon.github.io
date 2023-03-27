# 자바 ORM 표준 JPA 프로그래밍 - 기본편
[강의](https://www.inflearn.com/course/ORM-JPA-Basic)

## **강좌 소개**
<!-- <details>
<summary>보기</summary> -->

- 왜 JPA를 사용하는가
  - 기존에는 개발자가 SQL mapper 역할을 함
  - JPA는 SQL을 자동으로 생성해 줌
    - DB에 종속되지 않게 개발할 수 있음
      ∵ 사용하는 DB에 맞게 쿼리를 자동으로 생성(dialect)
<br>

- 실무에서는 테이블의 갯수가 많고, 사용하는 객체가 복잡해 어려울 수 있음; 아래의 연습이 필요
  - 객체와 테이블 설계 매핑
    - 객체, 테이블 설계 및 매핑
    - PK, FK 매핑
    - 1:N, N:1, 1:1, N:M 매핑
  <br>
  
  - JPA 내부 동작 방식 이해
    - JPA가 어떤 SQL을 생성하는가
    - JPA가 언제 SQL을 실행하는가
</details>
<br>

---
## **JPA 소개**
### SQL 중심적인 개발의 문제점
<!-- <details>
<summary>보기</summary> -->

- RDB : 현 DB의 헤게모니
  - RDB에 정보를 저장할 때 객체가 가진 정보를 SQL로 변환하여 RDB에 저장
  - 개발자가 객체와 DB entity를 매핑시킴
  - 객체를 RDB에 저장함에 있어 SQL 의존적 개발을 피하기 어려움
  <br>

- 객체와 RDB의 차이
  1. 상속
    ![](../../../../attachments/2023-03-13-14-40-43.png)
     - 객체의 상속 관계를 RBD는 슈퍼타입 - 서브타입 관계로 풀어냄
     - insert : DB의 두 테이블에 각각 insert
     - select : join 해서 조회 후 각각의 객체를 생성하게 될 수도?
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
</details>
<br>

---
### JPA 소개
<!-- <details>
<summary>보기</summary> -->

- JAVA의 ORM(Object-Relational Mapping) 기술 표준
  - 객체는 객체대로, RDB는 RDB 대로 설계 후 ORM으로 중간에서 매핑
<br>

- 애플리케이션과 JDBC 사이에서 동작
  |      |                                                      |
  | ---- | ---------------------------------------------------- |
  | 동작 | ![](../../../../attachments/2023-03-13-15-58-18.png) |
  | 저장 | ![](../../../../attachments/2023-03-13-15-58-39.png) |
  | 조회 | ![](../../../../attachments/2023-03-13-15-58-57.png) |
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
    ![](../../../../attachments/-%20.png)
    - 지연 로딩 : 객체가 **실제 사용될 때** 로딩
    - 즉시 로딩 : JOIN SQL로 연관된 객체까지 한 번에 **미리 조회**
    - 연관된 객체의 사용빈도를 고려하여 선택

</details>
<br>

---
## **JPA 시작하기**
### Hello JPA - 프로젝트 생성
<!-- <details>
<summary>보기</summary> -->

- DB : H2
- JPA 설정
  - persistentce.xml에 JPA 설정 정보 입력
  - /META-INF/persistentce.xml로 위치가 지정되어 있음
  - 보통 DB 하나 당 persistence-unit 하나를 지정해서 사용
  - 필수 속성 : DB 정보(driver, user, pw, url, dialect 등)

</details>
<br>

---
### Hello JPA - 애플리케이션 개발
<!-- <details>
<summary>보기</summary> -->

#### EntityManagerFactory, EntityManager
- EntityManagerFactory(이하 emf) 로부터 EntityManager(이하 em) 를 획득하여 사용
- 사용한 emf, em 는 `close()`로 반환해야 함
  ![](../../../../attachments/2023-03-13-17-28-29.png)
- `em.getTransaction()`으로 트랜잭션 객체 획득 후 `begin()`으로 트랜잭션 시작
  - `commit()` 또는 `rollback()`으로 트랜잭션을 종료해야 함
<br>

- **!주의**
  - emf는 **하나만 생성**해서 앱 전체에서 공유
  - em은 **쓰레드간 공유하면 안됨**(반드시 반환)
  - JPA의 모든 데이터 변경은 **트랜잭션 안**에서 실행
<br>
  
#### Entity
- 클래스에 `@Entity` 애노테이션으로 JPA가 관리할 객체임을 명시
- 필드에 `@Id` 애노테이션으로 해당 필드를 DB의 PK와 매핑
- `@Table`, `@Column` 등의 애노테이션은 객체의 필드명과 DB의 컬럼명이 다를 경우 name 엘리먼트로 명시
<br>

#### JPQL
- JPA가 제공하는 SQL을 추상화한 쿼리 언어
  ∴ 특정 DB SQL에 의존하지 않음
- SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 지원
- `em.createQuery()` 파라미터로 jpql을 작성해 SQL을 수행할 수 있음
- 검색 쿼리 등에서도 테이블이 아닌 **엔티티 객체를 대상**으로 검색
  - 모든 DB 데이터를 객체로 변환할 수 없음
  - 결국 검색 조건이 포함된 SQL을 사용해야 하므로 JPQL을 사용

</details>
<br>

---
## **영속성 관리 - 내부 동작 방식**
### 영속성 컨텍스트 1
<!-- <details>
<summary>보기</summary> -->

#### **영속성 컨텍스트**
  - 엔티티를 영구 저장하는 환경(논리적 개념)
  - em을 통해서 영속성 컨텍스트에 접근
  - `em.persist(entity);`
<br>

#### 엔티티의 생명주기
  ```
  Member member = new Member();
  member.setId("member1");
  member.setName("memberA");    // 비영속

  em.getTransaction().begin();
  em.persist(member);           // 영속
  em.detach(member);            // 준영속
  ```
   - 비영속 (new / transient)
  - 영속 (managed)
  - 준영속 (detached)
  - 삭제 (removed)

</details>
<br>

---
### 영속성 컨텍스트 2
<!-- <details>
<summary>보기</summary> -->

#### 엔티티 조회, 1차 캐시
  - 1차 캐시

    |    @Id    | Entity |
    | :-------: | :----: |
    | "member1" | member |

  - `em.find(Member.class, "member1");`
    1. 1차 캐시에서 조회 -> 조회 결과 존재
    2. 조회 결과를 반환
    <br>
    
  - `em.find(Member.class, "member2");`
    1. 1차 캐시에서 조회 -> 조회 결과 없음
    2. DB 조회
    3. DB 조회 결과를 1차 캐시에 저장
    4. 조회 결과를 반환
    <br>

  - 1차 캐시의 성능적 이익은 크지 않음
    - em은 보통 트랜잭션 단위로 존재하고, 요청이 종료 될 때 같이 종료; 1차 캐시도 삭제
    - 하나의 트랜잭션 안에서만 이익을 볼 수 있는 구조
    - JPA(또는 Hibernate)에서 애플리케이션 전쳉에서 공유하는 캐시는 2차 캐시라 함
<br>

#### 영속 엔티티의 동일성 보장
```
Member a = em.find(Member.class, "member1");
Member b = em.find(Member.class, "member1");

System.out.println(a == b); // true
```
- 1차 캐시를 이용하여 같은 참조를 가진 객체처럼 비교
- 반복 가능한 읽기(REPEATABLE READ) 등급의 트랜잭션 격리 수준을 DB가 아닌 애플리케이션에서 제공
<br>

#### 엔티티 등록
- 트랜잭션을 지원하는 쓰기 지연
  |                    `em.persist()`                    |                    `tx.commit()`                     |
  | :--------------------------------------------------: | :--------------------------------------------------: |
  | ![](../../../../attachments/2023-03-18-00-29-40.png) | ![](../../../../attachments/2023-03-18-00-31-04.png) |
    - `em.persist()` 때 SQL을 생성해서 쓰기 지연 SQL 저장소에 보관
    - 트랜잭션 commit 직전에 SQL을 flush(DB로 전송)하고 DB에서 commit
    - 예외의 경우가 존재함 (`GenerationType.SEQUENCE)`
    - 버퍼링 가능 : Hibernate의 경우 hibernate.jdbc.batch.size 옵션에서 설정한 만큼 쿼리를 보내고 DB commit
<br>

#### 엔티티 수정
- 변경 감지(dirty checking)
  ![](../../../../attachments/2023-03-18-00-43-08.png)
  - 스냅샷 : 엔티티를 읽어온 최초 시점의 상태를 보관한 것
  - `em.flush()` 할 때(트랜잭션 커밋 시점) 엔티티와 스냅샷을 비교
  - 만약 엔티티와 스냅샷에 차이가 있다면 쓰기 지연 SQL 저장소에 UPDATE 쿼리 추가
<br>

#### 엔티티 삭제
- `em.remove(entity)`로 삭제 
- 같은 방식으로 쓰기 지연 SQL 저장소에 DELETE 쿼리 생성 후 flush

</details>
<br>

---
### 플러시
<!-- <details>
<summary>보기</summary> -->
#### 정의 : 영속성 컨텍스트의 변경 내용을 DB에 반영하는 것
<br>

#### 트랜잭션 commit 시 플러시 발생
- 변경 감지
- 변경 감지한 결과로 수정된 엔티티에 대한 UPDATE 쿼리를 쓰기 지연 SQL 저장소에 등록
- 쓰기 지연 SQL 저장소의 쿼리(등록, 수정, 삭제)를 DB에 전송
<br>

#### 플러시 하는 방법
- `em.flush()`
  - 직접 호출
  - 커밋 전 미리 DB에 반영, 전송되는 쿼리 확인, 테스트 용도로 강제 호출해서 사용
  - 플러시 한다고 해서 1차 캐시가 지워지는 것은 아님
- 트랜잭션 커밋 : 플러시 자동 호출
- JPQL 쿼리 실행
  - 플러시 자동 호출
  - JPQL은 즉시 DB에서 실행되기 때문에 실행 전 영속성 컨텍스트의 상태를 DB에 동기화하기 위함
  - ex) `em.persist(memberC)` 호출 후 JPQL로 전체 회원을 조회한다면, DB에서 INSERT가 실행되어야 memberC의 정보도 조회할 수 있음
<br>

#### 플러시 모드 옵션 : `em.setFlushMode(FlushModeType flushModeType)`
- FlushModeType.AUTO : 커밋이나 쿼리를 실행할 때 플러시(default)
- FlushModeType.COMMIT : 커밋할 때만 플러시
<br>

#### **!주의**
- 플러시는 ***영속성 컨텍스트를 비우지 않음***
- 영속성 컨텍스트의 **변경내용을 DB에 동기화**
- 트랜잭션이라는 작업 단위가 중요하므로, 커밋 직전에만 동기화하면 됨

</details>
<br>

---
### 준영속 상태
<details>
<summary>보기</summary>

#### 준영속 상태로 만드는 방법
- `em.detach(entity)` : 특정 엔티티를 준영속 상태로 전환
- `em.clear()` : 영속성 컨텍스트를 초기화
- `em.close()` : 영속성 컨텍스트 종료
- 테스트 환경에서 사용, 단순한 로직에서는 사실상 필요 없음

</details>
<br>

---
### 정리
#### 걍 정리였음
<br>

---
## **엔티티 매핑**
### 객체와 테이블 매핑
<details>
<summary>보기</summary>

#### 엔티티 매핑 소개
- 객체-테이블 매핑 : `@Entity`, `@Table`
- 필드-컬럼 매핑 : `@Column`
- 기본 키 매핑 : `@Id`
- 연관관계 매핑 : `@ManyToOne`, `@JoinColumn` 등
<br>

#### @Entity
- 클래스에 붙이며, JPA가 엔티티로 관리한다는 의미
- JPA로 테이블과 매핑할 클래스에는 **필수**
- **!주의**
  - **기본 생성자 필수** (protected까지 허용)
  - fianl 클래스, enum, interface, inner 클래스에는 사용하지 않음
  - 저장할 필드에는 final 

</details>
<br>

---
### 데이터베이스 스키마 자동 생성 TODO
<details>
<summary>보기</summary>


</details>
<br>

---
### 필드와 컬럼 매핑 TODO
<details>
<summary>보기</summary>


</details>
<br>

---
### 기본 키 매핑
<!-- <details>
<summary>보기</summary> -->

#### 기본 키 매핑 애노테이션
- `@Id` : `@Id`만 사용 시 값을 직접 할당
- `@GeneratedValue(strategy = GenerationType.AUTO)` : 값 자동 생성
  - IDENTITY : DB에 위임 / MySQL
  - SEQUENCE : DB 시퀀스 사용, `@SequenceGenerator` 필요 / ORACLE
  - TABLE : 키 생성용 테이블 사용, `@TableGenerator` 필요 / 모든 DB 가능
  - AUTO : 방언에 따라 자동 지정, default

#### GenerationType.IDENTITY
- DB에 기본 키 생성 위임
- MySQL, PostgreSQL, SQL Server, DB2 등
  - ex) MySQL의 AUTO_INCREMENT 속성
- 단, AUTO_INCREMENT는 DB에서 INSERT가 실행되어야 ID를 얻을 수 있음
  - `em.persist()`  시점에 즉시 INSERT를 실행시켜 PK 값을 획득

#### GenerationType.SEQUENCE
- DB 오브젝트인 시퀀스 사용
- Oracle, PostgreSQL, DB2, H2 DB 등
- 클래스에 `@SequenceGenerator`를 선언해 매핑
  - ```
    @Entity
    @SequenceGenerator(
          name = "MEMBER_SEQ_GENERATOR",
          sequenceName = "SEQ_MEMBER",
          intitialValue = 1,
          allocationSize = 1)
    public class Member {

      @Id
      @GeneratedValue(strategy = GenerationType.SEQUENCE,
                      generator = "MEMBER_SEQ_GENERATOR")
      private Long id;
    }
    ```
</details>
<br>

---
### 실전 예제 1 - 요구사항 분석과 기본 매핑
<!-- <details>
<summary>보기</summary> -->

#### 기능 목록
- 회원 기능
  - 회원 등록
  - 회원 조회
<br>

- 상품 기능
  - 상품 등록
  - 상품 수정
  - 상품 조회
<br>

- 주문 기능
  - 상품 주문
  - 주문내역 조회
  - 주문 취소
<br>

#### 테이블 설계
![](../../../../attachments/2023-03-17-12-59-57.png)
- ORDERS - ITEM 의 다대다 관계를 중간에 ORDER_ITEM 테이블을 이용해 일대다로 매핑
<br>

#### 엔티티 설계와 매핑
![](../../../../attachments/2023-03-17-13-00-23.png)
<br>

#### 컬럼명 매핑
- Spring legacy 에서는 Camel case를 Snake case로 자동 변환하지 않음
  - 따로 설정해야 함
  - `@Column(name="MEMBER_ID")` 로 직접 매핑할 DB 컬럼을 명시할 수 있음
<br>

#### 엔티티 정보 설정
- 엔티티 클래스에서 제약조건, 인덱스, 컬럼의 크기 등을 설정할 수 있음
- DB를 조회하지 않고 파악할 수 있다는 장점
- 예시
  - 데이터 크기 : `@Column(length=10)` 등
  - 제약조건, 인덱스 : `@Table`의 uniqueConstraints, indexes element(`indexes = @Index()`) 등
<br>

#### 데이터 중심 설계
- 객체 설계를 테이블 설계에 맞춘 방식
  - ex) Order 클래스에 memberId를 필드로 선언
- 객체의 그래프 탐색이 불가능
- 사실상 참조가 없음
- 외래 키가 아닌 객체를 필드로 선언해야 함
<br>

</details>
<br>

---
## 연관관계 매핑 기초
### 단방향 연관관계
<details>
<summary>보기</summary>

#### 용어 이해
- 방향(dirction) : 단방향, 양방향
- 다중성(multiplicity) : 다대일, 일대다, 일대일, 다대다
- **연관관계의 주인(owner)** : 객체 양방향 관계는 관리 주인이 필요
<br>

#### 예제 시나리오
- 회원과 팀이 있음
- 회원은 하나의 팀에만 소속됨
- 회원과 팀은 다대일
<br>

#### 객체를 테이블에 맞추어 모델링(데이터 중심)
![](../../../../attachments/2023-03-23-05-45-52.png)
- 객체에 연관관계가 없음
- Member 객체에 외래 키 값을 그대로 가져옴
- 문제점
  - member를 persist 하기 위해서는 team을 조회해서 id를 가져온 뒤 member에 set 해줘야 함
  - member의 team을 조회하기 위해서는 member의 teamId를 가져와서 다시 team을 조회해야 함
  - 테이블은 외래 키로 조인해서 연관 테이블을 찾고, 객체는 참조를 사용해서 연관된 객체를 찾는 차이가 있기 때문
<br>

#### 객체 지향 모델링(단방향 연관관계)
![](../../../../attachments/2023-03-23-06-32-56.png)
- member 객체에서 team을 바로 가져올 수 있음

</details>
<br>

---
### 양방향 연관관계와 연관관계의 주인 1 - 기본
#### 양방향 매핑
![](../../../../attachments/2023-03-23-07-13-17.png)
- Team <-> Member(List) 양방향으로 객체 그래프 탐색 가능
- Member는 단방향 매핑과 동일하게 작성
- Team에 `List<Member> members` 필드를 추가
  - `@OneToMany(mappedBy = "team")` 애노테이션 필요
<br>

#### 객체와 테이블이 관계를 맺는 차이
- 객체 연관관계 : 2개
  - 회원 -> 팀 (단방향)
  - 팀 -> 회원 (단방향)
  - 객체의 양방향 연관관계는 **사실상 2개의 단방향 관계**
    - 던방향 매핑에서 반대방향으로 조회(객체 그래프 탐색))하는 기능만 추가
  - 객체를 양방향으로 참조하려면 **단방향 연관관계 2개**가 필요
<br>

- 테이블 연관관계 : 1개
  - 회원 <-> 팀 (양방향)
  - **외래 키 하나**로 양방향 연관관계를 가짐
<br>

#### 딜레마 : 둘 중 하나로 외래 키를 관리해야 한다
![](../../../../attachments/2023-03-23-07-30-12.png)
- 회원의 팀 정보가 변경될 때 (= 팀의 회원목록이 바뀔 때)
- DB에서 MEMBER 테이블의 외래 키(TEAM_ID)는 어떤 기준으로 UPDATE 해야 하는가? 
  = Member, Team 객체 중 어디에서 외래 키를 관리해야 하는가?
    1. Member의 team을 변경
    2. Team의 members를 변경
  - 사실 둘 다 가능
<br>

#### 연관관계의 주인(Owner)
- 양방향 매핑 규칙
  - 객체의 두 관계 중 하나를 연관관계의 주인으로 지정
  - 연관관계의 주인만 외래 키를 관리(등록, 수정)
  - 주인이 아닌 쪽은 읽기만 가능
  - 주인은 mappedBy 속성 지정 안함
  - 주인이 아닌 객체는 mappedBy 속성으로 주인을 지정
  <br>

#### 누구를 주인으로? (가이드)
- 외래 키가 있는 곳을 주인으로 정해라(= N:1 관계에서 N 쪽)
- 예시
  - 자동차(Car.wheels) - 바퀴 관계에서 바퀴(Wheel.car)에 주인을 설정
  - 이 경우에는 Member.team이 연관관계의 주인이 됨
  - Team.members를 주인으로 설정한다면?
    - 가능(JPA 짬킹 한정)
    - Team을 다뤘는데 MEMBER 테이블의 UPDATE 쿼리가 나가는 등의 문제의 소지가 있음(성능 이슈)
  <br>

- 연관관계의 주인은 비즈니스적으로 중요한 개념은 아님
<br>

---
### 양방향 연관관계와 연관관계의 주인 2 - 주의점, 정리
<details>
<summary>보기</summary>

#### 연관관계의 주인에 값을 입력하지 않음
```
Team team = new Team();
team.setName("teamA");
team.getMembers().add(member);
em.persist(team);

Member member = new Member();
member.setUsername("member1");
em.persist(member);

em.flush();
em.clear();

tx.commit();
```
- `team.getMembers().add()` 로 Member를 리스트에 추가해도 DB에는 MEMBER의 TEAM_ID에 FK가 들어가지 않음 (1)
- `member.setTeam()` 으로 Member에서 Team을 참조해야 MEMBER테이블에도 정상적으로 반영됨 (2)
    ∵ 연관관계의 주인은 Member이기 때문에, (1)은 DB에 영향을 주지 못함 
  <br>

- (1)을 하지 않더라도 JPA는 지연 로딩을 통해 연관된 Member의 목록을 조회할 수 있음
  - 단, **flush 되지 않고 1차 캐시에서 팀을 가져올 경우** DB에서 조회하지 않기 때문에 추가한 Member가 `getMembers()`에서 조회되지 않을 수 있음
<br>

- 객체 관계를 고려하면 (1), (2) 모두 해주는 것이 맞음
  - 하나만 하고 까먹기 쉽기 때문에 **연관관계 편의 메서드**를 만들어 사용 권장
  - `Team.addMember(Member member)` 또는 `Member.changeTeam(Team team)` 등 상황에 따라 택1
<br>

#### 양방향 매핑 시 무한 루프 주의
- `toString()`, lombok, JSON 생성 라이브러리
- 두 객체가 서로 참조할 때, `toString()`에서 참조하는 객체가 다시 `toString()`을 호출하는 경우
- JSON 생성 라이브러리에서는 **엔티티를 반환하지 말 것**
  - DTO로 변환해서 반환하는 것을 권장
<br>

</details>
<br>

---
### 실전 예제 2 - 연관관계 매핑 시작
<details>
<summary>보기</summary>

#### 테이블 구조
![](../../../../attachments/2023-03-27-03-13-42.png)
<br>

#### 객체 구조
![](../../../../attachments/2023-03-27-03-14-54.png)
- 참조를 사용하도록 변경
<br>

#### 매핑
- 단방향 관계를 우선적으로 매핑, 이후 필요 시 양방향 매핑
- 외래 키를 가진 객체에서 연관관계의 주인을 설정(`@JoinColumn`)
- 외래 키를 필드로 가지지 않고, 객체를 필드로 가짐
<br>

#### 설계
- 회원의 주문 목록을 가져올 경우
  - ORDER에서 MEMBER_ID를 이용해 조회하는 것이 자연스러움
  - Member에서  `List<Order>`로 주문 목록을 가asd져오고 있다면, 관심사를 제대로 끊어내지 못한 것일 수 있음
  <br> 

- 주문은 연관된 상품 목록을 필요로 할 확률이 높음
  - 비즈니스적으로 의미가 클 수 있으므로 양방향 매핑
  <br>

- Member.orders, Order.orderItems는 없어도 문제 없음
  - 단방향으로도 테이블 간 매핑은 완성되었기 때문
  - 개발상의 편의, 일괄 조회 등이 필요할 때 사용(JPQL 등)
<br>

</details>
<br>

---
## 다양한 연관관계 매핑
### 다대일 [N:1]
<details>
<summary>보기</summary>

#### 연관관계 매핑 시 고려사항 3가지
- 다중성
  - `@ManyToOne` / `@OneToMany` / `@OneToOne` / `@ManyToMany`
  - `@ManyToMany`는 거의 쓰이지 않음
<br>

- 단방향, 양방향
  |          테이블          |                     객체                     |
  | :----------------------: | :------------------------------------------: |
  | 외래 키 하나로 양쪽 join |       참조용 필드가 있는 쪽으로만 참조       |
  |      방향 개념 없음      | 한쪽만 참조하면 단방향, 서로 참조하면 양방향 |
<br>

- 연관관계의 주인
  - 외래 키를 관리하는 참조
  - 테이블은 **외래 키 하나**로 두 테이블이 연관관계를 맺음
  - 객체의 양방향 관계는 사실 **단방향 2개**
  - 따라서 참조도 2군데가 있어 둘 중 ***테이블의 외래 키를 관리할 곳을 지정해야함***
  - 주인의 반대편은 외래 키에 영향을 주지 않고, **단순 조회만** 가능
<br>

#### 다대일 단방향
![](../../../../attachments/2023-03-27-04-31-29.png)
- 가장 많이 사용
- 다대일의 반대는 일대다
<br>

#### 다대일 양방향
![](../../../../attachments/2023-03-27-04-33-35.png)
- Team에 `List<Member>` 추가
- 단, DB에는 전혀 영향을 주지 않고, 조회만 가능
- 외래 키가 있는 쪽이 연관관계의 주인
- 양쪽을 서로 참조하도록 개발
<br>

</details>
<br>

---
### 일대다 [1:N]
<details>
<summary>보기</summary>

#### 일대다 단방향
![](../../../../attachments/2023-03-27-04-38-06.png)
- DB에서는 무조건 Many 쪽(MEMBER)에 외래 키가 있음
- Team(One 쪽)에 연관관계의 주인이 설정될 경우
  ```
  Team team = new Team();
  team.getMembers().add(member);
  em.persist(team);
  ```
  - member를 추가하기 위해서 MEMBER 테이블에 UPDATE 쿼리가 추가로 필요
  - Team 엔티티를 다루는데 MEMBER 테이블에 SQL이 실행됨; 의도하지 않은 엔티티에서 변경이 일어날 수 있음
  <br>

- 객체와 테이블의 차이 때문에 반대편 테이블의 외래 키를 관리하는 특이한 구조
- `@JoinColumn`을 반드시 사용해야 함
  - 사용하지 않으면 조인 테이블 방식 사용(중간 테이블 추가)
  <br>

- 권장되지는 않음
  - 엔티티가 관리하는 외래 키가 다른 테이블에 있음
  - 연관관계 관리를 위해 추가로 UPDATE SQL 실행
  - 일대다 단방향 보다는 **다대일 양방향 매핑** 권장
<br>

#### 일대다 양방향
![](../../../../attachments/2023-03-27-06-46-11.png)
- 야매 매핑
- `@JoinColumn(insertable = false, updatable = false)`
- **읽기 전용 필드**를 사용해서 양방향처럼 사용
- **다대일 양방향 매핑** 권장
<br>

</details>유사
<br>

---
### 일대일 [1:1]
<details>
<summary>보기</summary>

#### 일대일 관계
- 주 테이블이나 대상 테이블 중 택1하여 외래 키 관리
- 외래 키에 데이터베이스 유니크 제약조건 추가 
<br>

#### 일대일 : 주 테이블에 외래 키 단방향
![](../../../../attachments/2023-03-27-15-04-06.png)
- 다대일 단방향 매핑과 유사
<br>

#### 일대일 : 주 테이블에 외래 키 양방향
![](../../../../attachments/2023-03-27-15-05-59.png)
- 다대일 양방향 매핑처럼 외래 키가 있는 곳이 연관관계의 주인
- 반대편은 mappedBy 적용
<br>

#### 일대일 : 대상 테이블에 외래 키 단방향
![](../../../../attachments/2023-03-27-15-08-34.png)
- 단방향 관계는 지원하지 않음
- 양방향 관계는 지원
<br>

#### 일대일 : 대상 테이블에 외래 키 양방향
![](../../../../attachments/2023-03-27-15-09-39.png)
- 일대일 주 테이블에 외래 키 양방향 매핑 방법과 동일
<br>

#### 정리
- 주 테이블에 외래 키
  - 주 객체가 대상 객체의 참조를 가짐 == 주 테이블에 외래 키를 두고 대상 테이블을 찾음
  - 객체지향 개발자가 선호
  - JPA 매핑 관리
  - 장점 : 주 테이블만 조회해도 대상 테이블에 데이터가 있는지 확인 가능
  - 단점 : 값이 없으면 외래 키에 null 허용
<br>

- 대상 테이블에 외래 키
  - 대상 테이블에 외래 키가 존재
  - 전통적인 DB 개발자가 선호
  - 장점 : 주 테이블과 대상 테이블을 일대일에서 일대다 관계로 변경할 때 테이블 구조 유지
  - 단점 : 프록시 기능의 한계로 **지연 로딩으로 설정해도 항상 즉시 로딩됨**
<br>

</details>
<br>