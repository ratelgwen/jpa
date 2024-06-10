# Week2

# 05 연관관계 매핑 기초

## 5.1 단방향 연관관계

- 회원과 팀이 있다.
- 회원은 하나의 팀에만 소속될 수 있다.
- 회원과 팀은 다대일 관계다.

- 객체 연관관계
    - 회원 객체는 팀 객체와 연관관계를 맺는다.
    - 회원 객체와 팀 객체는 단방향 관계
        - 회원은 Member.team필드를 통해서 팀을 알 수 있지만 반대로 팀은 회원을 알 수 없다.
            - member → team의 조회 = member.getTeam() (o)
            - team → member의 조회 = 없음
- 테이블 연관관계
    - 회원 테이블은 TEAM_ID 외래 키로 팀 테이블과 연관관계를 맺는다.
    - 회원 테이블과 팀 테이블은 양방향 관계
        - 외래키를 통해 팀과 회원 서로 조인 가능
- 객체 연관관계와 테이블 연관관계의 가장 큰 차이
    - 참조를 통한 연관관계는 언제나 단방향
    - 객체간에 연관관계를 양방향으로 만들고 싶으면 반대쪽에도 필드를 추가해서 참조를 보관해야 함 → 연관관계를 하나 더 만들어야 함
        - 양방향 관계가 아닌 서로 다른 단방향 관계 2개임
        - 테이블은 외래 키 하나로 양방향으로 조인 가능

- JPA를 사용한 매핑
    - 회원 엔티티
    
    ```java
    @Entity
    public class Member {
    
    	@Id
    	@Column(name = "MEMBER_ID")
    	private String id;
    	
    	private String username;
    	
    	//연관관계 매핑
    	@ManyToOne
    	@JoinCOlumn(name="TEAM_ID")
    	private Team team;
    	
    	//연관관계 설정
    	public void setTeam(Team team) {
    		this.team = team;
    	}
    	
    	//Getter, Setter ...
    }
    ```
    
    - 팀 엔티티
    
    ```java
    @Entity
    public class Team {
    
    	@Id
    	@Column(name = "TEAM_ID")
    	private String id;
    	
    	private String name;
    	
    	//Getter, Setter ...
    }
    ```
    

## 5.2 연관관계 사용

- 수정
    - em.update() 같은 메소드는 없음, 불러온 엔티티 값을 변경해두면 트랜잭션을 커밋할 때 플러시가 일어나면서 변경 감지 기능 작동
- 삭제
    
    ```java
    member1.setTeam(null); //회원1 연관관계 제거
    member2.setTeam(null); //회원2 연관관계 제거
    em.remove(team);
    ```
    
    - 외래 키 제약조건이 있으므로 팀1을 삭제하려면 연관관계를 먼저 끊어야 함

## 5.3 양방향 연관관계

팀에서 회원으로 접근하는 관계를 추가

변경된 팀 엔티티

```java
@Entity
public class Team {

	@Id
	@Column(name = "TEAM_ID")
	private String id;
	
	private String name;
	
	//==추가==//
	@OneToMany (mappedBy = "team")
	private List<Member> members = new ArrayList<Member>();
	
	//Getter, Setter ...
}
```

- 팀과 회원은 일대다 관계이므로 팀 엔티티에 컬렉션인 List<Member> members 추가
- 일대다 관계를 매핑하기 위해 @OneToMany 사용

## 5.4 연관관계의 주인

- mappedBy가 필요한 이유
    - 객체에는 양방향 연관관계라는 것이 없어서, 서로 다른 단방향 연관관계 2개를 애플리케이션 로직으로 잘 묶어서 양방향인 것처럼 보이게 하는 것

- 객체 연관관계
    - 회원 → 팀 연관관계 1개(단방향)
    - 팀 → 회원 연관관계 1개(단방향)
- 테이블 연관관계
    - 회원 ↔ 팀의 연관관계 1개(양방향)

1. 엔티티를 양방향 연관관계로 설정하면 객체의 참조는 둘인데 외래 키는 하나
2. → 둘 사이에 차이가 발생
3. 이런 차이로 인해 JPA에서는 두 객체 연관관계 중 하나를 정해서 테이블의 외래 키를 관리해야 하는데 이것을 연관관계의 **주인**이라 한다.

- 연관관계의 주인만이 데이터베이스 연관관계와 매핑되고 외래 키를 관리(등록, 수정, 삭제)할 수 있다. 반면에 주인이 아닌 쪽은 읽기만 할 수 있다.
- 어떤 연관관계를 주인
    - 테이블에 외래 키가 있는 곳으로 정해야 함
        - 예시에서는 회원 테이블이 외래 키를 가지고 있으므로 Member.team이 주인이 됨
        - 주인이 아닌 Team.members에는 mappedBy=”team” 속성을 사용해서 주인이 아님을 설정함
    - mappedBy 속성 사용하면 됨
    - 주인은 mappedBy 속성 사용x
    - @ManyToOne은 항상 연관관계의 주인이 되므로 mappedBy를 설정할 수 없다.

```java
team1.getMembers().add(member1); //무시(연관관계의 주인이 아님)
team1.getMembers().add(member2); //무시(연관관계의 주인이 아님)

member1.setTeam(team1); //연관관계 설정(연관관계의 주인)
member2.setTeam(team1); //연관관계 설정(연관관계의 주인)
```

