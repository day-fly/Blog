---
title: JAVA8 - Interface의 default 메소드 파해치기 (feat. 자바의 다중상속)
date: "2019-10-06"
template: "post"
draft: false
slug: "/posts/Java8-interface-default/"
category: "JAVA8"
tags:
  - "JAVA"
description: "Java 버전 8부터 Interface에서 default라는 키워드로 시작하는 메소드를 구현할 수 있게 되었다. 접근제어자인 default와는 다르다. (이미 알고 있겠지만, 자바 Interface에서는 명시하지 않아도 기본적으로 메소드가 Public abstract method이기 때문에 접근제어자는 default가 될 수 없다.)"
socialImage: "/media/image-2.jpg"
---


## 요약
                                            
- JAVA 8부터 Interface에 default method(함수 구현부까지 포함하는) 와 static method를 사용할 수 있다. 
- default 메소드는 method 앞에 default를 붙이며 이는 접근제한자가 아니다. (사실 자바 Interface에서 접근제한자는 무조건 public이고, 생략해도 public이고..)
- default 기능사용으로 사실상 동작에 대한 다중상속(여러 개의 Interface 구현은 원래 되므로) 구현이 가능하다.
- 공개된 Interface에 추상 메소드를 추가하면 소스 호환성이 깨지는데, default 메소드 덕분에 기존 버전과의 호환성 유지에 용이하다.
- 상속받는 다른 클래스나 Interface에서 같은 시그니처를 갖는 default 메소드가 있을 때는 우선권을 가지는 몇 가지 규칙이 있다.
 

## 1. 살펴보기

Java 버전 8부터 Interface에서 default라는 키워드로 시작하는 메소드를 구현할 수 있게 되었다. 접근제어자인 default와는 다르다. (이미 알고 있겠지만, 자바 Interface에서는 명시하지 않아도 기본적으로 메소드가 Public abstract method이기 때문에 접근제어자는 default가 될 수 없다.)

```java
package com.company;

public interface Company {

    void doSomthing();

    default void doDefault(){
        System.out.println("I'm default method");
    }
}
```
<center>[code 1-1] Interface 안에서 default method</center>


실제로 Java API에서 8버전으로 넘어갈 때 여러 부분에 이 default 키워드가 활용되었다. 예로 아래와 같이 List Interface에 sort메소드가 default 메소드로 사용되었다. 이렇게 default 메소드로 구현함으로써 Interface에 섣불리 추가하지 못했던 여러 메소드들을 추가할 수 있게 되었다. 이미 해당 Interface를 구현한 구현체에서 추가적인 작업이 필요없기 때문이다.


```java
public interface List<E> extends Collection<E> {
    //...
    default void sort(Comparator<? super E> c) {
        Object[] a = this.toArray();
        Arrays.sort(a, c);
        ListIterator<E> i = this.listIterator();
        Object[] var4 = a;
        int var5 = a.length;

        for(int var6 = 0; var6 < var5; ++var6) {
            Object e = var4[var6];
            i.next();
            i.set(e);
        }
    }
    //...
}
```
<center>[code 1-2] default 메소드 구현 예 - Java List Interface 의 sort 메소드</center>

이런식으로 구현이 가능하니 템플릿 메소드 패턴을 쓰기 위해 무조건 Abstract Class와 상속을 활용하지 않아도 Interface를 사용할 수 있게 되었다.


## 2. 해석 규칙
이미 알고 있듯이, 자바의 클래스는 하나의 부모 클래스만 상속받을 수 있지만, 여러 Interface를 동시에 구현할 수 있다. 만약 Interface 구현체에서 같은 시그니처를 갖는 default 메소드를 상속받는 상황에선 어떻게 동작할까?

### 상황 1. Interface vs 서브 Interface vs 구현 클래스

```java
public interface Company {
    default void print(){
        System.out.println("I'm Company");
    }
}

public interface KoreaCompany extends Company {
    default void print(){
        System.out.println("I'm Korea Company");
    }
}

public class Samsung implements KoreaCompany, Company{
    public static void main(String[] args){
        new Samsung().print();
    }
}
```
<center>[code 2-1] default 메소드의 우선순위 - Interface vs 서브 Interface</center>


