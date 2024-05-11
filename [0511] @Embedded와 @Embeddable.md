# @Embedded와 @Embeddable
오늘은 평소 Entity class에서 많이 사용하는 `@Embedded` annotation에 대해 알아보려 한다. `@Embedded`와 `@Embeddable`이 무엇이고, 언제 사용하는지 정리해보자!
<br>
<br>

## @Embedded와 @Embeddable이란 무엇이고 언제 사용하면 좋을까?
> An embeddable object is a Java class whose state maps to multiple columns of a table, but which doesn’t have its own persistent identity. That is, it’s a class with mapped attributes, but no `@Id` attribute.
>
> An embeddable object can only be made persistent by assigning it to the attribute of an entity. Since the embeddable object does not have its own persistent identity, its lifecycle with respect to persistence is completely determined by the lifecycle of the entity to which it belongs.
> 

Hibernate 공식 문서에 나와 있듯이, Embeddable 객체란 한 테이블의 여러 컬럼들에 해당 클래스의 상태가 매핑되지만 고유한 영속성 식별자가 없는 클래스를 의미한다. 즉, Embeddable 클래스는 테이블의 여러 컬럼과 연결되어 있지만, 그 자체로는 독립적으로 식별할 수 있는 고유한 ID가 없어 다른 Entity의 일부로 사용된다.
<br>그리고 이러한 Embeddable 객체를 나타내기 위해 사용하는 annotation이 바로 `@Embedded`와 `@Embeddable`이다.
<br>
<br>

예시를 보면서 더 쉽게 이해해보자!  
아래는 프로젝트에서 작성했던 User entity와 UserProfile 클래스 코드 일부이다.

**User**
```java
@Entity
public class User extends BaseEntity {
    ...
    @Embedded
    private UserProfile profile;
    ...
}
```
<br>

**UserProfile**
```java
@Embeddable
@NoArgsConstructor(access = AccessLevel.PROTECTED)
public class UserProfile {

    @Column(nullable = false, length = 20)
    private String nickname;
    private String profileImage;

    @Builder
    public UserProfile(String nickname) {
        this.nickname = nickname;
    }

    public void modifyNickname(String nickname) { this.nickname = nickname; }
    public void modifyProfileImage(String profileImage) { this.profileImage = profileImage; }
}
```

User entity를 보면, UserProfile 타입의 profile이라는 컬럼에 `@Embedded` annotation이 붙어있고 UserProfile 클래스에는 최상단에 `@Embeddable` annotation이 붙어있다. Embeddable 클래스에는 반드시 @Entity 대신 `@Embeddable` annotation을 붙여야 한다.  
`@Embeddable`을 붙여 클래스를 생성해주면, 이제 entity에서 컬럼의 타입으로 해당 클래스를 사용할 수 있게 된다. User entity에 UserProfile이라는 클래스 타입의 profile이 생기면서, UserProfile에 존재하는 여러 컬럼들(nickname, profileImage)이 User에 embed되는 것이다!
<br>
<br>

> An embeddable class must satisfy the same requirements that entity classes satisfy, with the exception that an embeddable class has no `@Id` attribute. In particular, it must have a constructor with no parameters.
> 
Embeddable 클래스는 entity 클래스와는 다르게 @Id 속성이 없고, 반드시 기본 생성자를 가지고 있어야 한다.
위 예시 코드에서는 직접 기본 생성자를 작성하는 대신 `@NoArgsConstructor(access = AccessLevel.PROTECTED)` annotation을 활용해 기본 생성자를 구현해 조건을 만족하도록 했다.
<br>
<br>
<br>

## 장점이 무엇일까?

개인적으로 User entity에 nickname, profileImage 등 user의 프로필과 관련된 여러 컬럼들을 늘어놓는 것보다는 UserProfile이라는 하나의 클래스로 관리하고 embed하는 방식을 선호하는데, 이 방식에는 여러 장점들이 존재한다.

> **장점**
> 
> 1. **코드 재사용성:** 코드 중복을 방지하면서 여러 entity에 embeddable 클래스를 재사용할 수 있다.
> 2. **정규화:** 테이블 수를 줄여 DB를 정규화하는데 도움이 되며 결과적으로 성능이 향상된다.
> 3. **데이터 무결성:** embeddable 클래스와 포함하는 클래스 간의 관계를 유지해 데이터 무결성을 보장한다.
> 4. **단순성:** 데이터를 매핑하는데 필요한 클래스와 테이블의 수를 줄여 개발 프로세스를 단순화한다.
> 5. **향상된 데이터 모델링:** 객체의 속성을 다른 객체 내에 캡슐화하여 데이터 구조를 보다 직관적이고 이해하기 쉽게 만들어 더 나은 데이터 모델링을 가능하게 한다.
> 6. **유지 관리 용이성:** 코드의 유지 관리를 보다 관리하기 쉽게 만든다.
> 7. **유연성:** embeddable 클래스는 여러 entity에서 사용할 수 있으며 @Embedded 및 @AttributeOverrides를 사용하여 복잡한 데이터 모델링도 지원한다.
> 8. **가독성:** @Embeddable 및 @Embedded 주석을 사용하면 코드를 더 쉽게 읽을 수 있고 자체 설명이 가능하다.
> 9. **성능:** 데이터를 가져오는 데 필요한 조인 수를 줄여 애플리케이션 성능을 향상시킨다.
> 10. **비즈니스 로직:** embeddable 클래스 내에 비즈니스 로직을 캡슐화하여 관리하기 쉽고 이해하기 쉽게 만든다.
<br>
<br>

**cf) @AttributeOverrides**  
[Jpa @Embedded and @Embeddable | Baeldung](https://www.baeldung.com/jpa-embedded-embeddable) <br>
@AttributeOverrides와 @AttributeOverride를 활용하면 embedded 타입의 컬럼 속성을 오버라이드할 수 있다.
<br>
<br>

## References
[An Introduction to Hibernate 6](https://docs.jboss.org/hibernate/orm/6.5/introduction/html_single/Hibernate_Introduction.html#embeddable-objects)  
[Jpa @Embedded and @Embeddable | Baeldung](https://www.baeldung.com/jpa-embedded-embeddable)  
[Hibernate - @Embeddable and @Embedded Annotation - GeeksforGeeks](https://www.geeksforgeeks.org/hibernate-embeddable-and-embedded-annotation/?ref=header_search)