## 5.6 양방향 연관관계의 주의점

연관관계의 주인에만 값을 저장하고 주인이 아닌 곳에는 값을 저장하지 않아도 될까?

→ 객체 관점에서 양쪽 방향에 모두 값을 입력해주는 것이 안전

```java
public void test순수한객체_양방향() {
	
	//팀1
	Team team1 = new Team("team1", "팀1");
	Member member1 = new Member("member1", "회원1");
	Member member2 = new Member("member2", "회원2");
	
	member1.setTeam(team1);         //연관관계 설정 member1 -> team1
	team1.getMembers().add(member1);//연관관계 설정 team1 -> member1

	member2.setTeam(team1);         //연관관계 설정 member2 -> team1
	team1.getMembers().add(member2);//연관관계 설정 team1 -> member2	
	
	List<Member> members = team1.getMembers();
	...
}
```

JPA를 사용

```java
public void testORM_양방향() {
	
	//팀1 저장
	Team team1 = new Team("team1", "팀1");
	em.persist(team1);
	
	Member member1 = new Member("member1", "회원1");
	
	//양방향 연관관계 설정
	member1.setTeam(team1);         //연관관계 설정 member1 -> team1
	team1.getMembers().add(member1);//연관관계 설정 team1 -> member1, 저장시사용x
	em.persist(member1);

	Member member2 = new Member("member2", "회원2");
	
	member2.setTeam(team1);         //연관관계 설정 member2 -> team1
	team1.getMembers().add(member2);//연관관계 설정 team1 -> member2, 저장시사용x	
	em.persist(member2);
}
```

양방향 리팩토링

```java
public void testORM_양방향_리팩토링() {
	
	//팀1 저장
	Team team1 = new Team("team1", "팀1");
	em.persist(team1);
	
	Member member1 = new Member("member1", "회원1");
	member1.setTeam(team1);         //연관관계 설정 member1 -> team1
	em.persist(member1);

	Member member2 = new Member("member2", "회원2");
	member2.setTeam(team1);         //연관관계 설정 member2 -> team1
	em.persist(member2);
}
```

- 연관관계 편의 메소드 작성 시 주의사항
    - setTeam() 메소드에는 버그가 있다.
    
    ```java
    member1.setTeam(teamA); //1
    member1.setTeam(teamB); //2
    Member findMember = teamA.getMember(); //member1이 여전히 조회됨
    ```
    
    - 연관관계를 변경할 때는 기존 팀이 있으면 기존 팀과 회원의 연관관계를 삭제하는 코드를 추가해야 함
        - teamB로 변경할 때 teamA → member1 관계를 제거해야 함
- 양방향 매핑 시에는 무한 루프에 빠지지 않게 조심해야 함
    - Member.toString()에서 getTeam()을 호출하고, Team.toString()에서 getMember()를 호출하면 무한 루프에 빠짐

# 06 다양한 연관관계 매핑

엔티티의 연관관계를 매핑할 때 고려해야 할 세가지

- 다중성
    - 다대일(@ManyToOne)
    - 일대다(@OneToMany)
    - 일대일(@OneToOne)
    - 다대다(@ManyToMany)
- 단방향, 양방향
- 연관관계의 주인
    - 왼쪽이 주인, 다대일이면 다(N)가 연관관계의 주인

## 6.1 다대일

DB 테이블의 일대다 관계에서 외래 키는 항상 다쪽에 있음

- 다대일 양방향 관계
    - 회원엔티티
    
    ```java
    @Entity
    public class Member {
    
    	@Id
    	@Column(name = "MEMBER_ID")
    	private Long id;
    	
    	private String username;
    	
    	//연관관계 매핑
    	@ManyToOne
    	@JoinCOlumn(name="TEAM_ID")
    	private Team team;
    	
    	//연관관계 설정
    	public void setTeam(Team team) {
    		this.team = team;
    		
    		//무한루프에 빠지지 않도록 체크
    		if(!team.getMembers().contains(this)){
    			team.getMembers().add(this);
    		}
    	}
    	
    }
    ```
    
    - 팀 엔티티
    
    ```java
    @Entity
    public class Team {
    
    	@Id
    	@Column(name = "TEAM_ID")
    	private Long id;
    	
    	private String name;
    	
    	@OneToMany(mappedBy = "team")
    	private List<Member> members = new ArrayList<Member>();
    	
    	public void addMember(Member member) {
    		this.members.add(member);
    		if(member.getTeam() != this){ //무한루프에 빠지지 않도록 체크
    			member.setTeam(this);
    		}
    	}	
    
    }
    ```
    
- 양방향을 외래 키가 있는 쪽이 연관관계의 주인이다.
    - ‘다’에 해당하는 Member 테이블이 외래 키를 가지고 있으므로 Member.team이 연관관계의 주인이다.
    - JPA는 외래 키를 관리할 때 연관관계의 주인만 사용함
        - 주인이 아닌 Team.members는 조회를 위한 JPQL이나 객체 그래프를 탐색할 때 사용
- 양방향 연관관계는 항상 서로를 참조해야 한다.
    - setTeam(), addMember()
    - 무한루프 주의

## 6.2 일대다

