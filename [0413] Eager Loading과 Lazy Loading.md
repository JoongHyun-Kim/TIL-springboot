# Eager Loading과 Lazy Loading

평소 Entity에서 `@ManyToOne(fetch = FetchType.LAZY)`과 같이 연관관계를 설정해주는 부분에서 습관적으로 FetchType을 지정해주고 있다는 생각이 들어, 오늘은 `FetchType`에 대해 정리해보려 한다.

Entity에서 Data fetching 및 loading의 두 가지 종류에 대해 정리하고 차이점 및 장/단점을 알아보자!
<br>
<br>

## Eager Loading과 Lazy Loading이란?

> **Eager Loading** is a design pattern in which data initialization occurs on the spot. <br>
**Lazy Loading** is a design pattern that we use to defer initialization of an object as long as it’s possible.
> 

Eager loading(즉시 로딩) 방식은 데이터를 조회할 때 연관된 데이터를 그 즉시 함께 로딩하는 방식이며, Lazy Loading(지연 로딩) 방식은 실제로 사용할 때(필요할 때) 객체를 로딩하는 방식이다.

다음의 예시 코드를 통해 두 방식이 어떻게 동작하는지 살펴보자.

```java
@Entity
@Table(name = "USER")
public class UserLazy {

    @Id
    @GeneratedValue
    @Column(name = "USER_ID")
    private Long userId;

    @OneToMany(fetch = FetchType.LAZY, mappedBy = "user")
    private Set<OrderDetail> orderDetail = new HashSet();
}
```

```java
@Entity
@Table (name = "USER_ORDER")
public class OrderDetail {
    
    @Id
    @GeneratedValue
    @Column(name="ORDER_ID")
    private Long orderId;
    
    @ManyToOne(fetch = FetchType.LAZY)
    @JoinColumn(name="USER_ID")
    private UserLazy user;
}
```

UserLazy와 OrderDetail의 연관관계를 살펴보면, 한 user가 여러 개의 user_order를 가질 수 있기 때문에 일대다 관계이다. 이 때, Eager Loading과 Lazy Loading은 각각 다음과 같이 동작한다.

**1) Eager Loading(즉시 로딩)**

만약 Eager Loading 방식을 사용한다면, User 데이터를 가져올 때 연관된 user_order 데이터들도 모두 가져와 메모리에 저장한다.

**2) Lazy Loading(지연 로딩)**

만약 Lazy Loading 방식을 사용한다면, User 데이터를 가져오더라도 명시적으로 user_order를 조회하지 않는 이상 user_order 데이터는 함께 가져오지 않는다.
<br>
<br>

FetchType 지정은 다음과 같이 명시적으로 작성해주면 된다.
```java
@OneToMany(fetch = FetchType.EAGER, mappedBy = "user")
private Set<OrderDetail> orderDetail = new HashSet();
```
```java
@OneToMany(fetch = FetchType.LAZY, mappedBy = "user")
private Set<OrderDetail> orderDetail = new HashSet();
```

> `@-ToMany` 연관관계는 `FetchType.LAZY`, `@-ToOne` 연관관계는 `FetchType.EAGER` 로 기본값이 설정되어 있다.
<br>
<br>
<br>

## 둘의 차이점은 뭘까?
Eager Loading과 Lazy Loading의 가장 큰 차이점은 앞서 정리했듯이, 데이터가 메모리에 로드되는 시점이 다르다는 것이다.
```java
List<UserLazy> users = sessionLazy.createQuery("From UserLazy").list(); // 모든 user 조회
UserLazy userLazyLoaded = users.get(3);
return (userLazyLoaded.getOrderDetail());
```
위 코드에서 Lazy Loading 방식을 채택한다면, getter 등을 통해 명시적으로 호출하는 경우에만 orderDetailSet이 initialized 되기 때문에, 두번째 줄인 `UserLazy userLazyLoaded = users.get(3);`에서 orderDetailSet 데이터를 로드하게 된다.
<br>
<br>

```java
List<UserEager> user = sessionEager.createQuery("From UserEager").list();
```
반면, Eager Loading 방식을 채택하는 경우에는 위와 같이 user를 로드할 때 orderDetailSet도 즉시 메모리에 로드된다.
또한, 생성되는 쿼리에도 차이가 있다. Eager Loading의 경우에는 연관된 모든 객체 데이터까지 한 번에 쿼리를 생성한다. 즉, 생성되는 쿼리의 양이 Lazy Loading과 비교해 많은 것이 특징이다. 
<br>
<br>
<br>

## 각 패턴의 장단점을 알아보자
### **Lazy Loading**

- 장점(Eager Loading과 비교해)
    - 초기 로딩 시간이 적다.
    - 메모리 사용량이 적다.
- 단점
    - Initialization이 Eager Loading에 비교해 지연되기 때문에 성능에 영향을 줄 수 있다.
    - 초기화되지 않은 객체를 다룰 때 주의가 필요하다. Lazy Loading을 사용하는 객체에 접근하려고 할 때, 해당 객체가 아직 로드되지 않았다면, 초기화가 필요한 순간에 `LazyInitializationException`과 같은 예외가 발생할 수 있다.
    

### Eager Loading

- 장점
    - Lazy Loading의 단점 중 하나인 초기화 지연으로 인한 성능 영향이 발생할 일이 없다.
- 단점
    - 한 번에 연관 데이터를 다 로드하기 때문에 초기 로딩 시간이 길다.
    - 불필요한 데이터까지 한 번에 로드하기 때문에 성능에 영향을 줄 수 있다.
<br>
<br>
<br>

## Q&A
> 스터디 시간에 받은 질문을 정리한 내용

**Q1)** FetchType.LAZY는 어느 연관관계에 지정해주는게 좋은가?


<br>
<br>
<br>

## References
[Eager/Lazy Loading in Hibernate Baeldung](https://www.baeldung.com/hibernate-lazy-eager-loading)
[Difference between FetchType LAZY and EAGER in Java Persistence API?](https://stackoverflow.com/questions/2990799/difference-between-fetchtype-lazy-and-eager-in-java-persistence-api)
