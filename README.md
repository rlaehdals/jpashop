
### JPA Study

## project setting
1. spring boot version 2.4.3
2. java 8


## dependedcy
1. spring-data JPA
2. h2 database
3. thymeleaf
4. validation
5. spring-boot web
6. lombok
7. querydsl


## application.yml explanation

```
spring: 
  datasource:
    url: ~      -> 본인이 설정한 h2 database 주소
    useranme:sa    -> sa라고 해준다 h2 database의 기본 사용자
    password:      -> 설정 하지 않을시 공란
    driver-calss-name: org.h2.Driver -> h2 database를 사용하기 위한 driver 설정
  jpa:
    hibernate:
      ddl-auto: create  -> DBMS의 스키마에 무슨 행위를 할지에 대한 설정 create, drop-create, none, update, validate가 있다. 
    properties:
      hiberante:
        format_sql: true  -> database에 날라가는 query가 log창에 보인다. 
      default_batch_fetch_size: 100  -> colloection에 대하여 fetchJoin을하면 페이징이 안되므로 한 번에 많은 데이터를 끌어오는 양을 설정하여 최적화
sever:
  error:
    include-stacktrace: always
    include-message: always
logging:
  level:
    org.hiberate.SQL: debug
```

## 데이터베이스 방언

1. JPA는 특정 데이터베이스에 종속되지 않는다.
2. 각각의 데이터베이스가 제공하는 SQL 문법과 함수는 조금씩 다르지만 구현체에서 컨트롤 해준다.
3. hibernate는 약 30가지 이상의 데이터베이스 방언을 지원해준다


## JPQL이란

1. sql을 추상화한 객체 지향 쿼리 언어이다.
2. 테이블이 아닌 객체를 대상으로 검색하는 객체지향 쿼리이다. 
3. SQL은 데이터베이스 테이블을 대상으로 쿼리이다.

# EX

```
private final EntityManager em;

public List<Member> findAll(){
  return em.createQuery(" select m from Member m ", Member.class)  -> query를 날리고 Member.class로 반환형 결정
                .getResultList();
}
```

## 영속성 컨텍스트
>엔티티를 영구 저장하는 환경

**생명주기 4가지**
1. 비영속 ( new/transient )
> 영속성 컨테스트와 전혀 관계가 없는 새로운 상태
2. 영속 ( managed )
> 영속성 컨텍스트에 관리되는 상태
3. 준영속 ( detached )
> 영속성 컨텍스트에 저장되었다가 분리된 상태
4. 삭제 ( removed )
> 삭제된 상태

## 영속성 컨텍스트의 좋은점
1. 1차 캐시  -> 영속성 컨텍스트에 1차캐시를 두어 데이터베이스에 동일한 쿼리가 여러번 날라가는 상황을 방지
2. 동일성 보장
3. 트랜잭션을 지원하는 쓰기 지연 가능  -> transaction.commit()을 통하여 한 번에 sql 을 보낼 수 있다. 
4. Dirty Checking ( 변경 감지 ) -> 영속되어 있는 객체가 변한다면 데이터베이스에 쿼리가 날라감
5. Lazy Loading ( 지연 로딩 ) -> N+1 쿼리 문제를 해결 가능, 알 수 없는 쿼리가 날라가는 것을 방지한다.

## 엔티티 매핑
1. **@Entity**
  * JPA가 괸리하는 엔티티라는 어노테이션이다.
  * 기본 생성자가 필수이다. (파라미터가 없는 protected 생성자 )
  * final 클래스, enum, interface 사용 불가
  * 저장할 필드에 final 사용 X
  * 속성 name --> JPA에서 사용할 엔티티의 이름을 설정한다. default 값은 클래스의 이름 그대로
2. **@Column**
  * 컬럼 매핑이다. 
  * 속성
    * name: 필드와 매핑할 테이블의 컬럼 이름
    * insertable, updatable: 등록, 변경 가능 여부
    * nullable: null값 허용을 결정
    * unique: 유니크한 제약조건을 건다.
    * columnDefinition: 데이터베이스에 직접 컬럼 정보를 줄 수 있다.
    * length: 문자 길이 제약조건 String 타입에만 사용가능
    * precision, scale: 소수나 큰 자리 수
