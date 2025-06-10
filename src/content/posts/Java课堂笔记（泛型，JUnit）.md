---
title: Java笔记
published: 2025-04-03
updated: 2025-06-10
description: '泛型，JUnit实际使用'
image: ''
tags: [Java]
category: 'Java'
draft: false 
---

# Java笔记

## 泛型

### 使用案例

我们先来看一个例子

运用集合来存储狗

```java
public class Generic_ {
    public static void main(String[] args) {
        ArrayList list = new ArrayList();
        list.add(new Dog("旺财", 19));
        list.add(new Dog("来福", 33));
        list.add(new Dog("小黑", 12));
        Iterator iterator = list.iterator();
        while (iterator.hasNext()) {
            Dog d = (Dog) iterator.next();
            System.out.println(d);
        }
    }
}

class Dog {
    String name;
    int age;

    public Dog(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

可以通过迭代器遍历所有的狗，但是加入程序员不小心放入了一只猫，程序就会因为类型转化错误而抛出异常

```cmd
Exception in thread "main" java.lang.ClassCastException: com.Cat cannot be cast to com.Dog
	at com.Generic_.main(Generic_.java:19)
```

这样的情况下我们可以使用instanceof加上类型转换来进行处理

```java
public class Generic_ {
    public static void main(String[] args) {
        ArrayList list = new ArrayList();
        list.add(new Dog("旺财", 19));
        list.add(new Dog("来福", 33));
        list.add(new Dog("小黑", 12));
        list.add(new Cat("小花", 11));
        Iterator iterator = list.iterator();
        while (iterator.hasNext()) {
            Object obj = iterator.next();
            Object animal;
            if(obj instanceof Dog){
                animal = (Dog) obj;
            } else {
                animal = (Cat) obj;
            }
            System.out.println(animal);
        }
    }
}
```

但是如果我们后续有不小心放入了一只羊，我们就还要去改进代码逻辑

为了解决这个问题，我们引入了泛型，以约束传入的类型，使得在放入了不同于Dog类型的元素的时候就会发生编译器的编译警告，而不是在运行的时候报错

```java
public class Generic_ {
    public static void main(String[] args) {
        ArrayList<Dog> list = new ArrayList<Dog>();
        list.add(new Dog("旺财", 19));
        list.add(new Dog("来福", 33));
        list.add(new Dog("小黑", 12));
        //list.add(new Cat("黑猫", 12));编译出错
        for(Dog d : list) {
            System.out.println(d);
        }
    }
}

class Dog {
    String name;
    int age;