- 일대다 관계는 다대일 관계의 반대 방향
- 일대다 관계는 엔티티를 하나 이상 참조할 수 있으므로 자바 컬렉션인 Collection, List, Set, Map 중에 하나를 사용해야 함

일대다 단방향[1:N]

ex) 하나의 팀은 여러 회원을 참조할 수 있음

```java
@Entity
public class Team {

	@Id @GeneratedValue
	@Column(name = "TEAM_ID")
	private String id;
	
	private String name;
	
	@OneToMany (mappedBy = "team")
	**@JoinColumn(name = "TEAM_ID") //MEMBER 테이블의 TEAM_ID (FK)**
	private List<Member> members = new ArrayList<Member>();
	
	//Getter, Setter ...
}
```

일대다 단방향 매핑의 단점

- 매핑한 객체가 관리하는 외래 키가 다른 테이블에 있음
    - 연관관계 처리를 위한 UPDATE SQL을 추가로 실행해야 함.
- → 다대일 양방향 매핑 사용 권장

일대다 양방향[1:N, N:1]

- 일대다 양방향 매핑은 존재하지x, 다대일 양방향 매핑을 사용해야 함.(왼쪽이 연관관계의 주인일 때)
    - @OneToMany는 연관관계의 주인이 될 수 x
    - @ManyToOne이 연관관계의 주인
        - mappedBy 속성 없음
    - 일대다 양방향 팀 엔티티
    
    ```java
    @Entity
    public class Team {
    
    	@Id @GeneratedValue
    	@Column(name = "TEAM_ID")
    	private String id;
    	
    	private String name;
    	
    	@OneToMany (mappedBy = "team")
    	**@JoinColumn(name = "TEAM_ID")**
    	private List<Member> members = new ArrayList<Member>();
    	
    	//Getter, Setter ...
    }
    ```
    
    - 일대다 양방향 회원 엔티티
    
    ```java
    @Entity
    public class Member {
    
    	@Id @GeneratedValue
    	@Column(name = "MEMBER_ID")
    	private String id;
    	
    	private String username;
    	
    	//연관관계 매핑
    	@ManyToOne
    	**@JoinCOlumn(name="TEAM_ID", insertable = false, updatable = false)**
    	private Team team;
    
    	
    	//Getter, Setter ...
    }
    ```
    
    - 둘 다 같은 키를 관리하므로 문제 발생할 수 o
        - 다대일 쪽은 읽기만 가능하게 함
        

## 6.3 일대일 [1:1]

일대일 관계는 양쪽이 서로 하나의 관계만 가짐

ex) 회원은 하나의 사물함만 사용, 사물함도 하나의 회원에 의해서만 사용됨

테이블은 주 테이블이든 대상 테이블이든 외래 키 하나만 있으면 양쪽으로 조회 가능

- 주 테이블에 외래 키
    - 외래 키를 객체 참조와 비슷하게 사용 가능
        - 객체지향 개발자들이 선호
    - 장점
        - 주 테이블만 확인해도 대상 테이블과 연관관계가 있는지 알 수 있음
    
    ```java
    @Entity
    public class Member {
    
    	@Id @GeneratedValue
    	@Column(name = "MEMBER_ID")
    	private Long id;
    	
    	private String username;
    	
    	**@OneToOne
    	@JoinCOlumn(name="LOCKER_ID")**
    	private Locker locker;
    	...
    	
    }
    
    @Entity
    public class Locker {
    
    	@Id
    	@Column(name = "LOCKER_ID")
    	private Long id;
    	
    	private String name;
    	
    	**@OneToOne(mappedBy = "locker")**
    	private Member member;
    	...
    }
    ```
    
- 대상 테이블에 외래 키
    - 장점
        - 테이블 관계를 일대일에서 일대다로 변경할 때 테이블 구조를 그대로 유지할 수 있음
    
    ```java
    @Entity
    public class Member {
    
    	@Id @GeneratedValue
    	@Column(name = "MEMBER_ID")
    	private Long id;
    	
    	private String username;
    	
    	**@OneToOne(mappedBy = "member")**
    	private Locker locker;
    	...
    	
    }
    
    @Entity
    public class Locker {
    
    	@Id
    	@Column(name = "LOCKER_ID")
    	private Long id;
    	
    	private String name;
    	
    	**@OneToOne
    	@JoinColumn(name = "MEMBER_ID")**
    	private Member member;
    	...
    }
    ```
    

## 6.4 다대다 [N:N]

- 관계형 데이터베이스는 정규화된 테이블 2개로 다대다 관계를 표현할 수 없다.
- → 보통 다대다 관계를 일대다, 다대일 관계로 풀어내는 연결 테이블 사용
    - ex) 회원들은 상품을 주문하고, 상품들은 회원들에 의해 주문된다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc391c19-fd40-485e-b1a0-21dddd8108c7/9ce0c91d-5890-45b9-b259-5287f82835af/Untitled.png)

@ManyToMany를 사용하면 아래와 같이 다대다 관계를 편리하게 매핑할 수 있음

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc391c19-fd40-485e-b1a0-21dddd8108c7/2a3c2c74-20c0-42ae-b196-33e4aa6809ec/Untitled.png)