3. **@Enumerated**
  * 자바 enum 타입을 매핑할 때 사용한다.
  * 속성
    * EnumType.ORDINAL: 순서를 데이터베이스에 저장
    * EnumType.STRING: 이름을 데이터 베이스에 저장
    > ORDINAL을 사용할 경우 추가로 Enum에 데이터를 추가했을 경우 문제가 생길 수 있다. 따라서 STRING을 사용하자

## 기본 키 매핑
1. @Id, @Generated
  * @Id만 사용할 경우 직접 할당하며 @Generated가 붙으면 자동생성
  * 자동 생성 전략
    * IDENTITY: 데이터 베이스에 위임
    * SEQUENCE: 데이터베이스 시퀀스 오브젝트 사용
    * TABLE: 키 생성용 테이블 사용
    * AUTO: 방언에 따라 자동 지정
IDENTIY 예시

```
@Entity
public class User{
  
  @Id @Generated( strategy = GenerationType.IDENTITY)
  private Long id;
}
```
SEQUENCE 예시

```
@Entity
@SequenceGeneratort(
  name = " USER_SEQ_GENERATOR "
  sequenceName = " USER_SEQ "
  initiaValue = 1, allocatioSize = 1)  // 시작 지점, 한 번에 가져올 SIZE 
public class User{

  @Id 
  @Generated( strategy = GenerationType.SEQUENCE, generator = "USER_SEQ_GENERATOR" )
  private Long id;
```
TABLE은 잘 사용하지 않아서 스킵

## 식별자 전략
1. 기본 키 제약 조건: null 이면 안되고, 유일, 변하면 안된다.
2. 미래까지 이 조건을 만족하는 자연키는 찾기 어렵다.
3. 권장형: Long + key

## 연관 관계

1. 단반향 연관관계

ex
```
@Entity
public class User{
  
  @Id @GeneratedValue
  private Long id;
  
  @Column
  private String name;
  
  @Column
  private int age;
  
  @ManyToOne
  @JoinColumn(name= "team_id")
  private Team team           // 연관 관계 설정
}
```

2. 양방향 연관관계 

ex
```
@Entity
public class User{
  
  @Id @GeneratedValue
  private Long id;
  
  @Column
  private String name;
  
  @Column
  private int age;
  
  @ManyToOne
  @JoinColumn(name= "team_id")
  private Team team           // 연관 관계 설정
}
@Entity
public class Team {
 
 @Id @GeneratedValue
 private Long id;
 
 @Column
 private String name;
 
 @OneToMany(mappedBy = "team")
 List<User> users = new ArrayList<User>();
 }
```
## 연관관계의 주인과 mappedBy
1. 객체의 두 관계중 하나를 연관관계의 주인으로 지정
2. 연관관계의 주인만이 외래키를 관리 
3. 주인이 아닌쪽은 읽기만 가능 -> mappedBy 속성이다.
4. 주인은 mappedBy 설정하지 않는다.
5. 주인이 아니면 mappedBy 속성으로 주인 지정
6. 연관관계의 주인은 외래키가 있는 곳으로 해라
7. 외래키는 1:N중에서 N에 보통 위치시키고 1:1의 경우 access가 많이 사용되는 곳에 둔다. 

## 연관관계 annotation
1. **@JoinColumn**
     * name: 매핑할 외래 키 이름
     * @Column 속성과 같은 것을 다 가지고 있다.
2. **@ManyToOne**
     * optional: false로 설정하면 연관된 엔티티가 항상 있어야한다.
     * fetch: 글로벌 패치 전략을 설정한다.   최적화를 위해 LAZY로 설정한다.
     * cascade: 영속성 전이 기능을 사용한다.
3. **@OneToMany**
     * mappedBy: 연관관계의 주인 필드를 선택한다.
     * fetch: 글로벌 패치 전략을 설정한다.
     * cascade: 영속성 전이 기능을 사용한다.
  
