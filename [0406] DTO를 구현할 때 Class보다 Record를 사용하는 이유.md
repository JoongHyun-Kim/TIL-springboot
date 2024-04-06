## DTO를 구현할 때 Class보다 Record를 사용하는 이유

Java 16부터 정식으로 스펙에 도입된 **Record**에 대해 정리해보려 한다.

평소 Dto를 구현할 때 class 타입으로 만들어 사용했었는데, 프로젝트를 함께 하던 팀원 분을 통해 record를 사용해 Dto를 구현하는 방식을 접하게 되었다. Class가 아닌 Record를 사용하면 Boilerplate Code를 많이 줄여 좀 더 간결하게 표현할 수 있어서, 요즘에 dto를 구현할 때 record로 선언해 잘 활용하고 있다. 

오늘은 Record란 무엇인지, 그리고 특징에는 어떤 것들이 있는지 정리해보려 한다.
<br>
<br>

### 📌 Record란 무엇일까?
**Record로 선언한 Book Dto**

```java
public record Book(Long id, String title, String author, String isbn) {}

```

Record 타입으로 Book Dto를 선언하려면 위와 같이 작성하면 된다. 괄호 안에 필드명과 타입을 나열 해주기만 하면 된다. 즉, `Record`는 **필드명과 타입만 요구하는 불변성 데이터 클래스**이다.

IntelliJ에서는 record를 class 타입으로 변경해주는 기능을 제공하고 있다. 해당 기능을 사용해 Book Dto를 class로 리팩토링 하면, 아래와 같이 `생성자들`과 `toString`, `equals`, `hashCode`메서드를 추가로 작성되는 것을 볼 수 있다.
<br>
<br>

**Class로 변환한 Book Dto**

```java
import java.util.Objects;

public final class Book {
    private final String title;
    private final String author;

    public book(String title, String author) {
        this.title = title;
        this.author = author;
    }

    public String title() {
        return title;
    }

    public String author() {
        return author;
    }

    @Override
    public boolean equals(Object obj) {
        if (obj == this) return true;
        if (obj == null || obj.getClass() != this.getClass()) return false;
        var that = (book) obj;
        return Objects.equals(this.title, that.title) &&
                Objects.equals(this.author, that.author);
    }

    @Override
    public int hashCode() {
        return Objects.hash(title, author);
    }

    @Override
    public String toString() {
        return "book[" +
                "title=" + title + ", " +
                "author=" + author + ']';
    }
}
```

Class 타입으로 Dto를 구현하는 경우에는, `getter`, `setter` 뿐만 아니라, `equals`, `hashCode`, `toString`과 같은 오버라이드된 메서드를 계속 반복해서 작성하게 된다. 이는 곧 Boilerplate Code가 되고, 이를 Record를 활용해 불필요한 코드 반복을 줄일 수 있게 되는 것이다.
<br>
<br>

### 📌 Record의 특징에는 어떤 것들이 있을까?
**Record의 특징**
1. Record는 불변성을 보장하며, final로 선언된다. 그렇기 때문에 한 번 값이 정해지면 setter로 값을 변경하는 것이 불가능하다.
2. 각 필드의 접근자 메서드를 자동으로 생성해주기 때문에 Boilerplate Code를 효과적으로 줄일 수 있다.
3. toString, equals, hashCode 메서드를 자동으로 구현해준다.
4. Body 부분에 새로 메서드를 작성할 수 있다.
    
    ```java
    public record SignupRequest(String loginId,
                                String nickname,
                                String password,
                                String passwordCheck) {
        public User toUser(String encodedPassword) {
    
            UserProfile profile = UserProfile.builder()
                    .nickname(nickname)
                    .build();
            return User.builder()
                    .loginId(this.loginId)
                    .password(encodedPassword)
                    .profile(profile)
                    .build();
        }
    }
    ```
    
    위 코드에서 처럼, Body 부분에({…}) toUser라는 메서드를 추가로 작성할 수 있다.
<br>
<br>

**제한 사항**
1. 다른 클래스를 상속 받을 수 없다. (extends를 사용할 수 없다.)
2. Record에 선언된 각 필드는 private final로 선언되기 때문에 수정이 불가능하다.