- 다대다 매핑의 한계와 극복, 연결 엔티티 사용
    - @ManyToMany를 사용하면 연결 테이블을 자동으로 처리해주어 편리하지만, 실무에서 사용하기에는 한계
        - ex) 회원이 상품을 주문하면 연결 테이블에 단순히 주문한 회원 아이디와 상품 아이디만 담고 끝나지 않음. 보통 연결 테이블에 주문 수량, 주문한 날짜와 같은 컬럼이 더 필요함
            - → 이렇게 컬럼을 추가하면 주문엔티티나 상품엔티티에는 추가한 컬럼들을 매핑할 수 없기 때문에 @ManyToMany를 사용할 수 없음.
    - 다대다 → 일대다, 다대일 관계로 풀어야 함.
        - ex) 회원상품(MemberProduct) 엔티티 추가
            
            ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc391c19-fd40-485e-b1a0-21dddd8108c7/c1dbb96b-7cda-4aaf-96ed-816c3d3ec54e/Untitled.png)
            
            - 회원상품 엔티티 코드
            
            ```java
            @Entity
            **@IdClass(MemberProductId.class)**
            public class MemberProduct {
            	
            	@Id
            	@ManyToOne
            	@JoinColumn(name = "MEMBER_ID")
            	private Member member; //MemberProductId.member와 연결
            	
            	@Id
            	@ManyToOne
            	@JoinColumn(name = "PRODUCT_ID")
            	private Product product; //MemberProductId.product와 연결
            	
            	private int orderAmount;
            	
            	...
            }
            ```
            
            - 회원상품 식별자 클래스
                - (복합키는 별도의 식별자 클래스를 만들어야 함)
                - Serializable을 구현해야 함
                - equals와 hashCode 메소드를 구현해야 함
                - 기본 생성자가 있어야 함
                - 식별자 클래스는 public이어야 함
                - @IdClass를 사용하는 방법 외에 @EmbeddedId를 사용하는 방법도 있음
        
        - 새로운 기본 키 사용
            - DB에서 자동으로 생성해주는 대리 키를 Long 값으로 사용
                - 장점
                    - 간편하고 거의 영구적으로 사용 가능
                    - 비즈니스에 의존하지 않음
                    - ORM 매핑 시에 복합 키를 만들지 않아도 되므로 간단한 매핑 가능
                
                ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc391c19-fd40-485e-b1a0-21dddd8108c7/06730970-b9ad-4463-877d-5c994fce5bec/Untitled.png)
                
        

# 07 고급 매핑

## 7.1 상속 관계 매핑

ORM에서 이야기하는 상속 관계 매핑은 객체의 상속 구조와 DB의 슈퍼타입 서브타입 관계를 매핑하는 것이다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc391c19-fd40-485e-b1a0-21dddd8108c7/af2ba3ca-013a-426f-8cd9-583806a7ac63/Untitled.png)

슈퍼타입 서브타입 논리 모델을 실제 물리 모델인 테이블로 구현할 때의 세가지 방법

- 조인 전략
    - 각각을 모두 테이블로 만들고 조회할 때 조인을 사용
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc391c19-fd40-485e-b1a0-21dddd8108c7/451d6598-fe67-43a0-a227-06ec9a9bba60/Untitled.png)
    
    - 특징
        - 조회할 때 조인을 자주 사용
            - 엔티티 각각을 모두 테이블로 만들고 자식 테이블이 부모 테이블의 기본 키를 받아서 기본 키 + 외래 키로 사용하는 전략
        - 타입을 구분하는 컬럼을 추가해야 함
            - 객체는 타입으로 구분할 수 있지만 테이블은 타입의 개념이 없기 때문
    - 장점
        - 테이블이 정규화 된다.
        - 외래 키 참조 무결성 제약조건을 활용할 수 있다.
        - 저장공간을 효율적으로 사용한다.
    - 단점
        - 조회할 때 조인이 많이 사용되므로 성능이 저하될 수 있다.
        - 조회 쿼리가 복잡하다.
        - 데이터를 등록할 INSERT SQL을 두 번 실행한다.
        
        ```java
        @Entity
        **@Inheritance(strategy = InheritanceType.JOINED)
        @DiscriminatorColumn(name = "DTYPE")**
        public abstract class Item {
        	
        	@Id @GeneratedValue
        	@Column(name = "ITEM_ID")
        	private Long id;
        	
        	private String name; //이름
        	private int priva;   //가격
        	...
        }
        
        @Entity
        @DiscriminatorValue("A")
        public class Album extends Item {
        	
        	private String artist;
        	...
        }
        
        @Entity
        **@DiscriminatorValue("M")**
        public class Movie extends Item {
        	
        	private String director; //감독
        	private String actor;    //배우
        	...
        }
        ```
        
        - @Inheritance(strategy = InheritanceType.JOINED)
            - 상속 매핑은 부모 클래스에 @Inheritance를 사용해야 함
            - 조인 전략을 사용하므로 InheritanceType.JOINED 사용
        - @DiscriminatorColumn(name = "DTYPE")
            - 부모 클래스에 구분 컬럼 지정
            - 이 컬럼으로 저장된 자식 테이블 구분 가능
            - 기본값 DTYPE
        - @DiscriminatorValue("M")
            - 엔티티를 저장할 때 구분 컬럼에 입력할 값 지정
            - ex) 영화 엔티티를 저장하면 구분 컬럼인 DTYPE에 값 M이 저장됨