## 상속관계 매핑
1. **슈퍼타입 서브타입 논리 모델을 실제 물리 모델로 구현하는 방법**
  * 각각 테이블로 변환 -> 조인 전력
  * 통합 테이블로 변환 -> 단일 테이블 전략
  * 서브타입 테이블로 변환 -> 구현 클래스마다 테이블 전략
2. **주요 annotation**
  * @Inheritance ( strategy = InheritaneType.xxx)
    * JOINED: 조인전력
      * 장점: 테이블 정규화, 외래키 참조 무결성 제약조건 활용가능, 저장공간 효율화
      * 단점: 조회시 조인을 많이 사용, 성능저하, 조회 쿼리가 복잡함
    * SINGLE_TABLE: 단일 테이블 전략
      * 장점: 조인이 필요 없어서 일반적으로 조회 성능이 빠름, 조회 쿼리가 단순함
      * 단점: 자식 엔티티가 매핑한 걸럼은 모두 null 허용, 단이 테이블에 모든 것을 저장하므로 크기가 커진다.
    * TABLE_PER_CLASS: 구현 클래스마다 테이블 전략
      * 장점: 서브 타입을 명확하게 구분해서 처리할 때 효과적
      * 단점: 여러 자식 테이블을 함께 조회할 때 성능이 느림
  * @DiscriminatorColumn( name = "DTYPE" )
  * @DiscriminatorValue("xxx")
3. **MappedSuperClass**
  * 공통 매핑 정보가 필요할 때 사용
     * 상속관계 매핑 x
     * 엔티티x, 테이블과 매핑x
     * 부모 클래스를 상속 받는 자식 클래스에 매핑 정보만 제공
     * 조회, 검색 불가능
     * 직접 생성해서 사용할 일이 없으므로 추상 클래스 권장
 
 
 ## 프록시

1. 실제 클래스를 상속 받아서 만들어진다.
2. 실제 클래스와 겉 모양이 같다.
3. 사용하는 입장에서는 진짜 객체인지 프록시 객체인지 구분하지 않고 사용하면 된다.
4. 프록시 객체는 실제 객체의 참조( target )를 보관
5. 프록시 객체를 호출하면 프록시 객체는 실제 객체의 메소드 호출
 
 
## 즉시 로딩과 지연 로딩

1. 지연 로딩은 프록시를 두고 실제로 사용할 때 불러온다. 
2. 즉시 로딩은 바로 엔티티를 불러온다 
3. 지연 로딩 최적화 시 fetchJoin, default_batch_fetch_size를 설정하여 최적화한다.
> 가급적 지연 로딩만 사용한다. -> 즉시 로딩 적용 시 예상하지 못한 sql이 발생하고 N+1문제도 일으킬 수 있다.
 
## 영속성 전이: CASCADE

1. 특정 엔티티를 영속 상태로 만들 때 연관된 엔티티도 함께 영속 상태로 만들고 싶을 때
2. 영속성 전이는 연관관계를 매핑하는 것과는 아무런 관계가 없다. 
3. 종류
  * ALL: 모두 적용
  * PERSIST: 영속
  * REMOVE: 삭제
  * MERGE: 병합
  * REFRESH: REFRESH
  * DETACH: DETACH

