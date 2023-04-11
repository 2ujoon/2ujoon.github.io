[목록으로 가기](./%EC%9E%90%EB%B0%94%20ORM%20%ED%91%9C%EC%A4%80%20JPA%20%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D%20-%20%EA%B8%B0%EB%B3%B8%ED%8E%B8.html)
[강의자료](../../../../../attachments/jpa_basic/02.%20JPA%20%EC%8B%9C%EC%9E%91.pdf)

## **JPA 시작하기**
### Hello JPA - 프로젝트 생성
- DB : H2
- JPA 설정
  - persistentce.xml에 JPA 설정 정보 입력
  - /META-INF/persistentce.xml로 위치가 지정되어 있음
  - 보통 DB 하나 당 persistence-unit 하나를 지정해서 사용
  - 필수 속성 : DB 정보(driver, user, pw, url, dialect 등)
<br>

---
### Hello JPA - 애플리케이션 개발
#### EntityManagerFactory, EntityManager
- EntityManagerFactory(이하 emf) 로부터 EntityManager(이하 em) 를 획득하여 사용
- 사용한 emf, em 는 `close()`로 반환해야 함
  ![](../../../../../attachments/2023-03-13-17-28-29.png)
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
<br>