- 단일 테이블 전략
    - 테이블을 하나만 사용해서 통합함
    - 구분 컬럼(DTYPE)으로 어떤 자식 데이터가 저장되었는지 구분함
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc391c19-fd40-485e-b1a0-21dddd8108c7/adaf80e2-0495-4438-a400-68b64f5b4ecb/Untitled.png)
    
    - 장점
        - 조회할 때 조인을 사용하지 않으므로 가장 빠름
        - 조회 쿼리가 단순함
    - 단점
        - 자식 엔티티가 매핑한 컬럼은 모두 null을 허용해야 함.
        - 단일 테이블에 모든 것을 저장하므로 테이블이 커질 수 있다.
            - 그러므로 상황에 따라서는 조회 성능이 오히려 느려질 수 있다.
    - 특징
        - 구분 컬럼을 꼭 사용해야 함
            - @DiscriminatorColumn을 꼭 설정해야 함
            - @DiscriminatorValue를 지정하지 않으면 기본으로 엔티티 이름을 사용한다.
                - ex) Movie,Album, Book
    
    ```java
    @Entity
    **@Inheritance(strategy = InheritanceType.SINGLE_TABLE)**
    @DiscriminatorColumn(name = "DTYPE")
    public abstract class Item {
    	
    	@Id @GeneratedValue
    	@Column(name = "ITEM_ID")
    	private Long id;
    	
    	private String name; //이름
    	private int price;   //가격
    	...
    }
    
    @Entity
    @DiscriminatorValue("A")
    public class Album extends Item { ... }
    
    @Entity
    @DiscriminatorValue("M")
    public class Movie extends Item { ... }
    
    @Entity
    @DiscriminatorValue("B")
    public class Book extends Item { ... }
    ```
    
- 구현 클래스마다 테이블 전략
    - 자식 엔티티마다 테이블을 만듦
    - 자식 테이블 각각에 필요한 컬럼이 모두 있다
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc391c19-fd40-485e-b1a0-21dddd8108c7/1de00ed0-1440-4854-9eec-6bc41eced529/Untitled.png)
    
    ```java
    @Entity
    **@Inheritance(strategy = InheritanceType.TABLE_PER_CLASS)**
    @DiscriminatorColumn(name = "DTYPE")
    public abstract class Item {
    	
    	@Id @GeneratedValue
    	@Column(name = "ITEM_ID")
    	private Long id;
    	
    	private String name; //이름
    	private int price;   //가격
    	...
    }
    
    @Entity
    public class Album extends Item { ... }
    
    @Entity
    public class Movie extends Item { ... }
    
    @Entity
    public class Book extends Item { ... }
    ```
    
    - 장점
        - 서브 타입을 구분해서 처리할 때 효과적
        - not null 제약조건을 사용할 수 있다.
    - 단점
        - 여러 자식 테이블을 함께 조회할 때 성능이 느리다(SQL에 UNION을 사용해야 한다).
        - 자식 테이블을 통합해서 쿼리하기 어렵다.
    - 특징
        - 구분 컬럼을 사용하지 않는다.
    - 추천하지 않는 전략.
    

## 7.2 @MappedSuperclass

등록일, 수정일 같이 여러 엔티티에서 공통으로 사용하는 매핑 정보만 상속 받고 싶을 때 사용, 실제 테이블과는 매핑되지 x

↔ @Entity는 실제 테이블과 매핑

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc391c19-fd40-485e-b1a0-21dddd8108c7/7bdebbe9-aef9-4d83-aee3-8b1147ed8b36/Untitled.png)

```java
**@MappedSuperclass ////**
public abstract class BaseEntity {
	
	@Id @GeneratedValue
	private Long id;
	private String name;
	...
}

@Entity
public class Member extends BaseEntity {

	//ID 상속
	//NAME 상속
	private String email;
	...
}

@Entity
public class Seller extends BaseEntity {

	//ID 상속
	//NAME 상속
	private String shopName;
	...
}
```

- 부모로부터 물려받은 매핑 정보 재정의
    - @AttributeOverride(s)

```java
@Entity
@**AttributeOverride**(name = "id", column = @Column(name = "MEMBER_ID"))
public class Member extends BaseEntity { ... }
```

- 연관관계를 재정의
    - AssociationOverride(s)
    
- 특징
    - 테이블과 매핑되지 않고 자식 클래스에 엔티티의 매핑 정보를 상속하기 위해 사용
    - @MappedSuperclass로 지정한 클래스는 엔티티가 아니므로 em.find()나 JPQL에서 사용할 수 x
    - 이 클래스를 직접 생성해서 사용할 일은 거의 없음, → 추상 클래스로 만드는 것을 권장
    

## 7.3 복합 키와 식별 관계 매핑

식별 관계 vs 비식별 관계

- 식별 관계
    - 부모 테이블의 기본 키를 내려받아서 자식 테이블의 기본 키 + 외래 키로 사용하는 관계
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc391c19-fd40-485e-b1a0-21dddd8108c7/4337d0f4-9afc-4a4a-8b88-8c2e5f4c48e1/Untitled.png)
    
