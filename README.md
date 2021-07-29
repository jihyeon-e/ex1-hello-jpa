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

  ### 참고

    - 김영한님의 자바 ORM 표준 JPA 프로그래밍 - 기본편 강의
    - 김영한님의 자바 ORM 표준 JPA 프로그래밍 책