Company와 KoreaCompany에 동일한 시그니처의 default 메소드가 구현되어 있다. KoreaCompany는 Company의 서브 Interface이다. 이러한 상황에서 Samsung Class에서 print메소드는 어떤 메소드를 콜하게 될까?<br>
정답은 서브 Interface의 메소드이다. 즉, "I'm Korea Company" 가 찍히게 된다. 이 상황에서 알 수 있는 사실 하나는 **상속관계를 갖는 Interface에서 같은 시그니처의 default 메소드가 있으면 서브 Interface 메소드가 우선이라는 사실이다.**

그럼 이 상황에서 Samsung Class에 아래와 같이 print 메소드를 구현하면 어떻게 될까?
```java
public class Samsung implements KoreaCompany, Company{

    public void print(){
        System.out.println("I'm Samsung");
    }

    public static void main(String[] args){
        new Samsung().print();
    }
}
```
<center>[code 2-2] default 메소드의 우선순위 - Interface vs 서브 Interface vs 구현 클래스</center>

정답은 "I'm Samsung" 이 찍힌다. 이로써 알 수 있는 사실이 추가되었다. **구현클래스의 메소드가 Interface default 메소드보다 우선이다.**

그럼 상속관계에 있는 슈퍼클래스도 우선권을 가질까? 

```java
public interface Company {
    default void print(){
        System.out.println("I'm Company");
    }
}

public interface KoreaCompany extends Company {
    default void print(){
        System.out.println("I'm Korea Company");
    }
}

public class SuperClassCompany implements Company {
    default void print(){
        System.out.println("I'm Super Class Company");
    }
}

public class Samsung extends SuperClassCompany implements KoreaCompany, Company{
    public static void main(String[] args){
        new Samsung().print();
    }
}
```
<center>[code 2-3] default 메소드 vs 슈퍼클래스</center>

출력 결과는 I'm Super Class Company 이다. 
위에서 알 수 있었던 사실을 보완하면, **구현클래스와 슈퍼클래스의 메소드가 Interface의 default 메소드보다 우선이다.** 


### 상황 2. 같은 레벨의 Interface

위에서는 Interface와 그의 서브 Interface 간의 우선순위에 대해 살펴보았다. 그럼 같은 레벨의 Interface에서 동일한 시그니쳐를 가진 default 메소드가 있는데, 구현 클래스에서 해당 Interface를 모두 구현한다고 하면 어떤 상황이 벌어질까? 

```java
public interface Company {
    default void print(){
        System.out.println("I'm Company");
    }
}

public interface Employee {
    default void print(){
        System.out.println("I'm Employee");
    }
}

public class CompanyEmployee implements Company, Employee{
    public static void main(String[] args){
        new CompanyEmployee().print();
    }
}
```
<center>[code 2-4] 같은 레벨의 Interface에서의 동일 시그니처의 default 메소드</center>

CompanyEmployee클래스에서는 Company와 Employee의 print 메소드를 구별할 기준이 모호하다. 자바 컴파일러는 아래와 같은 에러를 발생시킨다.
> Error:(3, 8) java: types com.company.Company and com.company.Employee are incompatible;
  class com.company.CompanyEmployee inherits unrelated defaults for print() from types com.company.Company and com.company.Employee

이런 상황에서는 메소드를 오버라이드해서 어떤 메소드를 호출할지 명시적으로 선택해줘야 한다. 

```JAVA
public class CompanyEmployee implements Company, Employee{
    public static void main(String[] args){
        new CompanyEmployee().print(); // I'm Company
    }

    @Override
    public void print() {
        Company.super.print(); //명시적으로 Company의 함수 호출 
    }
}
```


알게 된 사실들을 다시 정리해보면 아래와 같다.

**1) 상속관계를 갖는 Interface(Interface의 Interface..) 에서 같은 시그니처의 default 메소드가 있으면 서브 Interface 메소드가 우선이라는 사실이다.**

**2) 구현클래스와 슈퍼클래스의 메소드가 Interface의 default 메소드보다 우선이다.**

**3) 우선순위를 결정하는 못하는 상황에서는 호출할 메소드를 명시해줘야 한다.**






--

