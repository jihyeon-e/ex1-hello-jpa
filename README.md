# 1. JPA 소개

### 발전 방향

- 순수 JDBC -> JdbcTemplate -> JPA
- 과거에는 객체를 데이터베이스에 저장,조회를 하려면 복잡한 JDBC API 와 sql문을 한땀한땀 직접 작성해야 했다.
- JDBC Template이나 mybatis가 등장하면서 코드는 짧아졌지만 개발자가 sql문을 직접 작성해야 했다.
- JPA는 sql 조차도 작성할 필요가 없다. JPA를 이용하면 개발자 대신에 적절한 sql을 생성하고, 데이터베이스에 실행해서 객체를 저장하거나 불러오게 된다.
- JPA를 사용하면 개발 생산성, 개발 속도, 유지보수 측면에서도 확연히 차이가 난다.

### JPA 실무에서 어려운 이유

- 처음 JPA이나 스프링 데이터 JPA를 만나면 SQL가 자동화되고, 수십줄의 코드가 한, 두 줄로 되기 때문에 편할거 같지만 실무에 바로 도입하기 어렵다.
- 실무는 수십 개 이상의 복잡한 객체와 테이블 사용
- 객체와 테이블을 잘 설계하고 올바르게 매핑해야 한다.

### 강의 목표

**1. 객체와 테이블 설계 매핑**

- 객체와 테이블을 제대로 설계하고 매핑하는 방법
- 기본 키와 외래 키 매핑
- 1:N, N:1, 1:1, N:M 매핑
- 실무 노하우 + 성능까지 고려
- 어떠한 복잡한 시스템도 JPA로 설계 가능

**2. JPA 내부 동작 방식 이해**

- JPA의 내부 동작 방식을 이해하고 사용
- JPA 내부 동작 방식을 그림과 코드로 자세히 설명
- JPA가 어떤 SQL을 만들어 내는지 이해
- JPA가 언제 SQL을 실행하는지 이해

# 1. SQL 중심적인 개발의 문제점

## 1) SQL을 직접 다룰 때 발생하는 문제점

- 진정한 의미의 계층 분할이 어렵다.
- 엔티티를 신뢰할 수 없다.
- SQL에 의존적인 개발을 피하기 어렵다.

### (1)  무한 반복, 지루한 코드

- CRUD용 SQL을 반복해서 작성해야 한다.
- 데이터베이스는 객체 구조와는 다른 데이터 중심의 구조를 가지므로 객체를 데이터베이스에 직접 저장하거나 조회할 수 없다.
- 개발자가 객체지향 애플리케이션과 데이터베이스 중간에서 SQL과 JDBC API를 사용해 변환 작업을 해주어야 한다.

### (2) SQL에 의존적인 개발

- 객체를 데이터베이스에 저장함으로써 수정 요구사항이 있을 때 많은 코드를 수정해야 한다.

## 2) 패러다임의 불일치

애플리케이션을 자바라는 객체지향 언어로 개발하고 데이터는 관계형 데이터베이스에 저장해야 한다면, 패러다임 불일치 문제를 개발자가 중간에서 해결해야 한다.

**객체**

- 객체 지향 프로그래밍은 추상화, 캡슐화, 정보은닉, 상속, 다형성 등의 시스템 복잡성을 제어할 수 있는 다양한 장치들을 제공한다. 그래서 현대의 복잡한 애플리케이션은 대부분 객체지향 언어로 개발한다.
- 객체가 단순하면 객체의 모든 속성 값을 꺼내서 파일이나 데이터베이스에 저장하면 되지만, 부모 객체를 상속받았거나, 다른 객체를 참조하고 있다면 객체의 상태를 저장하기 쉽지 않다.
    - 예를 들어 회원 객체를 저장해야 하는데 회원 객체가 팀 객체를 참조하고 있다면, 회원 객체를 저장할 때 팀 객체도 함께 저장해야 한다.

**관계형 데이터베이스**

- 관계형 데이터베이스는 데이터 중심으로 구조화되어 있고, 집합적인 사고를 요구한다. 그리고 객체지향에서 이야기하는 추상화, 상속, 다형성 같은 개념이 없다. 따라서 객체 구조를 테이블 구조에 저장하는 데는 한계가 있다.

지금부터 패러다임의 불일치로 인해 발생하는 문제점과 JPA를 통한 해결책을 알아보자.

### (1) 상속

- 만약 해당 객체들을 데이터베이스가 아닌 자바 컬렉션에 보관한다면 다음 같이 부모 자식이나 타입에 대한 고민없이 컬렉션을 그냥 사용하면 된다.

```java
list.add(album);
list.add(movie);

Album album = list.get(albumId);
//부모 타입으로 조회 후 다형성을 활용할 수도 있다.
Item item = list.get(albumId);
```

- DB모델링에서는 객체를 나누어, 각 SQL을 생성하여 만들어야 한다.

### (2) 연관관계