## 고아객체
1. 고아 객체 제거: 부모 엔티티와 연관관계가 끊어진 자식 엔티티를 자동으로 삭제
2. orphanRemoval = true
3. 주의
  * 참조가 제거된 엔티티는 다른 곳에서 참조하지 않는 고아 객체로 보고 삭제하는 기능
  * 참조하는 곳이 하나일 때 사용해야한다.
  * 특정엔티티가 개인 소요할 때 사용
  
 ### JPA의 데이터 타입 분류 
 1. **엔티티 타입**
   * @Entity로 정의하는 객체
   * 데이터가 변해도 식별자로 지속해서 추적 가능
 2. **값 타입**
   * int, Integer, String처럼 단순히 값으로 사용하는 자바 기본타입
   * 식별자가 없고 값만 있으므로 변경시 추적 불가
   1. **값 타입 분류**
     * 자바 기본 타입: int, double
     * 래퍼 클래스: Integer, Long
     * String
     * 생명주기를 entity에 의존한다.
     * 값 타입은 공유하면 안된다.
   2. **임베디드 타입**
     * 새로운 값 타입을 직접 정의가 가능하다.
     * JPA는 임베디드 타입이라한다.
     * 주로 기본 값 타입을 모아서 만든다.
     * @Embeddable: 값타입을 정의하는 곳에 표시
     * @Embedded: 값 타입을 사용하는 곳에 표시
     * 기본 생성자 필수
   3. 컬렉션 값 타입 
     * 값 타입을 하나 이상 저장할 때 사용
     * @ElementCollection, @CollectionTable 사용
     * 데이터베이스는 컬렉션을 같은 테이블에 저장할 수 없다.
     * 컬렉션을 저장하기 위한 별도의 테이블이 필요함.
     * 고려 사항
       * 값 타입은 엔티티와 다르게 식별자 개념이 없다.
       * 값은 변경하면 추적이 어렵다
       * 일대다 관계를 생각하자

## JPQL
1. SQL 문법과 유사 SELECT, FROM, WHERE GROP BY, ORDER BY, HAVING, JOIN
2. JPQL은 엔티티 객체를 대상으로 쿼리
3. 엔티티와 속성은 대소문자 구분한다.
4. 별칭 가능
5. getResultList() -> List
6. getSingleResult() -> 단일 값


## 파라미터 바인딩
1. 이름 기준
  * em.createQuery( select m from Member m where m.username: username)
                  .setParameter("useranme", useranme);
2. 위치 기준
  * em.createQuery( select m from Member m where m.username=?1)
                  .setParameter(1, useranme);
3. 명확성을 위해서 이름 기준으로 한다.

## 조인
1. 내부조인 
  * SELECT m FROM Member m [INNER] JOIN m.team t
2. 외부조인
  * SELECT m FROM Member m LEFT [OUTER] JOIN m.team t 
3. 세타조인
  * select count(m) from Member m, Team t where m.username = t.name

## 서브쿼리
1. exists: 서브쿼리에 결과가 존재하면 참
2. all: 모두 만족하면 참
3. any, some: 조건을 하나라도 만족하면 참
4. in: 서브쿼리 결과중 하나라도 같은 것이 있으면 참
5. from절에서는 아직 사용할 수 없다.

## 조건식 - case식
1. select case when m.age <=10 then '어린이 요금'
               when m.age >= 60 then '노인 요금'
               else '기본요금'
          end
   from Member m

## 경로 표현식 특징
1. 상태 필드(state field): 경로 탐색의 끝, 탐색 x
2. 단일 값 연관 경로: 묵시적 내부 조인 (inner join) 발생, 탐색 o
3. 컬렉션 값 연관 경로: 묵시적 내부 조인 발생, 탐색x -> select t.members.username from Team t(x)
                                                 -> select m.username from Team t join t.members m (o)


## 페치 조인
1. SQL 조인 종류가 아니다.
2. JPQL에서 성능 최적화를 위해 제공하는 기능이다.
3. 연관된 엔티티나 컬렉션을 SQL 한 번에 함께 조회하는 기능이다.
4. join fetch 명령어 사용 -> select m from Member join fetch m.team
5. distinct를 사용하여 컬렉션에 의해서 발생하는 중복 제거 가능

## 페치 조인의 특징과 한계
1. 페치 조인 대상에는 별칭을 줄 수 없다.
  * 하이버네이트는 가능하지만 가급적 사용하지 말자
2. 둘 이상의 컬렉션은 페치 조인할 수 없다.
3. 컬렉션을 패치 조인하면 페이징 api를 사용할 수 없다.
4. 연관된 엔티티들을 SQL 한 번으로 조회하므로 성능이 최적화된다.
5. 실무에서는 글로벌 로딩 전략은 모두 지연 로딩

## 벌크 연산
1. UPDATE
> update Member m set m.age = m.age +1 where m.age>=10
2. Delete
> update와 동일하게 수행