- 비식별 관계
    - 부모 테이블의 기본 키를 받아서 자식 테이블의 외래 키로만 사용하는 관계
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc391c19-fd40-485e-b1a0-21dddd8108c7/ac59c5a7-37b0-40c8-a971-f9c65548c9cc/Untitled.png)
    
    - 필수적 비식별 관계
        - 외래 키에 NULL 비허용
    - 선택적 비식별 관계
        - 외래 키에 NULL 허용

복합 키 지원

- @IdClass
    - 데이터베이스에 맞춘 방법
    - 부모 클래스
    
    ```java
    @Entity
    **@IdClass(ParentId.class)**
    public class Parent {
    	
    	@Id
    	@Column(name = "PARENT_ID1")
    	private String id1; //ParentId.id1과 연결
    	
    	@Id
    	@Column(name = "PARENT_ID2")
    	private String id2; //ParentId.id2과 연결
    	
    	private String name;
    	...
    }
    ```
    
    - 식별자 클래스(복합키를 매핑하기 위해 별도로 생성해야 함)
    
    ```java
    public class ParentId implements Serializable {
    	
    	private String id1; //Parent.id1 매핑
    	private String id2; //Parent.id2 매핑
    	
    	public ParentId() {
    	}
    	
    	public ParentId(String id1, String id2) {
    		this.id1 = id1;
    		this.id2 = id2;
    	}
    	
    	@Override
    	public boolean equals(Object o) {...}
    	
    	@Override
    	public int hashCode() {...}
    }
    ```
    
    - 자식 클래스
    
    ```java
    @Entity
    public class Child {
    
    	@Id
    	private String id;
    	
    	@ManyToOne
    	@JoinCoulmns({
    		@JoinColumn(name = "PARENT_ID1",
    			referencedColumnName = "PARENT_ID1"),
    		@JoinColumn(name = "PARENT_ID2",
    			referencedColumnName = "PARENT_ID2")
    	})
    	private Parent parent;
    }
    ```
    
- @EmbeddedId
    - 객체지향적인 방법
    - Parent 엔티티에서 식별자 클래스를 직접 사용하고 @EmbeddedId 어노테이션 작성
    
    ```java
    @Entity
    public class Parent {
    	
    	@EmbeddedId
    	private ParentId id;
    	
    	private String name;
    	...
    }
    ```
    
    - 식별자 클래스
    
    ```java
    @Embeddable
    public class ParentId implements Serializable {
    	
    	@Column(name = "PARENT_ID1")
    	private String id1;
    	@Column(name = "PARENT_ID2")
    	private String id2;
    	
    	//equals and hashCode 구현
    	...
    }
    ```
    
- 복합 키와 equals(), hashCode()
    
    ```java
    ParentId id1 = new parentId();
    id1.setId1("myId1");
    id1.setId2("myId2");
    
    ParentId id2 = new parentId();
    id2.setId1("myId1");
    id2.setId2("myId2");
    
    id1.equals(id2) -> ?
    ```
    
    - equals를 적절히 오버라이딩하지 않았다면 결과는 거짓
    - 자바의 모든 클래스는 기본으로 Object 클래스를 상속받는데 이 클래스가 제공하는 기본 equals()는 인스턴스 참조 값 비교인 == 비교(동일성 비교)를 하기 때문
    - 영속성 컨텍스트
        - 엔티티의 식별자를 키로 사용해서 엔티티를 관리한다.
        - 식별자를 비교할 때 equals()와 hashCode()를 사용
        - 따라서 식별자와 객체의 동등성이 지켜지지 않으면 예상과 다른 엔티티가 조회되거나 엔티티를 찾을 수 없는 등 영속성 컨텍스트가 엔티티를 관리하는 데 문제 발생

- @IdClass와 식별 관계

```java
@Entity
public class Parent {
	
    @Id @Column(name = "PARENT_ID")
    private String id;
    private String name;
}

@Entity
@IdClass(ChildId.class) 
public class Child {
	
    @Id
    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    public Parent parent;
    
    @Id @Column(name = "CHILD_ID")
    private String childId;
    
    private String name;
}

public class ChildId implements Serializable {
	
    private String parent;
    private String childId;
    
    // euqals, hashCode 구현
}

@Entity
@IdClass(GrandChildId.class) 
public class GrandChild {
	
    @Id
    @ManyToOne
    @JoinColumns({
    	        @JoinColumn(name = "PARENT_ID"),
                @JoinColumn(name = "CHILD_ID")})
    private Child child;
    
    @Id @Column(nmae = "GRANDCHILD_ID")
    private String id;
    
    private String name;
}

//손자 ID
public class GrandChildId implements Serializable {
	
    private ChildId child;
    private String id;
    
    // equals, hashCode 구현
}
```

- EmbeddedId와 식별 관계

```java
@Entity
public class Parent {
	
    @Id @Column(name = "PARENT_ID")
    private String id;
    private String name;
}

@Entity
public class Child {
	
    @EmbeddedId
    private ChildId id;
    
    @MapsId("parentId")
    @ManyToOne
    @JoinColumn(name = "PARENT_ID")
    public Parent Parent;
    
    private String name;
}

@Embeddable
public class ChildId implements Serializable {
	
    private String parentId;
    
    @Column(name = "CHILD_ID")
    private String childId;
    
    // euqals, hashCode 구현
}

@Entity
public class GrandChild {
	
    @EmbeddedId
    private GrandChildId id;
	
    @MapsId("childId")
    @ManyToOne
    @JoinColumns({
    	        @JoinColumn(name = "PARENT_ID"),
                @JoinColumn(name = "CHILD_ID")})
    private Child child;
    
    private String name;
}

@Embeddable
public class GrandChildId implements Serializable {
	
    private ChildId child;
    
    @Column(name = "GRANDCHILD_ID")
    private String id;
    
    // equals, hashCode 구현
}
```