![https://i.imgur.com/6Erg2nI.png](https://i.imgur.com/6Erg2nI.png)

- 객체는 참조를 사용해서 연관된 객체를 조회하고, 테이블은 외래 키를 사용해서 조인으로 연관된 테이블을 조회한다.
- 객체는 한방향으로 밖에 조회할 수 없지만, 테이블의 경우 외래키만 존재한다면 양 테이블에 접근이 가능하다.

### (3) 객체 그래프 탐색

- SQL을 직접 다루면 처음 실행하는 SQL에 따라 객체 그래프를 어디까지 탐색할 수 있는지 정해진다.

```java
class MemberService {
	
		public void process() {
			Member member = memberDAO.find(memberId);
			member.getTeam();
			member.getOrder().getDelivery();
		}
}
```

- 위의 예제에서 member 객체와 연관된 Team, Order, Delivery 방향으로 객체 그래프를 탐색할 수 있는지는 이 코드만 보고는 전혀 알 수 없다. SQL문까지 직접 확인해야 한다. 이것은 엔티티 신뢰 문제가 발생할 수 있다.
- 진정한 의미의 계층 분할이 어렵다.
- 그렇다고 모든 객체를 미리 로딩할 수 없다.

### (4) 비교하기

- 기본 키 값이 같은 회원 객체를 같은 데이터베이스 로우에서 조회했지만, 객체 측면에서 볼 때 둘은 다른 인스턴스이기 때문에 false가 반환된다.
- memberDAO.getMember() 를 호출할 때 마다 new Memver() 로 인스턴스가 새로 생성된다.

```java
String memberId = "100";
Member member1 = memberDAO.getMember(memberId);
Member member2 = memberDAO.getMember(memberId);

member1 == member2; //다르다.

class MemberDAO {
	
		public Member getMember(String memberId) {
			String sql = "SELECT * FROM MEMBER WHERE MEMBER_ID = ?";
			...
			//JDBC API, SQL 실행
			return new Member(...);
		}
}
```

- 객체를 컬렉션에 보관했다면 true가 반환된다.

```java
String memberId = "100";
Member member1 = list.get(memberId);
Member member2 = list.get(memberId);

member1 == member2; //같다.
```

### (5) 정리

- 객체 모델과 관계형 데이터베이스 모델은 지향하는 패러다임이 다르기 때문에 이 차이를 극복하기 위해서는 너무 많은 시간과 코드를 소비해야 한다.
- 더 어려운 문제는 객체지향 애플리케이션 답게 정교한 객체 모델링을 할수록 패러다임의 불일치 문제가 더 커져서 결국 객체 모델링은 힘을 잃고 점점 데이터 중심의 모델로 변해간다.
- 객체를 자바 컬렉션에 저장 하듯이 DB에 저장할 수 없을까?
- JPA는 패러다임의 불일치 문제를 해결해주고 정교한 객체 모델링을 유지하게 도와준다.

# 2. JPA란?

- JPA(Java Persistence API)는 자바 진영의 ORM 기술 표준이다.
    - ORM이란?
        - Object-relational mapping(객체와 관계형 데이터베이스를 매핑)
            - 객체와 RDB(관계형 DB) 라는 두 기둥위에 있는 기술
        - 객체는 객체대로 설계하고 관계형 데이터베이스는 관계형 데이터베이스대로 설계하여 ORM 프레임워크가 중간에서 매핑한다.
        - 대중적인 언어에는 대부분 ORM 기술이 존재
- JPA는 애플리케이션과 JDBC 사이에서 동작한다.

  ![https://i.imgur.com/rSIgWYe.png](https://i.imgur.com/rSIgWYe.png)

- JPA를 사용하여객체를 데이터베이스에 저장
    - Entity 분석
    - INSERT SQL 생성
    - JDBC API 사용
    - 패러다임 불일치 해결
    - 코드

        ```java
        jpa.persist(member);
        ```

- 객체를 조회할 때도 JPA를 통해 객체를 직접 조회
    - SELECT SQL 생성
    - JDBC API 사용
    - ResultSet 매핑
    - 패러다임 불일치 해결
    - 코드

        ```java
        Member member = jpa.find(memberId);
        ```

  ## 1) JPA 소개

  ### (1) 역사

  EJB - 엔티티 빈(자바 표준) → 하이버네이트 (오픈소스) → JPA (자바 표준)

  ### (2) JPA는 표준 명세

    - JPA는 인터페이스의 모음
    - JPA 2.1 표준 명세를 구현한 3가지 구현체
    - 하이버네이트, EclipseLink, DataNucleus

  ### (3) JPA 버전별 특징

    - JPA 1.0(JSR 220) 2006년 : 초기 버전. 복합 키와 연관관계 기능이 부족
    - JPA 2.0(JSR 317) 2009년 : 대부분의 ORM 기능을 포함, JPA Criteria 추가
    - JPA 2.1(JSR 338) 2013년 : 스토어드 프로시저 접근, 컨버터(Converter), 엔티
      티 그래프 기능이 추가

  ## 2) 왜 JPA를 사용해야 하는가?

    - SQL 중심적인 개발에서 객체 중심으로 개발
    - 생산성
    - 유지보수
    - 패러다임의 불일치 해결
    - 성능
    - 데이터 접근 추상화와 벤더 독립성
    - 표준

  ### (1) 생산성

  SQL을 작성하고 JDBC API를 사용하는 지루하고 반복적인 일은 JPA가 대신 처리해준다.

  **JPA와 CRUD**

    - 저장: **jpa.persist**(member)
    - 조회: Member member = **jpa.find**(memberId)
    - 수정: **member.setName**(“변경할 이름”)
    - 삭제: **jpa.remove**(member)

  ### (2) 유지보수

    - 기존에는 엔티티에 필드를 하나만 추가해도 관련된 CRUD SQL과 결과를 매핑하기 위한 JDBC API 코드를 모두 변경해야 했다.
    - JPA가 이런 과정을 대신 처리해주므로 유지보수해야 하는 코드 수가 줄어든다.

  ### (3) 패러다임의 불일치 해결

    - JPA는 상속, 연관관계, 객체 그래프 탐색, 비교하기와 같은 패러다임의 불일치 문제를 해결해준다.
        - 상속
            - 마치 자바 컬렉션에 객체를 저장하듯이 JPA에게 객체를 저장하면 된다.

              JPA는 객체를 ITEM, ALBUM 두 테이블에 나누어 저장한다.

            ```java
            jpa.persist(album);
            ```

            - 객체를 조회할 때는 find 메소드를 사용한다.

              JPA는 ITEM과 ALBUM 두 테이블을 조인해서 필요한 데이터를 조회하고 그 결과를 반환한다.

            ```java
            String albumId = "id100";
            Album album = jpa.find(Album.class, albumId);
            ```

        - 연관관계
            - 회원과 팀의 관계를 설정하고 회원 객체를 저장하면 된다. JPA는 team의 참조를 외래 키로 변환해서 적절한 INSERT SQL을 데이터베이스에 전달한다.

            ```java
            member.setTeam(team); //회원과 팀 연관관계 설정
            jpa.persist(member); //회원과 연관관계 함께 저장
            ```

            - 객체를 조회할 때 외래 키를 참조로 변환하는 일도 JPA가 처리해준다.

            ```java
            Member member = jpa.find(Member.class, memberId);
            Team team = member.getTeam();
            ```

        - 객체 그래프 탐색
            - JPA는 연관된 객체를 사용하는 시점에 적절한 SELECT SQL을 실행하기 때문에 연관된 객체를 신뢰하고 마음껏 조회할 수 있다. → 지연 로딩

            ```java
            class MemberService {
            		...
            		public void process() {
            				Member member = memberDAO.find(memberId);
            				member.getTeam(); //자유로운 객체 그래프 탐색
            				member.getOrder().getDelivery();
            		}
            }
            ```

        - 비교
            - JPA는 같은 트랜잭션일 때 같은 객체가 조회되는 것을 보장한다.

            ```java
            String memberId = "100";
            Member member1 = jpa.find(Member.class, memberId);
            Member member2 = jpa.find(Member.class, memberId);

            member1 == member2; //같다.
            ```

  ### (4) 성능

    - 1차 캐시와 동일성(identity) 보장, SQL 1번만 실행

        ```java
        String memberId = "100";
        Member member1 = jpa.find(Member.class, memberId); //SQL
        Member member2 = jpa.find(Member.class, memberId); //캐시

        member1 == member2; //같다.
        ```

        - 같은 트랜잭션 안에서는 같은 엔티티를 반환 - 아주 약간의 성능 향상(크게 도움은 안됨)
        - DB Isolation Level이 Read Commit이어도 애플리케이션에서 Repeatable Read 보장
    - 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)
        - 트랜잭션을 커밋할 때까지 INSERT SQL을 모음
        - JDBC BATCH SQL 기능을 사용해서 한번에 SQL 전송

        ```java
        transaction.begin(); // [트랜잭션] 시작

        em.persist(memberA);
        em.persist(memberB);
        em.persist(memberC);
        //여기까지 INSERT SQL을 데이터베이스에 보내지 않는다.

        //커밋하는 순간 데이터베이스에 INSERT SQL을 모아서 보낸다.
        transaction.commit(); // [트랜잭션] 커밋
        ```

    - 지연 로딩(Lazy Loading)과 즉시 로딩
        - 지연 로딩 : 객체가 실제 사용될 때 로딩
        - 즉시 로딩 : JOIN SQL로 한번에 연관된 객체까지 미리 조회
        - 두 가지 로딩을 옵션 설정으로 바꿀 수 있다.

          ![https://i.imgur.com/7osczKZ.png](https://i.imgur.com/7osczKZ.png)


# 2. JPA 시작하기

# 프로젝트 생성

# 1. H2 데이터베이스 설치와 실행

- 최고의 실습용 DB
- 가볍다.(1.5M)
- 웹용 쿼리툴 제공MySQL, Oracle 데이터베이스 시뮬레이션 기능
- 시퀀스, AUTO INCREMENT 기능 지원
- 터미널창에 입력하기

```
brew install h2
h2 -web
```

- 터미널에 실행된 url 확인

![https://i.imgur.com/8p1HOmm.png](https://i.imgur.com/8p1HOmm.png)

- [http://localhost:8082/login.do](http://localhost:8082/login.do) 주소 입력하기
- 서버 모드로 변경하기

![https://i.imgur.com/PZNM4Wr.png](https://i.imgur.com/PZNM4Wr.png)

![https://i.imgur.com/vvPJX4z.png](https://i.imgur.com/vvPJX4z.png)

# 2. 라이브러리와 프로젝트 구조

**maven으로 선택하기**

![https://i.imgur.com/PENr0EH.png](https://i.imgur.com/PENr0EH.png)

![https://i.imgur.com/DU4Q42u.png](https://i.imgur.com/DU4Q42u.png)

**pom.xml에 라이브러리 추가**

- 라이브러리 버전 선택 시
    - [https://spring.io](https://spring.io) → Projects → Spring Boot → Learn에서 내가 사용할 스프링 부트 버전을 보고  Reference Doc 문서 → Dependency Versions에서 버전 검색

```xml
<dependencies>
    <!-- JPA 하이버네이트 -->
    <dependency>
        <groupId>org.hibernate</groupId>
        <artifactId>hibernate-entitymanager</artifactId>
        <version>5.3.10.Final</version>
    </dependency>
    <!-- H2 데이터베이스 -->
    <dependency>
        <groupId>com.h2database</groupId>
        <artifactId>h2</artifactId>
        <version>1.4.200</version>
    </dependency>
</dependencies>
```

dependencies에 들어온 것을 확인할 수 있다.

![https://i.imgur.com/zWiKj3F.png](https://i.imgur.com/zWiKj3F.png)

- hibernate-core:5.3.10.Final
    - hibernate에 꼭 필요한 라이브러리
- javax.persistence-api:2.2
    - 인터페이스인 JPA의 구현체로 hibernate 선택했는데, 앞으로 사용할 JPA 인터페이스가 모아져있다.

# 3. persistence.xml 설정하기

- 위치 resources/META-INF/persistence.xml

![https://i.imgur.com/XDflLkc.png](https://i.imgur.com/XDflLkc.png)

- persistence-unit name으로 이름 지정
- javax.persistence로 시작 : JPA 표준 속성
- hibernate.dialect : 하이버네이트 속성, 데이터베이스 방언 설정
    - H2 : org.hibernate.dialect.H2Dialect
    - Oracle 10g : org.hibernate.dialect.Oracle10gDialect
    - MySQL : org.hibernate.dialect.MySQL5InnoDBDialect

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">
    <persistence-unit name="hello">
        <properties>
            <!-- 필수 속성 -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:~/test"/>
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <!-- 옵션 -->
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <!--<property name="hibernate.hbm2ddl.auto" value="create" />-->
        </properties>
    </persistence-unit>
</persistence>
```

### 데이터베이스 방언

- 방언: SQL 표준을 지키지 않는 특정 데이터베이스만의 고유한 기능
- JPA는 특정 데이터베이스에 종속 X
- 각각의 데이터베이스가 제공하는 SQL 문법과 함수는 조금씩 다름
    - 가변 문자: MySQL은 VARCHAR, Oracle은 VARCHAR2
    - 문자열을 자르는 함수: SQL 표준은 SUBSTRING(), Oracle은 SUBSTR()
    - 페이징: MySQL은 LIMIT , Oracle은 ROWNUM

![https://i.imgur.com/kuL3CWs.png](https://i.imgur.com/kuL3CWs.png)

# 4. 애플리케이션 개발

## 1) JPA 구동 방식

![https://i.imgur.com/oyR9PQk.png](https://i.imgur.com/oyR9PQk.png)

JPA는 Persistence 라는 class에서 시작을 하는데, 우리가 설정해준 persistence.xml을 읽어서 EntityManagerFactory 라는 class를 만든다. 그리고 필요할 때마다 EntityManager를 찍어낸다.

### (1) INSERT

**JpaMain**

아래와 같이 작성하는 것이 정석이지만 스프링이 전부 해준다.

```java
public class JpaMain {

    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

        EntityManager em = emf.createEntityManager();

        EntityTransaction tx = em.getTransaction();
        tx.begin();

        try {
            Member member = new Member();
            member.setId(2L);
            member.setName("HelloB");

            em.persist(member);

            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
        }
        emf.close();
    }
}
```

**table 생성**

![https://i.imgur.com/Mnc1AES.png](https://i.imgur.com/Mnc1AES.png)

**Member**

```java
@Entity
public class Member {

    @Id
    private Long id;
    private String name;

    public Long getId() {
        return id;
    }

    public void setId(Long id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

결과 화면

![https://i.imgur.com/hxqIrEK.png](https://i.imgur.com/hxqIrEK.png)

persistence.xml에서 설정해준 옵션들 때문에 쿼리가 콘솔창에 출력된다.

```java
<property name="hibernate.show_sql" value="true"/> // 쿼리 출력
<property name="hibernate.format_sql" value="true"/> // 보기좋게
<property name="hibernate.use_sql_comments" value="true"/> /* insert hellojpa.Member*/
```

DB에 저장된 모습

![https://i.imgur.com/a9VhWnv.png](https://i.imgur.com/a9VhWnv.png)

- 작성한 코드와 테이블 정보가 다를 경우
    - @Table(name = "MEMBER")
    - @Column(name = "name")

    ```java
    @Entity
    @Table(name = "MEMBER")
    public class Member {

        @Id
        private Long id;

        @Column(name = "name")
        private String name;

        public Long getId() {
            return id;
        }

        public void setId(Long id) {
            this.id = id;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }
    ```

### (2) SELECT

**JpaMain**

```java
Member findMember = em.find(Member.class, 1L);
```

결과 화면

![https://i.imgur.com/iNL01d3.png](https://i.imgur.com/iNL01d3.png)

### (3) DELETE

**JpaMain**

```java
Member findMember = em.find(Member.class, 2L);

em.remove(findMember);
```

결과 화면

![https://i.imgur.com/Bhse8oJ.png](https://i.imgur.com/Bhse8oJ.png)

테이블을 확인하면 삭제된 것을 볼 수 있다.

![https://i.imgur.com/L40wJWn.png](https://i.imgur.com/L40wJWn.png)

### (4) UPDATE

**JpaMain**

```java
Member findMember = em.find(Member.class, 1L);
findMember.setName("HelloJPA");
```

결과 화면

![https://i.imgur.com/sArXl9X.png](https://i.imgur.com/sArXl9X.png)

테이블을 확인하면 수정된 것을 볼 수 있다.

![https://i.imgur.com/aGgsuUJ.png](https://i.imgur.com/aGgsuUJ.png)

- 엔티티를 수정한 후에 수정 내용을 반영하려면 em.update() 같은 메소드(없음)를 호출해야 할 것 같지만 단순히 엔티티의 값만 변경하면 된다.
- JPA는 어떤 엔티티가  변경되었는지 추적하는 기능을 갖고 있다.
- 따라서 findMember.setName("HelloJPA")처럼 엔티티의 값만 변경하면 UPDATE SQL을 생성해서 데이터베이스에 값을 변경한다.

## 2) 주의해야 할 점

- 엔티티 매니저 팩토리는 하나만 생성해서 애플리케이션 전체에서 공유
- 엔티티 매니저는 쓰레드간에 공유X (사용하고 버려야 한다)
- JPA의 모든 데이터 변경은 트랜잭션 안에서 실행

## 3) JPQL

하나 이상의 회원 목록을 조회하려면?

- JPQL로 전체 회원 검색
    - Member 객체가 대상이 된다.

    ```java
    List<Member> result = em.createQuery("select m from Member", Member.class)
                        .getResultList();
    ```

![https://i.imgur.com/5p5GnYL.png](https://i.imgur.com/5p5GnYL.png)

- JPQL로 ID가 2 이상인 회원만 검색
- JPQL로 이름이 같은 회원만 검색
- 페이징 처리 메소드

    ```java
    List<Member> result = em.createQuery("select m from Member as m", Member.class)
            .setFirstResult(5)
            .setMaxResults(8)
            .getResultList();
    ```

- JPA는 SQL을 추상화한 JPQL이라는 객체 지향 쿼리 언어 제공
- SQL과 문법 유사, SELECT, FROM, WHERE, GROUP BY, HAVING, JOIN 지원
- JPQL은 엔티티 객체를 대상으로 쿼리
- SQL은 데이터베이스 테이블을 대상으로 쿼리
- 테이블이 아닌 객체를 대상으로 검색하는 객체 지향 쿼리
- SQL을 추상화해서 특정 데이터베이스 SQL에 의존X
- JPQL을 한마디로 정의하면 객체 지향 SQL

# 3. 영속성 관리 - 내부 동작 방식

### JPA에서 가장 중요한 2가지

- 객체와 관계형 데이터베이스 매핑하기(Object Relational Mapping)
- 영속성 컨텍스트(JPA 내부 동작 방식)

# 1. 엔티티 매니저 팩토리와 엔티티 매니저

![https://i.imgur.com/nCz85Wu.png](https://i.imgur.com/nCz85Wu.png)

- 엔티티 매니저 팩토리
    - 엔티티 매니저를 만드는 공장이다.
    - 고객의 요청이 올 때마다 엔티티 매니저를 생성한다.
    - 여러 스레드가 동시에 접근해도 안전하므로 서로 다른 스레드 간에 공유해도 된다.
- 엔티티 매니저
    - 내부적으로 데이터베이스 커넥션을 사용해서 DB를 사용한다.
    - 여러 스레드가 동시에 접근하면 동시성 문제가 발생하므로 스레드 간에 절대 공유하면 안 된다.

# 2. **영속성 컨텍스트**

- 엔티티를 영구 저장하는 환경이라는 뜻. 논리적인 개념
- EntityManager.persist(entity);
    - 객체를 DB에 저장한다는 뜻이 아니라, 영속성 컨텍스트를 통해서 엔티티를 영속화한다는 뜻이다.
    - 더 정확하게는 엔티티를 영속성 컨텍스트에 저장한다.
- 엔티티 매니저를 생성할 때 하나 만들어진다.
- 엔티티 매니저를 통해서 영속성 컨텍스트에 접근한다.

# 3. 엔티티의 생명주기

![https://i.imgur.com/KKamXt4.png](https://i.imgur.com/KKamXt4.png)

## 1) 비영속(new/transient)

영속성 컨텍스트와 전혀 관계가 없는 **새로운** 상태

![https://i.imgur.com/PsEQYyx.png](https://i.imgur.com/PsEQYyx.png)

```java
//객체를 생성한 상태(비영속)
Member member = new Member();
member.setId("member1");
member.setUsername("회원1");
```

## 2) 영속(managed)

영속성 컨텍스트에 **관리되는** 상태

![https://i.imgur.com/dhIxZSs.png](https://i.imgur.com/dhIxZSs.png)

```java
//객체를 생성한 상태(비영속)
Member member = new Member();
member.setId("member1");
member.setUsername(“회원1”);

EntityManager em = emf.createEntityManager();
em.getTransaction().begin();

//객체를 저장한 상태(영속)
em.persist(member);
```

- em.persist(member)을 할 때는 DB에 저장되지 않는다.

```java
public class JpaMain {

    public static void main(String[] args) {
        EntityManagerFactory emf = Persistence.createEntityManagerFactory("hello");

        EntityManager em = emf.createEntityManager();

        EntityTransaction tx = em.getTransaction();
        tx.begin();

        try {
            //비영속
            Member member = new Member();
            member.setId(100L);
            member.setName("HelloJPA");

            //영속
            System.out.println("===BEFORE===");
            em.persist(member);
            System.out.println("===AFTER===");

            tx.commit();
        } catch (Exception e) {
            tx.rollback();
        } finally {
            em.close();
        }
        emf.close();
    }
}
```

실행하면 ===AFTER=== 다음에 INSERT 쿼리가 처리된다.

![https://i.imgur.com/bX9y1xx.png](https://i.imgur.com/bX9y1xx.png)

- 영속상태가 된다고 해서 바로 DB에 쿼리가 반영되는 것이 아니라, 트랜잭션을 커밋하는 순간에 DB에 쿼리가 반영된다.

## 3) 준영속(detached)

영속성 컨텍스트에 저장되었다가 **분리**된 상태

```java
//회원 엔티티를 영속성 컨텍스트에서 분리, 준영속 상태
em.detach(member);
```

## 4) 삭제(removed)

데이터베이스에서 **삭제한** 상태

```java
//객체를 삭제한 상태(삭제)
 em.remove(member);
```

# 4. 영속성 컨텍스트의 특징

### 영속성 컨텍스트와 식별자 값

- 영속성 컨텍스트는 엔티티를 식별자 값으로 구분한다.
- 따라서 영속 상태는 식별자 값이 반드시 있어야 한다.

### 영속성 컨텍스트와 데이터베이스 저장

- JPA는 트랜잭션을 커밋하는 순간 영속성 컨텍스트에 새로 저장된 엔티티를 데이터베이스에 반영하는데, 이를 플러시(flush)라 한다.

### 영속성 컨텍스트의 장점

- 1차 캐시
- 동일성(identity) 보장
- 트랜잭션을 지원하는 쓰기 지연(transactional write-behind)
- 변경 감지(Dirty Checking)
- 지연 로딩(Lazy Loading)

## 1) 엔티티 조회, 1차 캐시

- 영속성 컨텍스트는 내부에 캐시를 가지고 있는데 이것을 1차 캐시라 한다.

![https://i.imgur.com/XrczKVV.png](https://i.imgur.com/XrczKVV.png)

```java
//객체를 생성한 상태(비영속)
Member member = new Member();
member.setId("member1");
member.setUsername(“회원1”);

//객체를 저장한 상태(영속)
em.persist(member);
```

- 위의 코드를 실행하면 회원 엔티티를 데이터베이스가 아닌 1차 캐시에 저장한다.
- 1차 캐시의 키는 식별자 값이다.

### **엔티티 조회**

```java
Member findMember = em.find(Member.class, "member1");

//EntityManager.find() 메소드 정의
public <T> find(Class<T> entityClass, Object primaryKey);
```

- find() 메소드를 보면 첫 번째 파라미터는 엔티티 클래스의 타입이고, 두 번째는 조회할 엔티티의 식별자 값이다.
- 조회할 엔티티의 식별자 값은 primaryKey만 가능하다.

  ![https://i.imgur.com/QBFbmIp.png](https://i.imgur.com/QBFbmIp.png)

```java
Member findMember2 = em.find(Member.class, "member2");
```

- em.find()를 호출하면 먼저 1차 캐시에서 엔티티를 찾고 만약 찾는 엔티티가 1차 캐시에 없으면 데이터베이스에서 조회한다.
- member2가 1차 캐시에 없으므로 엔티티를 생성해서 1차 캐시에 저장한다.(영속 상태가 됨)
- 조회한 엔티티를 반환한다.

직접 코드를 통해 확인해보자.

```java

//비영속
Member member = new Member();
member.setId(101L);
member.setName("HelloJPA");

//영속
System.out.println("===BEFORE===");
em.persist(member);
System.out.println("===AFTER===");

Member findMember = em.find(Member.class, 101L);

System.out.println("findMember.id = " + findMember.getId());
System.out.println("findMember.name = " + findMember.getName());

tx.commit();
```

실행한 결과

![https://i.imgur.com/hMyCsqa.png](https://i.imgur.com/hMyCsqa.png)

SELECT 쿼리가 처리되지 않았지만 id와 name 값이 출력됐다. → em.persist() 를 통해 1차 캐시에 저장이 되고, 그 값을 불러왔기 때문이다.

만약 같은 값을 두 번 불러올 때는 처음에는 데이터베이스에서 불러오지만 두 번째는 1차 캐시에서 불러오기 때문에 처음 한 번만 쿼리가 출력될 것이다.

```java
Member findMember1 = em.find(Member.class, 101L);
Member findMember2 = em.find(Member.class, 101L);
```

실행한 결과

![https://i.imgur.com/QOgCEak.png](https://i.imgur.com/QOgCEak.png)

SELECT 쿼리가 한 번만 처리된다.

- 조회할 때 메모리에 있는 1차 캐시에서 바로 불러온다는 성능상의 이점은 있지만, 현업에서 큰 도움은 되지 않는다.
- 1차 캐시는 데이터베이스 한 트랜잭션 안에서만 효과가 있다. 쉽게 말하면, 고객의 요청이 들어오고 비지니스가 끝나버리면 영속성 컨텍스트를 지우기 때문에 1차 캐시도 날라간다.

### 영속 엔티티의 동일성 보장

```java
Member findMember1 = em.find(Member.class, 101L);
Member findMember2 = em.find(Member.class, 101L);

System.out.println("result = " + (findMember1 == findMember2));
```

영속성 컨텍스트는 1차 캐시에 있는 같은 엔티티 인스턴스를 반환하기 때문에 둘은 같은 인스턴스고, result = true 가 출력된다.

## 2) 엔티티 등록

![https://i.imgur.com/dRC8V2h.png](https://i.imgur.com/dRC8V2h.png)

- 엔티티 매니저는 트랜잭션을 커밋하기 직전까지 데이터베이스에 엔티티를 저장하지 않고, 내부 쿼리 저장소에 INSERT SQL을 쌓아둔다.

![https://i.imgur.com/PZUFxuM.png](https://i.imgur.com/PZUFxuM.png)

- 트랜잭션을 커밋할 때 모아둔 쿼리를 데이터베이스에 보내는데, 이것을 트랜잭션을 지원하는 쓰기 지연이라 한다.

코드를 통해 확인해보자.

```java
Member member1 = new Member(150L, "A");
Member member2 = new Member(160L, "B");

em.persist(member1);
em.persist(member2);

System.out.println("=================");

tx.commit();
```

실행한 결과

![https://i.imgur.com/TQFkPvS.png](https://i.imgur.com/TQFkPvS.png)

System.out이 출력된 후, tx.commit() 할 때 INSERT 쿼리가 데이터베이스에 반영됐다.

persistence.xml에 다음과 같이 옵션을 주면 여러 개의 구문을 여러 번 network 를 통해 보내는 것이 아니라 합쳐서 1개로 보내기에 성능 개선을 할 수 있다.

```xml
<property name="hibernate.jdbc.batch_size" value="10"/>
```

## 3) 엔티티 수정

- JPA로 엔티티를 수정할 때는 단순히 엔티티를 조회해서 데이터만 변경하면 된다.

```java
Member member = em.find(Member.class, 150L);
member.setName("ZZZZZ");

System.out.println("=================");

tx.commit();
```

실행한 결과

![https://i.imgur.com/eaIGAsg.png](https://i.imgur.com/eaIGAsg.png)

자바 컬렉션을 다루듯이 값만 바꿨는데 UPDATE 쿼리가 처리됐다.

- 엔티티의 변경사항을 데이터베이스에 자동으로 반영하는 기능을 **변경 감지**(dirty checking)라 한다.

![https://i.imgur.com/uvGbgns.png](https://i.imgur.com/uvGbgns.png)

- 변경 감지 매커니즘
    - 트랜잭션을 커밋하면 엔티티 매니저 내부에서 먼저 플러시가 호출되는데, 플러시 시점에 스냅샷과 엔티티를 비교해서 변경된 엔티티를 찾는다.
        - JPA는 엔티티를 영속성 컨텍스트에 보관할 때, 최초 상태를 복사해서 저장해두는데 이것을 스냅샷이라 한다.
    - 변경된 엔티티가 있으면 수 정 쿼리를 생성해서 쓰기 지연 SQL 저장소에 보낸다.
    - 쓰기 지연 저장소의 SQL을 데이터베이스에 보낸다.
    - 데이터베이스 트랜잭션을 커밋한다.

## 4) 엔티티 삭제

- 커밋 시점에 DELETE 쿼리가 실행된다.

```java
//삭제 대상 엔티티 조회
Member memberA = em.find(Member.class, “memberA");

em.remove(memberA); //엔티티 삭제
```

# 5. 플러시

- 영속성 컨텍스트의 변경 내용을 데이터베이스에 반영하는 것
- 영속성 컨텍스트를 비우는 것이 아님
- 1차 캐시를 지우는 것은 아님
- 트랜잭션이라는 작업 단위가 중요 → 커밋 직전에만 동기화하면 됨

## 1) 플러시 발생 시

- 변경 감지(dirty checking) 동작
- 수정된 엔티티를 쓰기 지연 SQL 저장소에 등록
- 쓰기 지연 SQL 저장소의 쿼리(**등록, 수정, 삭제** 쿼리)를 데이터베이스에 전송

## 2) 영속성 컨텍스트를 플러시 하는 방법

### (1) em.flush() : 직접 호출

```java
Member member = new Member(200L, "member200");
em.persist(member);

em.flush();

System.out.println("=================");

tx.commit();
```

실행한 결과

![https://i.imgur.com/9g83se2.png](https://i.imgur.com/9g83se2.png)

commit하기 전 em.fluch()가 실행될 때 쿼리가 DB에 반영됐다.

### (2) 트랜잭션 커밋 : 플러시 자동 호출

### (3) JPQL 쿼리 실행 : 플러시 자동 호출

```java
em.persist(memberA);
em.persist(memberB);
em.persist(memberC);

//중간에 JPQL 실행
query = em.createQuery("select m from Member m", Member.class);
List<Member> members= query.getResultList();
```

JPQL 실행할 때 플러시가 자동 호출되지만, 만약 Member 테이블이 아닌 전혀 다른 테이블을 SELECT 한다면, 'FlushModeType.COMMIT' 이라는 플러시 모드 옵션을 통해 커밋할 때만 플러시를 호출하게 변경할 수 있다.

```java
em.setFlushMode(FlushModeType.COMMIT)
```

기본값은 'FlushModeType.AUTO' 이다.

# 6. 준영속 상태

- 영속 → 준영속
- 영속 상태의 엔티티가 영속성 컨텍스트에서 분리(detached)된 것
- 영속성 컨텍스트가 제공하는 기능을 사용 못함

## 1) **영속 상태의 엔티티를 준영속 상태로 만드는 방법**

### (1) em.detach(entity) : 특정 엔티티만 준영속 상태로 전환

```java
Member member = em.find(Member.class, 150L);
member.setName("AAAAA");

em.detach(member);

System.out.println("=================");

tx.commit();
```

실행한 결과

![https://i.imgur.com/S45YULa.png](https://i.imgur.com/S45YULa.png)

SELECT 쿼리만 처리되고 UPDATE는 처리되지 않았다.

### (2) em.clear() : 영속성 컨텍스트를 완전히 초기화

```java
Member member = em.find(Member.class, 150L);
member.setName("AAAAA");

em.clear();

Member member2 = em.find(Member.class, 150L);

System.out.println("=================");

tx.commit();
```

실행한 결과

![https://i.imgur.com/3Xxu1GP.png](https://i.imgur.com/3Xxu1GP.png)

em.clear()로 인해 영속성 컨텍스트가 초기화됐기 때문에 SELECT 쿼리를 데이터베이스에 한 번 더 처리한다.

### (3) em.close() : 영속성 컨텍스트를 종료

# 4. 엔티티 매핑

### JPA의 다양한 매핑 어노테이션

- 객체와 테이블 매핑: **@Entity, @Table**
- 필드와 컬럼 매핑: **@Column**
- 기본 키 매핑: **@Id**
- 연관관계 매핑: **@ManyToOne,@JoinColumn**

# 1. 객체와 테이블 매핑

## 1) @Entity

- @Entity가 붙은 클래스는 JPA가 관리, 엔티티라 한다.
- JPA를 사용해서 테이블과 매핑할 클래스는 **@Entity** 필수다.
- 주의
    - **기본 생성자 필수**(파라미터가 없는 public 또는 protected 생성자)
        - JPA가 엔티티 객체를 생성할 때 기본 생성자를 사용하기 때문이다.
    - final 클래스, enum, interface, inner 클래스 사용할 수 없다.
    - 저장한 필드에 final 사용할 수 없다.
- 속성 : name

    ```java
    @Entity(name = "Member")
    public class Member {
    }
    ```

    - JPA에서 사용할 엔티티 이름을 지정한다.
    - 기본값: 클래스 이름을 그대로 사용(예: Member)
    - 같은 클래스 이름이 없으면 가급적 기본값을 사용한다.

## 2) @Table

- 엔티티와 매핑할 테이블을 지정한다.
- 속성
    - name : 매핑할 테이블 이름
    - catalog : 데이터베이스 catalog 매핑
    - schema : 데이터베이스 schema 매핑
    - uniqueConstraints(DDL) : DDL 생성 시에 유니크 제약 조건 생성

# 2. 데이터베이스 스키마 자동 생성

- DDL(Data Definition Language, 테이블 생성, 삭제)을 애플리케이션 실행 시점에 자동 생성
- 테이블중심 → 객체중심
- 데이터베이스 방언을 활용해서 데이터베이스에 맞는 적절한 DDL 생성
- 이렇게 **생성된 DDL은 개발 장비에서만 사용**
- 생성된 DDL은 운영서버에서는 사용하지 않거나, 적절히 다듬은 후 사용

**persistence.xml에서 다음과 같은 속성 부여하기**

```xml
<property name="hibernate.hbm2ddl.auto" value="create" />
```

![https://i.imgur.com/dmDP2D1.png](https://i.imgur.com/dmDP2D1.png)

[제목 없음](https://www.notion.so/b6855219dfb244db9609cee6db767de3)

### 주의

- 운영 장비에는 절대 create, create-drop, update 사용하면 안된다.
- 개발 초기 단계는 create 또는 update
- 테스트 서버는 update 또는 validate
- 스테이징과 운영 서버는 validate 또는 none

# 3. DDL 생성 기능

- 이런 기능들은 단지 DDL을 자동 생성할 때만 사용되고 JPA의 실행 로직에는 영향을 주지 않는다.

## 1) 데이터베이스 컬럼명 바꾸기

```java
@Column(name = "USERNAME")
private String name;
```

## 2) not null 제약 조건, 문자의 크기 조건 추가하기

```java
@Column(nullable = false, length = 10)
private String name;
```

## 3) 유니크 제약조건 추가하기

```java
@Entity(name = "Member")
@Table(uniqueConstraints = {@UniqueConstraint(
        name="NAME_AGE_UNIQUE", 
        columnNames={"NAME", "AGE"} )})
public class Member {
}
```

생성된 DDL

```sql
AlTER TABEL MEMBER
	ADD CONSTRAINT NAME_AGE_UNIQUE UNIQUE (NAME, AGE)
```

# 4. 필드와 컬럼 매핑

```java
@Entity
public class Member {

    @Id
    private Long id;

    @Column(name = "name")
    private String username;

    private Integer age;

    @Enumerated(EnumType.STRING)
    private RoleType roleType;

    @Temporal(TemporalType.TIMESTAMP)
    private Date createdDate;

    @Temporal(TemporalType.TIMESTAMP)
    private Date lastModifiedDate;

    @Lob
    private String description;

    @Transient
    private int temp;

    public Member() {

    }
}
```

실행한 결과

![https://i.imgur.com/ViJPDMU.png](https://i.imgur.com/ViJPDMU.png)

### 매핑 어노테이션 정리

![https://i.imgur.com/dU1yS7X.png](https://i.imgur.com/dU1yS7X.png)

## 1) @Column

![https://i.imgur.com/qKPmEew.png](https://i.imgur.com/qKPmEew.png)

## 2) @Enumerated

- 자바 Enum 타입을 매핑할 때 사용

![https://i.imgur.com/wKWNoFS.png](https://i.imgur.com/wKWNoFS.png)

- EnumType.ORDINAL 사용할 때 주의해야 하는 이유
    - 데이터베이스에 숫자 0, 1, 2... 로 차례로 들어가는데, 만약 Enum 클래스에서 Enum 순서가 바뀐다면 이미 들어간 데이터베이스와 중복된다.

```java
@Enumerated(EnumType.STRING)
private RoleType roleType;
```

## 3) @Temporal

- 날짜 타입(java.util.Date, java.util.Calendar)을 매핑할 때 사용
- 참고로 LocalDate, LocalDateTime을 사용할 때는 생략 가능(최신 하이버네이트 지원)

![https://i.imgur.com/5GtxsPV.png](https://i.imgur.com/5GtxsPV.png)

```java
@Temporal(TemporalType.TIMESTAMP)
private Date createdDate;

@Temporal(TemporalType.TIMESTAMP)
private Date lastModifiedDate;
```

## 4) @Lob

- 데이터베이스 BLOB, CLOB 타입과 매핑
- @Lob에는 지정할 수 있는 속성이 없다.
- 매핑하는 필드 타입이 문자면 CLOB 매핑, 나머지는 BLOB 매핑
    - CLOB: String, char[], java.sql.CLOB
    - BLOB: byte[], java.sql. BLOB

```java
@Lob
private String description;
```

## 5) @Transient

- 필드 매핑X
- 데이터베이스에 저장X, 조회X
- 주로 메모리상에서만 임시로 어떤 값을 보관하고 싶을 때 사용

```java
@Transient
private Integer temp;
```

# 5. 기본 키 매핑

### 기본 키 매핑 어노테이션

- @Id
- @GeneratedValue

```java
@Id @GeneratedValue(strategy = GenerationType.AUTO) 
private Long id;
```

### 기본 키 매핑 방법

- 직접 할당: **@Id만 사용**
- 자동 생성(**@GeneratedValue**)
    - **IDENTITY**: 데이터베이스에 위임, MYSQL
    - **SEQUENCE**: 데이터베이스 시퀀스 오브젝트 사용, ORACLE
        - @SequenceGenerator 필요
    - **TABLE**: 키 생성용 테이블 사용, 모든 DB에서 사용
        - @TableGenerator 필요
    - **AUTO**: 방언에 따라 자동 지정, 기본값

## 1) 기본 키 직접 할당 전략

- 기본 키를 직접 할당하려면 다음 코드와 같이 @Id로 매핑하면 된다.

```java
@Id
@Column(name = "id")
private String id;
```

- 기본 키 직접 할당 전략은 em.persist() 로 엔티티를 저장하기 전에 애플리케이션에서 기본 키를 직접 할당하는 방법이다.

```java
Member member = new Member();
member.setId("ID_A");
member.setUsername("C");

em.persist(member);
```

## 2) IDENTITY 전략

### (1) 특징

- 기본 키 생성을 데이터베이스에 위임
- 주로 MySQL, PostgreSQL, SQL Server, DB2에서 사용 (예: MySQL의 AUTO_ INCREMENT)
- JPA는 보통 트랜잭션 커밋 시점에 INSERT SQL 실행
- AUTO_ INCREMENT는 데이터베이스에 INSERT SQL을 실행한 이후에 ID 값을 알 수 있음
- IDENTITY 전략은 commit하는 시점이 아니라, em.persist() 시점에 즉시 INSERT SQL 실행하고 DB에서 식별자를 조회
- 즉시 실행되기 때문에 INSERT 쿼리를 모아서 할 수 없다.

### (2) 매핑

**H2의 AUTO_INCREMENT 기능 수행하는 예제**

persistence.xml

```xml
<property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
```

Member

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
private String id;

@Column(name = "name", nullable = false)
private String username;
```

실행한 결과

![https://i.imgur.com/uIVPrkX.png](https://i.imgur.com/uIVPrkX.png)

id varchar(255) generated by default as identity 가 처리된다.

**mySQL의 AUTO_INCREMENT 기능 수행하는 예제**

persistence.xml

```xml
<property name="hibernate.dialect" value="org.hibernate.dialect.MySQL5Dialect"/>
```

실행한 결과

![https://i.imgur.com/N0D8OGl.png](https://i.imgur.com/N0D8OGl.png)

id varchar(255) not null auto_increment 로 바껴서 처리된다.

두 번 실행하면 id값이 자동으로 생성되어 데이터베이서에 순서대로 저장된다.

![https://i.imgur.com/QVUuCQ5.png](https://i.imgur.com/QVUuCQ5.png)

## 3) SEQUENCE 전략

### (1) 특징

- 데이터베이스 시퀀스는 유일한 값을 순서대로 생성하는 특별한 데이터베이스 오브젝트(예: 오라클 시퀀스)
- 오라클, PostgreSQL, DB2, H2 데이터베이스에서 사용 가능
- INSERT 쿼리가 처리되기 전에 시퀀스값을 불러와서 id에 저장한 후 영속성 컨텍스트에 올림

### (2) 매핑

Member

```java
@Entity
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE)
    private Long id;

    @Column(name = "name", nullable = false)
    private String username;
```

실행한 결과

![https://i.imgur.com/qDMtMk6.png](https://i.imgur.com/qDMtMk6.png)

**테이블마다 시퀀스를 따로 관리하고 싶을 때**

Member

```java
@Entity
@SequenceGenerator(
        name = "MEMBER_SEQ_GENERATOR",
        sequenceName = "MEMBER_SEQ") //매핑할 데이터베이스 시퀀스 이름
public class Member {

    @Id
    @GeneratedValue(strategy = GenerationType.SEQUENCE, 
						generator = "MEMBER_SEQ_GENERATOR")
    private Long id;
```

실행한 결과

![https://i.imgur.com/k98yjab.png](https://i.imgur.com/k98yjab.png)

**@SequenceGenerator**

![https://i.imgur.com/hovqKaS.png](https://i.imgur.com/hovqKaS.png)

## 4) TABLE 전략

- 키 생성 전용 테이블을 하나 만들고 여기에 이름과 값으로 사용할 컬럼을 만들어 데이터베이스 시퀀스를 흉내내는 전략
- 장점 : 모든 데이터베이스에 적용 가능
- 단점 : 성능

### (1) 매핑

```java
@Entity
@TableGenerator(
            name = "MEMBER_SEQ_GENERATOR",
            table = "MY_SEQUENCES",
            pkColumnValue = "MEMBER_SEQ", allocationSize = 1)
    public class Member {
    @Id
    @GeneratedValue(strategy = GenerationType.TABLE,
            generator = "MEMBER_SEQ_GENERATOR")
    private Long id;
```

실행한 결과

![https://i.imgur.com/L1fhNnU.png](https://i.imgur.com/L1fhNnU.png)

데이터베이스

![https://i.imgur.com/e4u0Pq1.png](https://i.imgur.com/e4u0Pq1.png)

![https://i.imgur.com/X1jcv74.png](https://i.imgur.com/X1jcv74.png)

**@TableGenerator**

![https://i.imgur.com/lkOEhKO.png](https://i.imgur.com/lkOEhKO.png)

## 5) 권장하는 식별자 전략

- 기본 키 제약 조건 : null 아님, 유일, 변하면 안된다.
- 미래까지 이 조건을 만족하는 자연키는 찾기 어렵다. 대리키(대체키)를 사용하자.
- 예를 들어 주민등록번호도 기본 키로 적절하기 않다.
- 권장 : Long형 + 대체키(sequence) + 키 생성전략 사용


### 참고

- 김영한님의 자바 ORM 표준 JPA 프로그래밍 - 기본편 강의
- 김영한님의 자바 ORM 표준 JPA 프로그래밍 책