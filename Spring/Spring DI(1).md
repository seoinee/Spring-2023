# Spring DI(1)

## Spring Container
---
<br>

> 핵심기능
- Java 객체("bean")의 life-cycle 관리
  - 객체 생성, 초기화, 이용, 삭제(소멸)
- Dependency Injection(DI) 수행
  - 의존 관계에 따라 bean들을 연결
    - 의존하는 bean에 대한 참조 생성("bean wiring")

> Container의 종류
- BeanFactory interface의 구현체(class)
  - Bean 객체를 생성하고 DI를 실행하는 기본 기능 제공
- ApplicationContext interface 및 그 sub-interface의 구현체(class)
  - BeanFactory의 기능을 포함하여 다양한 부가 기능을 추가적으로 제공
    - Annotation 기반 설정
    - Java code 기반 설정("JavaConfig")
    - 웹 어플리케이션 관련 기능, 트랜잭션 처리, 메시지 처리 등
    - 
> BeanFactory interface
- org.springframework.beans.factory.BeanFactory
- Spring container에 대한 기본적인 API 정의
  - <T> T getBean(String name, Class<T> requiredType)
    → 주어진 이름과 타입을 가진 bean을 찾아 반환
  - <T> T getBean(Class<T> requiredType) → 주어진 타입을 가진 bean을 반환
  - Object getBean(String name) → 주어진 이름을 가진 bean을 반환 (타입 변환 필요)
  - Class<?> getType(String name) → 주어진 이름을 가진 bean의 타입을 반환
  - boolean containsBean(String name)
    → 주어진 이름을 가진 bean이 존재하는지 여부를 반환
  - boolean isPrototype(String name) / boolean isSingleton(String name)
  - boolean isTypeMatch(String name, Class targetType)
- 구현 클래스: XmlBeanFactory, SimpleJndiBeanFactory, StaticListableBeanFactory 등

> ApplicationContext interface
- org.springframework.context.ApplicationContext
- BeanFactory의 sub-interface
- 기본적인 bean 관리 외에, annotation 기반 설정, 메시지 및 이벤트 처리, 국제화, 트랜잭션 처리 등 다양한 부가 기능 제공
- 구현 클래스
 - GenericXmlApplicationContext
   - File system 또는 classpath에 속한 XML file의 설정 정보를 이용
   - FileSystemXmlApplicationContext, ClassPathXmlApplicationContext
 - AnnotationConfigApplicationContext
   - @Configuration annotation이 적용된 Java class의 설정 정보를 이용
 - XmlWebApplicationContext, AnnotationConfigWebApplicationContext
   - Web application (Servlet) 개발 시 사용하는 container
   - Spring MVC Framework에서 사용됨
<br>

##  Bean 생성 및 사용
---
<br>

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns=“http://www.springframework.org/schema/beans”
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans
			http://www.springframework.org/schema/beans/spring-beans.xsd">
	<bean id="bean ID" class="생성할 bean 객체의 class 이름(package 경로 포함)">
		<property name="bean의 property 이름">
			<value>property 값</value>
		</property>
		<property name="bean의 property 이름" ref="참조할 bean의 ID" />
	</bean>
	<bean id="bean ID" class="생성할 bean 객체의 class 이름">
		<constructor-arg><value>생성자의 파라미터 값</value></constructor-arg>
		<constructor-arg><ref bean="참조할 bean의 ID" /></constructor-arg>
	</bean>
</beans>

```

- beans : 설정 파일의 root element
- bean : container에 의해 생성 및 관리될 bean 객체를 정의
  - id/name 속성: bean을 구별하기 위한 식별자(유일한 값 지정)
  - class 속성: bean의 완전한 클래스 경로 지정(package 경로 포함)


> Bena 객체 획득
```
// Container 생성
ApplicationContext ctx = new ClassPathXmlApplicationContext("springIdol.xml");
// Container로부터 특정 bean 객체의 참조 정보를 가져옴
Performer performer1 = (Performer) ctx.getBean("duke"); // type casting 필요
Performer performer2 = ctx.getBean("duke", Performer.class); // type 제공
```
- Bean을 관리하는 container(BeanFactory 또는 ApplicationContext 객체)에 대해 getBean() method 호출

##  의존 관계 설정
---
<br>

1. 생성자 방식(constructor-based injection)
- Bean의 생성자(constructor)를 통해 의존 객체를 주입
- 이용가능한 생성자가 bean 클래스에 정의되어야 함
- 설정 방법
  - constructor-arg를 사용하여 생성자의 인자(argument)를 지정
    - <value>값</value>: int, double, String 등 기본 데이터 타입의 값을 전달
    - <ref bean=“bean 식별자”/>: bean 속성으로 bean 객체의 참조를 전달
  - 값이나 객체를 <constructor-arg>의 value, ref 속성을 통해 지정 가능
    - <constructor-arg value=“값” />
    - <constructor-arg ref =“bean 식별자” />
– 효과
  - Container는 다른 bean이 의존하는 bean 객체를 먼저 생성한 후 의존 객체나 값의 타입을 이용, 적절한 생성자를 찾아 실행
    - 의존 객체/값과 일치하거나 가장 근접한 타입의 인자를 가진 생성자 선택
    - 생성자 호출 시 의존 객체/값을 인자로 전달 (Dependency Injection)

2. Setter method 방식 (setter-based injection)
- Property에 대한 setter method를 통해 의존 객체 주입
- Bean class에 property에 대한 setter method가 정의되어 있어야 함
  - Property와 setter method의 선언은 JavaBeans 규약을 따름
     - private String name; // property 선언
     - public void setName(String s) { … } // property에 대한 setter 선언
- 설정 방법
  - <property> 태그를 사용하여 값이나 의존 객체 지정
    - <value>값</value>: int, double, String 등 기본 데이터 타입의 값을 전달
    - <ref bean=“bean 식별자” />: bean 속성을 통해 의존 객체를 전달
  - 값이나 bean 객체를 <property>의 속성 형태로도 지정 가능
    - <property value=“값” />, <property ref=“bean 식별자” />
- 효과
  - Container는 property로 전달할 의존 객체를 먼저 생성
  - 각 property에 대한 setter method를 실행하여 객체나 값을 전달

** Anonymous bean 객체 생성 및 주입
- <constructor-arg>나 <property>에서 <ref> 또는 ref 속성을 통해 외부 bean 객체를 지정하지 않고 직접 필요한 의존 객체를 정의
  - <bean> element 중첩 사용
- 내부에 선언된 anonymous bean은 식별자(id)를 갖지 않음
  - 공유(재사용) 불가