@EmbeddedId는 식별 관계로 사용할 연관관계의 속성에 @MapsId를 사용하면 된다. Child 엔티티의 parent 필드를 보자. @IdClass와 다른 점은 @Id 대신에 @MapsId를 사용한 것이다. 

```java
@MapsId("parendId")
@ManyToOne
@JoinColumn(name = "PARENT_ID")
public Parent parent;
```

- 비식별 관계로 구현
    - 복합 키가 없으므로 복합 키 클래스를 만들지 않아도 된다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc391c19-fd40-485e-b1a0-21dddd8108c7/9d8201c6-3c43-4da1-82df-3a694fa9d08a/Untitled.png)

- 일대일 식별 관계
    - 일대일 식별자 관계는 자식 테이블의 기본 키 값으로 부모 테이블의 기본 키 값만 사용한다. 그래서 부모 테이블의 기본 키가 복합 키가 아니면 자식 테이블의 기본 키는 복합 키로 구성하지 않아도 된다.

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc391c19-fd40-485e-b1a0-21dddd8108c7/2b856eee-797c-495e-a9b5-a88e79d3a7e3/Untitled.png)

```java
@Entity
public class Board {
	
    @Id @Generatedvalue
    @Column(name = "BOARD_ID")
    private Long id;
    
    private String title;
    
    @OneToOne(mappedBy = "board")
    private BoardDetail boardDetail;
}

@Entity
public class BoardDetail {
	
    @Id
    private Long boardId;
    
    @MapsId
    @OneToOne
    @JoinColumn(name = "BOARD_ID")
    private Board board;
    
    private String content;
}
```

- 식별, 비식별 관계의 장단점
    - 데이터베이스 설계 관점에서 봤을 때 식별 관계보다 비식별 관계를 선호하는 이유
        - 식별 관계는 부모 테이블의 기본 키를 자식 테이블로 전파하면서 자식 테이블의 기본 키 컬럼이 점점 늘어남.
            - 조인할 때 SQL이 복잡해지고 기본 키 인덱스가 불필요하게 커질 수 있다.
        - 식별 관계는 2개 이상의 컬럼을 합해서 복합 기본 키를 만들어야 하는 경우가 많다.
        - 식별 관계를 사용할 때 기본 키로 비즈니스 의미가 있는 자연 키 컬럼을 조합하는 경우가 많음.
            - ↔ 비식별 관계의 기본 키는 비즈니스와 전혀 관계없는 대리 키를 주로 사용.
            - 비즈니스 요구사항은 시간이 지남에 따라 언젠가는 변하는데, 식별 관계의 자연 키 컬럼들이 자식에 손자까지 전파되면 변경하기 힘듦
            - 식별 관계는 부모 테이블의 기본 키를 자식 테이블의 기본 키로 사용하므로 비식별 관계보다 테이블 구조가 유연하지 못함
    - 객체 관계 매핑의 관점에서 봤을 때 비식별 관계를 선호하는 이유
        - 일대일 관계를 제외하고 식별 관계는 2개 이상의 컬럼을 묶은 복합 기본 키를 사용
            - 컬럼이 하나인 기본 키를 매핑하는 것보다 많은 노력 필요
        - 비식별 관계의 기본 키는 주로 대리 키를 사용하는데 JPA는 @GenerateValue처럼 대리 키를 생성하기 위한 편리한 방법 제공

- 추천 방법
    - 비식별 관계 사용
    - 기본 키는 Long 타입의 대리 키 사용

## 7.4 조인 테이블

테이블은 외래 키 하나로 연관관계를 맺을 수 있지만 연관관계를 관리하는 연결 테이블을 두는 방법

데이터베이스 테이블의 연관관계를 설계하는 방법

- 조인 컬럼 사용(외래 키)
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc391c19-fd40-485e-b1a0-21dddd8108c7/0d463574-3307-413b-9b03-f4b01b82b43d/Untitled.png)
    
- 조인 테이블 사용(테이블 사용)
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc391c19-fd40-485e-b1a0-21dddd8108c7/fddff331-d749-4a70-be2a-aa505000d659/Untitled.png)
    
    - 조인 컬럼을 사용하는 방법은 단순히 외래 키 컬럼만 추가해서 연관관계를 맺지만
    - 조인 테이블을 사용하는 방법은 연관관계를 관리하는 조인 테이블을 추가하고 이 테이블에서 두 테이블의 외래 키를 가지고 연관관계를 관리
    - 따라서 member와 locker에는 연관관계를 관리하기 위한 외래 키 컬럼이 없다.
    - 단점
        - 테이블을 하나 추가해야 함
        - 따라서 관리해야 하는 테이블이 늘어나고 회원과 사물함 두 테이블을 조인하려면 MEMBER_LOCKER까지 추가로 조인해야 함
        - → 기본은 조인 컬럼 사용, 필요시 조인 테이블 사용