위와 같은 제한 사항들이 있지만, 프로젝트에서 dto를 구현할 때마다 반복적인 코드를 작성하는 것보다는 Record로 선언해 간편하고 깔끔하게 사용할 수 있다는 장점이 정말 크다고 생각한다. 
Record를 활용하면 코드의 간결성과 가독성을 높일 수 있고, 유지보수에도 좋기 때문에 Record를 사용해 Dto를 구현하는 방법도 활용해보는 것을  추천한다.
<br>
<br>

## Q&A
> 스터디 시간에 나온 Q&A를 정리한 내용

**Q) Record의 경우 필드 위에 @JsonProperty와 같은 annotation을 붙일 수 있는가?** <br>
**A)** 아래는 Java Langauge Docs의 Record Classes 관련 부분이다.

**You can annotate a record class and its individual components**, for example:
```java
import java.lang.annotation.*;
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
public @interface GreaterThanZero { }
```
```java
record Rectangle(
    @GreaterThanZero double length,
    @GreaterThanZero double width) { }
```

> If you annotate a record component, then the annotation may be propagated to members and constructors of the record class. This propagation is determined by the contexts in which the annotation interface is applicable. In the previous example, the @Target(ElementType.FIELD) meta-annotation means that the @GreaterThanZero annotation is propagated to the field corresponding to the record component. Consequently, this record class declaration would be equivalent to the following normal class declaration:

<br>

문서에 따르면 Record 클래스는 물론 그 구성 요소에도 annotation을 달 수 있는데, Record의 구성 요소에 annotation을 달면 record의 멤버 및 생성자에 전파될 수 있다. 
이 때, 전파 여부는 annotation interface가 적용 가능한 컨텍스트에 의해 결정된다. 위 예시에서 @Target(ElementType.FIELD)에 의해, @GreaterThanZero가 record의 구성 요소에 해당하는 필드로 전파될 수 있는 것이다.
결국 위 record class는 아래와 같은 일반 Class와 동일하다.
```java
public final class Rectangle {
    private final @GreaterThanZero double length;
    private final @GreaterThanZero double width;
    
    public Rectangle(double length, double width) {
        this.length = length;
        this.width = width;
    }
    
    double length() { return this.length; }
    double width() { return this.width; }
}
```
<br>
<br>

**Q) Record를 사용하면 @Data annotation이 자동으로 붙는 것인가?** <br>
**A)** [Lombok 공식문서](https://projectlombok.org/features/Data)에 따르면, `Data` annotation은 @ToString, @EqualsAndHashCode, @Getter, @Setter and @RequiredArgsConstructor 기능 묶음이다.
@Data annotation이 자동으로 붙는다기 보다는, Record를 사용하면 `모든 필드에 대한 getter`, `equals`, `hashCode`, `toString` 메서드가 자동으로 제공된다고 이해하면 좋을 것 같다.

<br>
<br>

**Q) Record를 사용하는 경우 새로운 객체를 어떤 방식으로 생성할 수 있는가?** <br>
**A)**
```java
public record StudentRecord(
  String firstName, 
  String lastName, 
  Long studentId, 
  String email, 
  String phoneNumber, 
  String address, 
  String country, 
  int age) {
}
```
위와 같은 Student 정보를 담고 있는 Record가 있다고 하자. 새로운 StudentRecord 객체를 생성하려면 아래와 같이 new 키워드를 사용해 간단히 구현할 수 있다.
```java
StudentRecord john = new StudentRecord(
  "John", "Doe", null, "john@doe.com", null, null, "England", 20);
```
그러나 코드를 보면 알 수 있듯이, StudentRecord의 경우처럼 필드가 많은 경우에는, 각 필드가 어떤 것을 의미하는지 알아보기 어려울 수 있다는 단점이 존재한다. 
결국 이렇게 필드가 많거나 일부 필드는 필수값이 아닌 경우라면, record 보다는 class와 @Builder 패턴을 활용해 어떤 필드값에 어떤 값이 들어가는지 명시해주는게 더 좋을 수도 있다.

<br>
<br>

## References
[Java 14 Record Keyword | Baeldung](https://www.baeldung.com/java-record-keyword)

[Using Java Records with JPA | Baeldung](https://www.baeldung.com/spring-jpa-java-records)

[Deep Dive with Java Records with Jason Young](https://www.youtube.com/watch?v=eC5X0NEZ8hE)

[Record (Java SE 19 & JDK 19 [build 1])](https://download.java.net/java/early_access/panama/docs/api/java.base/java/lang/Record.html)
[Java Language Updates-Record Classes](https://docs.oracle.com/en/java/javase/17/language/records.html)
