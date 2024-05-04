# @Component와 @Bean
오늘은 @Component에 대해 정리해보고, @Bean과의 차이점도 알아보려 한다.
<br>
<br>

## @Component란 무엇일까?
`@Component`란 Spring이 사용자가 정의한 Bean을 자동으로 감지할 수 있도록 하는 annotation이다. 다시 말해, @Component annotaion을 통해 Bean을 생성한다고 볼 수 있다.

다른 코드를 작성하지 않고도 annotation만으로 Spring은 다음을 수행한다.

```md
1. @Component가 달린 클래스가 있는지 application을 스캔한다.
2. 인스턴스화하고 지정된 dependency를 주입한다.
```

Spring은 `@Controller`, `@Service`, `@Repository`와 같은 몇 가지 특수한 stereotype annotation을 제공하는데, 이는 모두 `@Component`를 메타 annotaion으로 사용하기 때문에 `@Component`와 동일한 기능을 제공한다. 즉, @Component에 특수한 용도와 의미를 추가한 것이라고 할 수 있다. <br>
Spring에서 자동으로 bean을 감지하게 하고 싶을 때 @Component를 사용할 수도 있지만, @Component를 사용해 custom annotation을 만들어 해당 클래스에 @Component를 붙여 사용할 수도 있다.
이러한 custom 어노테이션 또는 stereotype annotation을 사용하면, Spring은 특정 컴포넌트의 역할을 명확하게 파악하고, 그에 맞는 추가적인 처리나 최적화를 자동으로 수행할 수 있기 때문에, 가능한 @Component 대신 특화된 annotation을 사용할 수 있는 상황이라면 특화된 annotation을 사용하는 것이 좋다.
<br>
<br>

cf) What are Meta-Annotations?
> At its core, a meta-annotation is an annotation that annotates another annotation. It acts as a wrapper, allowing for the bundling of multiple annotations under one umbrella. This bundling provides developers with the ability to create custom annotations tailored to specific patterns or behaviors in their applications.
> 

<br>
<br>
<br>

## @ComponentScan이란 무엇일까?
@Component는 단순히 plain annotation이고, bean과 도메인 객체와 같은 다른 객체들을 구분하는 것이 목표라고 할 수 있다. <br>
그러나, Spring은 `@ComponentScan`을 사용해 ApplicationContext에 bean들을 모은다. <br>
우리가 보통 프로젝트의 root로 사용하는 @SpringBootApplication은 @ComponentScan을 포함하고 있어, 프로젝트의 루트에 @SpringBootApplication 클래스가 있으면 @ComponentScan에 의해 기본적으로 정의한 모든 @Component를 자동으로 스캔하게 된다. <br>
만약 @SpringBootApplication 클래스를 프로젝트의 루트에 위치시킬 수 없거나 외부 소스를 스캔하려는 경우에는 classpath에 존재하는 어떤 패키지든 명시적으로 스캔하도록 @ComponentScan을 사용할 수 있다.
<br>

예시를 살펴보자.
```java
package com.baeldung.component.scannedscope;

@Component
public class ScannedScopeExample {
}
```

```java
package com.baeldung.component.inscope;

@SpringBootApplication
@ComponentScan({"com.baeldung.component.inscope", "com.baeldung.component.scannedscope"})
public class ComponentApplication {
    //public static void main(String[] args) {...}
}
```

`com.baeldung.component.scannedscope` 패키지 안에 `ScannedScopeExample`이라는 이름의 클래스를 `@Component`를 붙여 정의한다. 이를 통해 spring이 자동으로 이 클래스를 bean으로 등록할 수 있게 한다.

`@ComponentScan`은 `com.baeldung.component.inscope`와 `com.baeldung.component.scannedscope` 두 패키지를 명시적으로 스캔 대상으로 지정한다. 이를 통해 spring은 기본적으로 스캔하는 패키지(보통 `@SpringBootApplication` 어노테이션이 위치한 패키지와 그 하위 패키지) 이외에도 추가적으로 지정된 패키지 안의 `@Component`가 붙은 클래스들을 bean으로 자동 등록할 수 있게 된다.

<br>
<br>
<br>

## @Component vs @Bean

`@Bean`은 Spring이 런타임에 bean을 모으는 데 사용하는 또 다른 주요 annotation이지만, class level에서 사용되지 않는다. 대신 메소드에 @Bean을 붙여 Spring이 해당 메소드의 결과를 Spring bean으로 저장하도록 한다.
@Component와 @Bean의 차이점을 정리해보면, 아래와 같다.

### @Component

- `@Component`는 클래스 레벨에서 사용되기 때문에, Spring이 해당 클래스를 자동으로 찾아서 ApplicationContext에 bean으로 등록한다. 즉, 별도의 설정 없이 Spring이 자동으로 객체를 생성하고 관리하게 해준다. 다시 말해, Spring이 필요할 때 인스턴스를 만들어 사용한다.
<br>

### @Bean

- `@Bean`은 메소드 레벨에서 사용된다. 이는 주로 `@Configuration`이 붙은 클래스 안의 메소드에 사용되며, 해당 메소드가 반환하는 **객체**를 Spring 컨테이너에 bean으로 등록한다.
- `@Bean`을 사용하면 bean의 생성 방식을 직접 제어할 수 있다. 즉, 객체를 생성하는 로직을 직접 작성할 수 있으며, 외부 라이브러리와 같이 직접 코드를 수정할 수 없는 클래스도 Spring bean으로 등록할 수 있다. 또한, 상황에 따라 다른 인스턴스를 반환하는 등의 추가 로직을 구현할 수도 있다.

정리하자면, `@Component`는 Spring이 자동으로 클래스를 찾아서 bean으로 만들어주는 반면, `@Bean`은 개발자가 bean의 생성 방식을 더 세밀하게 제어할 수 있게 해준다. 따라서 외부 라이브러리를 사용하거나, 생성 로직에 특별한 처리가 필요한 경우에 `@Bean`을 사용하는 것이 좋다.
<br>
<br>
<br>

## References
https://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/
https://www.baeldung.com/spring-component-annotation
https://medium.com/@AlexanderObregon/the-art-of-meta-annotations-in-spring-creating-custom-compositions-86d000d33796