    public Dog(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

而且可以通过增强的for循环来直接取出Dog对象，使得遍历更加的方便

*泛型的好处*

1.在编译的时候会检查元素的类型，提升了安全性

2.减少了类型的转换次数，提升了效率

详细说明：在不使用泛型的时候，放入的时候需要转为Object放入，而取出时通过Object取出引用，需要进行类型的转换。在使用泛型的时候，放入和取出的时候，不需要类型转化

3.不再提示编译警告（无需加入SuppressWarnings的注解）



### 泛型的定义

泛型是一种可以表示数据类型的数据类型，由程序员来指定

1.泛型又称为参数化类型，是Jdk5.0出现的新特性，解决数据类型的安全问题

2.在类声明或者实例化时只需要指定好需要的具体类型即可

3.Java泛型可以保证如果程序在编译时发出警告，而不是运行时出现异常，同时代码更加的简介健壮

4.泛型的作用是，可以在类声明时通过一个标识表示类中的某个属性的类型，或者是某个方法的返回值类型，或者是参数类型

```java
public class Generic03 {
    public static void main(String[] args) {
        Person<String> person = new Person<String>("礼物");
        Person<Integer> person1 = new Person<Integer>(1);
    }
}

class Person<E> {
    E s;
    //E表示的是 s的数据类型，该数据类型在定义Person对象时指定
    //即在编译期间就确定E的类型

    public Person(E s) {
        this.s = s;
    }

    public E f() {
        return s;
    }
}
```

E表示的是 s的数据类型，该数据类型在定义Person对象时指定，即在编译期间就确定E的类型

```java
public class Generic03 {
    public static void main(String[] args) {
        Person<String> person = new Person<String>("礼物");
        person.classReturn();//class java.lang.String
        Person<Integer> person1 = new Person<Integer>(1);
        person1.classReturn();//class java.lang.Integer
    }
}

class Person<E> {
    E s;

    public Person(E s) {
        this.s = s;
    }

    public void classReturn() {
        System.out.println(s.getClass());
    }
}
```

### 泛型的语法

#### 声明

```java
interface 接口<E> {}
class 类<E,F> {}
```

#### 实例化

```java
List<String> strList = new ArrayList<String>();
Iterator<Customer> iterator = customers.iterator();
```

#### 例题

总而言之在创建泛型集合的时候，传入的泛型类型就会在源码中赋给所有的泛型变量，在实际使用的时候运用var补全即可方便的获取迭代器、EntrySet

```java
public class Generic03 {
    public static void main(String[] args) {
        HashSet<Student> set= new HashSet<Student>();
        Student person1 = new Student("李明", 19);
        Student person2 = new Student("王名", 29);
        Student person3 = new Student("王王", 9);
        set.add(person1);
        set.add(person2);
        set.add(person3);
        Iterator<Student> it = set.iterator();
        while(it.hasNext()) {
            Student student = it.next();
            System.out.println(student);
        }

        for(Student student : set){
            System.out.println(student);
        }
    }
}

class Student {
    String name;
    int age;

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "学生 " + name + " " + age;
    }
}
```

HashMap来使用泛型

```java
public class Generic03 {
    public static void main(String[] args) {
        HashMap<String, Student> map= new HashMap<String, Student>();
        Student person1 = new Student("李明", 19);
        Student person2 = new Student("王名", 29);
        Student person3 = new Student("王王", 9);
        map.put("学生1", person1);
        map.put("学生2", person2);
        map.put("学生3", person3);
        Set<Map.Entry<String, Student>> entrySet = map.entrySet();
        Iterator<Map.Entry<String, Student>> iterator = entrySet.iterator();
        while(iterator.hasNext()) {
            Map.Entry<String, Student> entry = iterator.next();
            String lable = entry.getKey();
            Student student = entry.getValue();
            System.out.println(lable + student);
        }
    }
}

class Student {
    String name;
    int age;

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return " " + name + " " + age;
    }
}
```

### 泛型的使用细节

1.泛型里面的类型只能够是引用数据类型，不可以传入基本数据类型

```java
public class Generic04 {
    public static void main(String[] args) {
        List<Integer> list = new ArrayList<Integer>();
        //List<int> listInt = new ArrayList<int>();编译错误
    }
}
```

2.在指定泛型具体类型后，可以传入该类型或者该类型的子类型

```java
public class Generic04 {
    public static void main(String[] args) {
        List<A> list = new ArrayList<A>();
        list.add(new A());
        list.add(new AA());
    }
}

class A {}
class AA extends A {}
```

或者自定义一个类型

```java
public class Generic04 {
    public static void main(String[] args) {
        A a = new A();
        AA aa = new AA();
        Test<A> test1 = new Test<A>(a);
        Test<A> test2 = new Test<A>(aa);//传入泛型A的子类型实例引用aa
    }
}

class Test<E> {
    E e;

    public Test(E e) {
        this.e = e;
    }
}

class A {}
class AA extends A {}
```

3.泛型的使用形式

```java
List<Integer> list1 = new ArrayList<Integer>();
List<Integer> list2 = new ArrayList<>();//编译器会自动推断泛型类型

List list3 = new ArrayList();//等价于如下形式
List<Object> list4 = new ArrayList<>();
```

我们可以去证明一下，其中e可已接收任何类型的实例，判断出e被指定为一个Object类型

```java
public class Generic04 {
    public static void main(String[] args) {
        Test test = new Test();
        test.e = 100;
        System.out.println(test.e.getClass());
        test.e = "Hello";
        System.out.println(test.e.getClass());
    }
}

class Test<E> {
    public E e;
}
```

输出：

```cmd
class java.lang.Integer
class java.lang.String
```

### 练习题

一个练习题，实现员工的排序

```java
import java.util.*;

public class Generic04 {
    public static void main(String[] args) {
        Employee person1 = new Employee("丽华", 19100.0, new MyDate(2004, 5, 2));
        Employee person2 = new Employee("李丽", 13900.0, new MyDate(1964, 1, 12));
        Employee person3 = new Employee("王芳", 3900.0, new MyDate(1994, 3, 23));
        Employee person4 = new Employee("王芳", 300.0, new MyDate(1993, 11, 23));
        Employee person5 = new Employee("王芳", 900.0, new MyDate(1994, 3, 23));
        List<Employee> list = new ArrayList<>();
        list.add(person1);
        list.add(person2);
        list.add(person3);
        list.add(person4);
        list.add(person5);

        list.sort(new Comparator<Employee>() {
            @Override
            public int compare(Employee o1, Employee o2) {
                int i = o1.getName().compareTo(o2.getName());
                if (i != 0) {
                    return i;
                }
                return o1.getBirthday().compareTo(o2.getBirthday());
            }
        });

        for (Employee e : list) {
            System.out.println(e);
        }
    }
}

class Employee {
    private String name;
    private double sal;
    private MyDate birthday;


    public Employee(String name, double sal, MyDate birthday) {
        this.name = name;
        this.sal = sal;
        this.birthday = birthday;
    }

    @Override
    public String toString() {
        return name + " 薪水是 " + sal + " 生日是 " + birthday.toString();
    }

    public double getSal() {
        return sal;
    }

    public void setSal(double sal) {
        this.sal = sal;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public MyDate getBirthday() {
        return birthday;
    }

    public void setBirthday(MyDate birthday) {
        this.birthday = birthday;
    }
}

class MyDate implements Comparable<MyDate>{
    private int year;
    private int month;
    private int day;

    public MyDate(int year, int month, int day) {
        this.year = year;
        this.month = month;
        this.day = day;
    }

    public int compareTo(MyDate o2) {
        int year = getYear() - o2.getYear();
        if (year != 0) {
            return year;
        }
        int  month= getMonth() - o2.getMonth();
        if (month != 0) {
            return month;
        }
        int day = getDay() - o2.getDay();
            return day;
    }

    @Override
    public String toString() {
        return "" + year + "-" + month + "-" + day;
    }

    public int getYear() {
        return year;
    }

    public void setYear(int year) {
        this.year = year;
    }

    public int getMonth() {
        return month;
    }

    public void setMonth(int month) {
        this.month = month;
    }

    public int getDay() {
        return day;
    }

    public void setDay(int day) {
        this.day = day;
    }
}
```

## 自定义泛型

### 自定义泛型类

#### 基本语法

```java
class 类名<T,E ...> {
	成员
}
```

#### 使用细节

1.普通成员可以使用泛型（属性，方法）

2.使用泛型的数组，不可以初始化（因为不知道空间大小）

3.静态方法中不可以使用类的泛型

4.泛型类的类型，是在创建对象时确定的（因为创建对象时，需要确定类型）

5.如果在创建对象时没有指定类型，默认为Object

```java
class Tiger<T, R, M> {
    String name;
    T t;
    R r;
    M m;

    public Tiger (String name, T t, R r, M m) {
        this.name = name;
        this.t = t;
        this.r = r;
        this.m = m;
    }

    public T gerT() {
        return this.t;
    }
}
```

```java
public class generic {
    public static void main(String[] args) {
        Tiger<String, Integer, Double> tiger = new Tiger<>("Tiger", "Tiger", 1, 1.1);
    }
}
```

可以使用泛型实现复数相加

```java
public class Complex<T extends Number> {
    private final T real;  // 实部
    private final T imaginary;  // 虚部

    public Complex(T real, T imaginary) {
        this.real = real;
        this.imaginary = imaginary;
    }

    // 复数相加（返回新的 Complex<T>）
    public Complex<T> add(Complex<T> other, BinaryOperator<T> adder) {
        T newReal = adder.apply(this.real, other.real);
        T newImaginary = adder.apply(this.imaginary, other.imaginary);
        return new Complex<>(newReal, newImaginary);
    }

    @Override
    public String toString() {
        return real + " + " + imaginary + "i";
    }
}
```

### 自定义泛型接口

#### 基本语法

```java
interface 接口名<T, R...> {
}
```

#### 使用细节

1.接口中，静态成员也不可以使用泛型

2.泛型的接口类型，在继承接口，或者实现接口时确定

3.没有指定类型，泛型默认为Object

使用样例，这里没什么实际运用价值，只是作为展示如何使用（在源码中有很多这种泛型接口），这里是直接指定接口的泛型，事实上，你也可以去添加类的泛型和接口一样，从而保留泛型

```java
public class Interface_ {
    public static void main(String[] args) {
        SomeNumber num = new SomeNumber("数学", 10);
        System.out.println(num.start(" 难"));
        System.out.println(num.end(10));

    }
}

interface Number<K, V> {
    K start(K k);
    V end(V v);
}

//实现接口的时候，直接指定
class SomeNumber implements Number<String, Integer>{
    String name;
    Integer i;

    public SomeNumber(String name, Integer i) {
        this.name = name;
        this.i = i;
    }

    @Override
    public String start(String s) {
        return name + s;
    }

    @Override
    public Integer end(Integer integer) {
        return i + integer;
    }
}
```

### 自定义泛型方法

#### 基本语法

```java
修饰符 <T, R ...>返回类型 方法名 (参数列表) {
}
```

#### 使用细节

1.泛型方法，可以定义在普通类中，也可以定义在泛型类中

2.当泛型方法被调用的时候，类型会确定

3.public void eat(E e) {}，修饰符后没有<T, R> eat不是泛型方法，只是使用了泛型

```java
public class Function {
    public static void main(String[] args) {
        MyTool.printT("你好");
        MyTool.printT(23);
    }
}

class MyTool {
    public static<T> void printT(T t) {
        System.out.println(t);
    }
}
```

## 泛型的继承

1.泛型不具备继承性，不可以把创建的以子类泛型赋值给父类泛型修饰的类型

2.<?>支持任意泛型类型

3.<? extands A>支持A类以及A类的子类，规定了泛型的上限

4.<? super A>支持A类以及A类的父类，不限于直接父类，规定了泛型的下限

```java
public class Function {
    public static void main(String[] args) {
        List<Object> list1 = new ArrayList<>();
        List<String> list2 = new ArrayList<>();
        List<AA> list3 = new ArrayList<>();
        List<BB> list4 = new ArrayList<>();
        List<CC> list5 = new ArrayList<>();
        printCollection1(list1);
        printCollection1(list2);
        printCollection1(list3);
        printCollection1(list4);
        printCollection1(list5);

        printCollection2(list1);//错误，上限是AA，Object是AA的父类
        printCollection2(list2);//错误，String和AA不存在继承关系
        printCollection2(list3);
        printCollection2(list4);
        printCollection2(list5);

        printCollection3(list1);
        printCollection3(list2);//错误，String和AA不存在继承关系
        printCollection3(list3);
        printCollection3(list4);
        printCollection3(list5);

    }

    public static void printCollection1(List<?> c) {
        for (Object obj : c) {
            System.out.println(obj);
        }
    }

    public static void printCollection2(List<? extends AA> c) {
        for (Object obj : c) {
            System.out.println(obj);
        }
    }

    public static void printCollection3(List<? super CC> c) {
        for (Object obj : c) {
            System.out.println(obj);
        }
    }
}

class AA {

}

class BB extends AA {

}

class CC extends BB {

}
```

## 泛型例题

创建一个泛型类，放入map<String, T>，实现相关方法

```java
package com;
import org.junit.jupiter.api.Test;
import java.util.*;

public class Homework {
    public static void main(String[] args) {

    }

    @Test
    public void test1 () {
        User usr1 = new User(1, 18, "lory");
        User usr2 = new User(2, 38, "marry");
        User usr3 = new User(3, 28, "jack");

        Dao<User> map = new Dao<>();
        map.save("100", usr1);
        map.save("200", usr3);
        map.save("300", usr2);
        System.out.println(map.get("200"));
        List<User> list = map.list();
        for(User usr : list) {
            System.out.println(usr);
        }
    }
}

class Dao<T> {
    private Map<String, T> map = new HashMap<>();

    public void save(String id, T entity) {
        map.put(id, entity);
    }

    public T get(String id) {
        return map.get(id);
    }

    public List<T> list() {
        Set<Map.Entry<String, T>> entrySet = map.entrySet();
        Iterator<Map.Entry<String, T>> iterator = entrySet.iterator();
        ArrayList<T> target = new ArrayList<>();
        while (iterator.hasNext()) {
            Map.Entry<String, T> entry = iterator.next();
            target.add(entry.getValue());
        }
        return target;
    }

    public void delete(String id) {
        map.remove(id);
    }
}

class User {
    private int id;
    private int age;
    private String name;

    public User(int id, int age, String name) {
        this.id = id;
        this.age = age;
        this.name = name;
    }

    @Override
    public String toString() {
        return id + " " + age + " " + name;
    }
}
```



## JUnit单元测试框架

在以往的测试中，我们需要手动的将编写的方法类实例化，在主方法中进行调用，这样做似乎有些复杂了，我们可以通过在需要测试的方法上添加@Test注解，并引入相关的包就可以单独运行方法，甚至支持Debug

![93](../images/93.png)

```java
package com;
import org.junit.jupiter.api.Test;

public class Junit {
    public static void main(String[] args) {

    }

    @Test
    public void m1() {
        System.out.println("m1");//m1
    }

    @Test
    public void m2() {
        System.out.println("m2");//m2
    }
}
```



实际使用

```java
package com.batis.junit;

public class Calculate {
    public int add(int a, int b) {
        return a + b;
    }
    public int sub(int a, int b) {
        return a - b;
    }
}
```

通过断言式进行判别

```java
public class CalculateTest {
    @Test
    public void testAdd() {
        int actual = Calculate.add(1, 2);
        int expected = 3;
        Assert.assertEquals(expected, actual);
    }

    @Test
    public void testSub() {
        int actual = Calculate.sub(1, 2);
        int expected = -1;
        Assert.assertEquals(expected, actual);
    }
}
```

如果结果不一致抛出异常

```
java.lang.AssertionError: 
预期:-2
实际:-1
```