- 일대일 조인 테이블
    - 일대일 관계를 만들려면 조인 테이블의 외래 키 컬럼 각각에 총 2개의 유니크 제약조건을 걸어야 한다.
    
    ```java
    @Entity
    public class Parent {
    	
        @Id @GeneratedValue
        @Column(name = "PARENT_ID")
        private Long id;
        
        private String name;
        
        @OneToOne
        @JoinTable(name = "PARENT_CHILD",
        	        joinColumns = @JoinColumn(name = "PARENT_ID"),
                    inverseJoinColumns = @JoinColumn(name = "CHILD_ID"))
        private Child child;
    
    }
    
    @Entity
    public class Child {
    	
        @Id @GeneratedValue
        @Column(name = "CHILD_ID")
        private Long id;
        
        private String name;
    }
    ```
    
    - 부모 엔티티를 보면 @JoinColumn 대신에 @JoinTable을 사용했다.
        - @JoinTable의 속성
            - name : 매핑할 조인 테이블 이름
            - joinColumns : 현재 엔티티를 참조하는 외래 키
            - inverseJoinColumns : 반대방향 엔티티를 참조하는 외래 키
- 일대다 조인 테이블
    - 일대다 관계를 만들려면 조인 테이블의 컬럼 중 다와 관련된 컬럼인 CHILD_ID에 유니크 제약조건을 걸어야 한다. (CHILD_ID는 기본 키이므로 유니크 제약 조건이 걸려 있다.)
    
    ```java
    @Entity
    public class Parent {
    	
        @Id @GeneratedValue
        @Column(name = "PARENT_ID")
        private Long id;
        
        private String name;
        
        @OneToMany
        @JoinTable(name = "PARENT_CHILD",
        	        joinColumns = @JoinColumn(name = "PARENT_ID"),
                    inverseJoinColumns = @JoinColumn(name = "CHILD_ID"))
        private List<Child> childs = new ArrayList<>();
    
    }
    
    @Entity
    public class Child {
    	
        @Id @GeneratedValue
        @Column(name = "CHILD_ID")
        private Long id;
        
        private String name;
    }
    ```
    
- 다대일 조인 테이블
    - 다대일은 일대다에서 방향만 반대이므로 조인 테이블 모양은 일대다와 동일하다.
    
    ```java
    @Entity
    public class Parent {
    	
        @Id @GeneratedValue
        @Column(name = "PARENT_ID")
        private Long id;
        
        private String name;
        
        @OneToMany(mappedBy = "parent")
        private List<Child> childs = new ArrayList<>();
    
    }
    
    @Entity
    public class Child {
    	
        @Id @GeneratedValue
        @Column(name = "CHILD_ID")
        private Long id;
        
        private String name;
        
        @ManyToOne(optional = false)
        @JoinTable(name = "PARENT_CHILD",
        	        joinColumns = @JoinColumn(name = "PARENT_ID"),
                    inverseJoinColumns = @JoinColumn(name = "CHILD_ID"))
        private Parent parent;
    }
    ```
    
- 다대다 조인 테이블
    - 다대다 관계를 만들려면 조인 테이블의 두 컬럼을 합해서 하나의 복합 유니크 제약 조건을 걸어야 한다.
    
    ```java
    @Entity
    public class Parent {
    	
        @Id @GeneratedValue
        @Column(name = "PARENT_ID")
        private Long id;
        
        private String name;
        
        @ManyToMany
        @JoinTable(name = "PARENT_CHILD",
        	        joinColumns = @JoinColumn(name = "PARENT_ID"),
                    inverseJoinColumns = @JoinColumn(name = "CHILD_ID"))
        private List<Child> childs = new ArrayList<>();
    
    }
    
    @Entity
    public class Child {
    	
        @Id @GeneratedValue
        @Column(name = "CHILD_ID")
        private Long id;
        
        private String name;
    }
    ```
    

## 7.5 엔티티 하나에 여러 테이블 매핑

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/fc391c19-fd40-485e-b1a0-21dddd8108c7/9f707e70-4fd8-4b1c-bef5-7b3558b9664b/Untitled.png)

- @SecondaryTable을 사용하여 한 엔티티에 여러 테이블을 매핑할 수 있다.
    
    ```java
    @Entity
    @Table(name = "BOARD")
    @SecondaryTable(name = "BOARD_DETAIL",
    	pkJoinColumns = @PrimaryKeyJoinColumn(name = "BOARD_DETAIL_ID"))
    public class Board {
    	
        @Id @GeneratedValue
        @Column(name = "BOARD_ID")
        private Long id;
        
        private String title;
        
        @Column(table = "BOARD_DETAIL")
        private String content;
    }
    ```
    
    - @SecondaryTable.name : 매핑할 다른 테이블 이름
    - @SecondaryTable.pkJoinColumns : 매핑할 다른 테이블의 기본 키 컬럼 속성
    - @Column(table = "BOARD_DETAIL") : content 필드는 BOARD_DETAIl 테이블의 컬럼에 매핑했다. title과 같이 테이블을 지정하지 않으면 기본 테이블인 BOARD와 매핑된다.
    - 더 많은 테이블을 이용하려면 @SecondaryTables를 이용한다.
