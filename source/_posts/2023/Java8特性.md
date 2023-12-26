---
title: 'Java8特性'
author: ehzyil
tags:
  - Java
categories:
  - 技术
date: 2023-10-09
---
## functional interface 函数式接口

### 函数式接口介绍

**定义**：也称 SAM 接口，即 Single Abstract Method interfaces，有且只有一个抽象方法，但可以有多个非抽象方法的接口。

在 java 8 中专门有一个包放函数式接口`java.util.function`，该包下的所有接口都有 `@FunctionalInterface` 注解，提供函数式编程。

在其他包中也有函数式接口，其中一些没有`@FunctionalInterface` 注解，但是只要符合函数式接口的定义就是函数式接口，与是否有`@FunctionalInterface`注解无关，注解只是在编译时起到强制规范定义的作用。其在 Lambda 表达式中有广泛的应用。

### Java内置的函数式接口介绍及使用举例



| 函数式接口                       | 参数类型 | 返回类型 | 用途                                                         |
| -------------------------------- | -------- | -------- | ------------------------------------------------------------ |
| Consumer 消费型接口              | T        | void     | 对类型为T的对象应用操作，包含方法：void accept(T t)          |
| Supplier 供给型接口              | 无       | T        | 返回类型为T的对象，包含方法：T get()                         |
| Function函数型接口               | T        | R        | 对类型为T的对象应用操作，并返回结果。结果是R类型的对象。包含方法：R apply(T t) |
| Predicate断定型接口              | T        | boolean  | 确定类型为T的对象是否满足某约束，并返回boolean 值。包含方法：boolean test(T t) |
| BiFunction                       | T, U     | R        | 对类型为T,U参数应用操作，返回R类型的结果。包含方法为：Rapply(T t,U u); |
| UnaryOperator(Function子接口)    | T        | T        | 对类型为T的对象进行一元运算，并返回T类型的结果。包含方法为：Tapply(T t); |
| BinaryOperator(BiFunction子接口) | T,T      | T        | 对类型为T的对象进行二元运算，并返回T类型的结果。包含方法为：Tapply(T t1,T t2); |
| BiConsumer                       | T,U      | void     | 对类型为T,U参数应用操作。包含方法为：voidaccept(Tt,Uu)       |
| BiPredicate                      | T,U      | boolean  | 包含方法为：booleantest(Tt,Uu)                               |
| ToIntFunction                    | T        | int      | 计算int值的函数                                              |
| ToLongFunction                   | T        | long     | 计算long值的函数                                             |
| ToDoubleFunction                 | T        | double   | 计算double值的函数                                           |
| IntFunction                      | int      | R        | 参数为int类型的函数                                          |
| LongFunction                     | long     | R        | 参数为long类型的函数                                         |
| DoubleFunction                   | double   | R        | 参数为double类型的函数                                       |

消费型接口使用举例
```java
//消费型接口使用举例
    public void happyTime(double money, Consumer<Double> consumer) {
        consumer.accept(money);
    }

    @org.junit.jupiter.api.Test
    public void test() {
        //1. 以前的写法
        happyTime(1234, new Consumer<Double>() {
            @Override
            public void accept(Double money) {
                System.out.println("突然想回一趟成都了，机票花费" + money);
            }
        });
        System.out.println("------------------------");

        //2. Lambda表达式，将之前的6行代码压缩到了1行
        happyTime(648, money -> System.out.println("学习太累了，奖励自己一发十连，花费" + money));
    }

    /**
     * 突然想回一趟成都了，机票花费1234.0
     * ------------------------
     * 学习太累了，奖励自己一发十连，花费648.0
     */
```

根据给定的规则，过滤集合中的字符串。此规则由Predicate的方法决定

```java
 //根据给定的规则，过滤集合中的字符串。此规则由Predicate的方法决定
    public List<String> filterString(List<String> strings, Predicate<String> predicate) {
        ArrayList<String> res = new ArrayList<>();
        for (String string : strings) {
            if (predicate.test(string)) res.add(string);
        }
        return res;
    }

    @org.junit.jupiter.api.Test
    public void test2() {
        List<String> strings = Arrays.asList("东京", "西京", "南京", "北京", "天津", "中京");
        //1. 以前的写法
        List<String> list = filterString(strings, new Predicate<String>() {
            @Override
            public boolean test(String s) {
                return s.contains("京");
            }
        });

        System.out.println(list);
        System.out.println("------------------------");

        //2. 现在的写法，函数式接口
        List<String> list1 = filterString(strings, s -> s.contains("京"));
        System.out.println(list1);
    }
```

## Lambda表达式

- Lambda是一个匿名函数，我们可以把Lambda表达式理解为是一段可以传递的代码（将代码像数据一样进行传递）。
- 使用它可以写出更简洁、更灵活的代码。作为一种更紧凑的代码风格，是Java的语言表达能力得到了提升

### Lambda表达式使用举例

- 举例一

  ```JAVA
  @Test
  public void test01(){
      //1. 以前的写法
      Runnable runnable01 = new Runnable() {
          @Override
          public void run() {
              System.out.println("你 的 城 市 好 像 不 欢 迎 我");
          }
      };
      runnable01.run();
      System.out.println("------------------------");
      //2. Lambda表达式
      Runnable runnable02 = () -> System.out.println("所 以 我 只 好 转 身 离 开 了");
      runnable02.run();
  }
  ```
  
- 举例二

  ```JAVA
  @Test
  public void test02(){
      //1. 以前的写法
      Comparator<Integer> comparator01 = new Comparator<Integer>() {
          @Override
          public int compare(Integer o1, Integer o2) {
              return o1.compareTo(o2);
          }
      };
      System.out.println(comparator01.compare(95, 27));
      System.out.println("------------------------");
  
      //2. Lambda表达式
      Comparator<Integer> comparator02 = (o1,o2) -> o1.compareTo(o2);
      System.out.println(comparator02.compare(12, 21));
      System.out.println("------------------------");
  
      //3. 方法引用
      Comparator<Integer> comparator03 = Integer::compareTo;
      System.out.println(comparator03.compare(20, 77));
  }
  ```

### Lambda表达式语法的使用

1. 举例： (o1,o2) -> Integer.compare(o1,o2);

2. 格式：

   - `->：`lambda操作符或箭头操作符
   - `->左边：`lambda形参列表（其实就是接口中的抽象方法的形参列表）
   - `->右边：`lambda体（其实就是重写的抽象方法的方法体）

3. Lambda表达式的使用：（分为6种情况介绍）

   语法格式一：**无参无返回值**

   ```JAVA
   @Test
   public void test01(){
       //1. 以前的写法
       Runnable runnable01 = new Runnable() {
           @Override
           public void run() {
               System.out.println("你 的 城 市 好 像 不 欢 迎 我");
           }
       };
       runnable01.run();
       System.out.println("-------------------------");
       //2. Lambda表达式
       Runnable runnable02 = () -> System.out.println("所 以 我 只 好 转 身 离 开 了");
       runnable02.run();
   }
   ```

   语法格式二：**Lambda需要一个参数，但是没有返回值**

   ```JAVA
   @Test
   public void test03(){
       //1. 以前的写法
       Consumer<String> consumer01 = new Consumer<String>() {
           @Override
           public void accept(String s) {
               System.out.println(s);
           }
       };
       consumer01.accept("其实我存过你照片 也研究过你的星座");
   
       System.out.println("-------------------------");
       //2. Lambda表达式
       Consumer<String> consumer02 = (String s) -> {System.out.println(s);};
       consumer02.accept("你喜欢的歌我也会去听 你喜欢的事物我也会想去了解");
   }
   ```

   语法格式三：

   数据类型可以省略，因为可由`类型推断`得出

   ```JAVA
   
   @Test
   public void test04(){
       //1. 以前的写法
       Consumer<String> consumer01 = new Consumer<String>() {
           @Override
           public void accept(String s) {
               System.out.println(s);
           }
       };
       consumer01.accept("我远比表面上更喜欢你");
   
       System.out.println("-------------------------");
       //2. Lambda表达式
       Consumer<String> consumer02 = (s) -> {System.out.println(s);};
       consumer02.accept("但我没有说");
   }
   ```

   语法格式四：

   Lambda若只需要一个参数，参数的小括号可以省略

   ```JAVA
   
   @Test
   public void test04(){
       //1. 以前的写法
       Consumer<String> consumer01 = new Consumer<String>() {
           @Override
           public void accept(String s) {
               System.out.println(s);
           }
       };
       consumer01.accept("我远比表面上更喜欢你");
   
       System.out.println("-------------------------");
       //2. Lambda表达式
       Consumer<String> consumer02 = s -> {System.out.println(s);};
       consumer02.accept("但我没有说");
   }
   ```

   语法格式五：

   Lambda需要两个或以上参数，多条执行语句，并且有返回值

   ```JAVA
   
   @Test
   public void test02() {
       //1. 以前的写法
       Comparator<Integer> comparator01 = new Comparator<Integer>() {
           @Override
           public int compare(Integer o1, Integer o2) {
               System.out.println(o1);
               System.out.println(o2);
               return o1.compareTo(o2);
           }
       };
       System.out.println(comparator01.compare(95, 27));
       System.out.println("-------------------------");
   
       //2. Lambda表达式
       Comparator<Integer> comparator02 = (o1, o2) -> {
           System.out.println(o1);
           System.out.println(o2);
           return o1.compareTo(o2);
       };
       System.out.println(comparator02.compare(12, 21));
   }
   ```

   语法格式六：

   当Lambda体只有一条语句时，return与{}若有，则都可以省略

   ```JAVA
   
   public void test02() {
       //1. 以前的写法
       Comparator<Integer> comparator01 = new Comparator<Integer>() {
           @Override
           public int compare(Integer o1, Integer o2) {
               return o1.compareTo(o2);
           }
       };
       System.out.println(comparator01.compare(95, 27));
       System.out.println("-------------------------");
   
       //2. Lambda表达式
       Comparator<Integer> comparator02 = (o1, o2) -> o1.compareTo(o2);
       System.out.println(comparator02.compare(12, 21));
   }
   ```



## 方法引用和构造器引用

- 当要传递给Lambda体的操作，已经有实现的方法了，可以使用方法引用
- 方法引用可以看做会Lambda表达式的深层次表达，换句话说，方法引用就是Lambda表达式，也就是函数式接口的一个实例，通过方法的名字来指向一个方法，可以认为是Lambda表达式的一个语法糖
- 要求：实现接口的抽象方法的参数列表和返回值类型，必须与方法引用的方法的参数列表和返回值类型保持一致



### 方法引用的使用情况

- 方法引用的使用
  1. 使用情境：当要传递给Lambda体的操作，已经有实现的方法了，可以使用方法引用！
  2. 方法引用，本质上就是Lambda表达式，而Lambda表达式作为函数式接口的实例。所以方法引用，也是函数式接口的实例。
  3. 使用格式： 类(或对象) :: 方法名
  4. 具体分为如下的三种情况：
     - 对象::实例方法名
     - 类::静态方法名
     - 类::实例方法名
  5. 方法引用使用的要求：要求接口中的抽象方法的形参列表和返回值类型与方法引用的方法的形参列表和返回值类型相同！（针对于情况1和情况2）

先写一个实体类

```java
@Data
public class Student {
    private  String name;
    private Integer id;
}
```

情况一：`对象::实例方法名，`抽象方法`的`形参列表`和`返回值类型`与`方法引用`的方法的`形参列表`和`返回值类型相同

```java
    //情况一：对象::实例方法名，抽象方法的形参列表和返回值类型与方法引用的方法的形参列表和返回值类型相同
    //Consumer中的void accept(T t)
    //PrintStream中的void println(T t)
    //形参列表均为(T t)，返回值均为void，可以使用方法引用
    @org.junit.jupiter.api.Test
    public void test03() {
        //1. Lambda
        Consumer<String> consumer01 = s -> System.out.println(s);
        consumer01.accept("她的手只有我的手四分之三那麼大");

        System.out.println("-----------------------------");

        //2. 方法引用
        PrintStream printStream = System.out;
        Consumer<String> consumer02 = printStream::println;
        consumer02.accept("\"可我還是沒能抓住\"");

        System.out.println("-----------------------------");

        //3. 但貌似也可以这么写
        Consumer<String> consumer03 = System.out::println;
        consumer03.accept("\"花落下的时候没死 风捡起花 又丢下 花才死了\"");
    }

    /**
     * 她的手只有我的手四分之三那麼大
     * -----------------------------
     * "可我還是沒能抓住"
     * -----------------------------
     * "花落下的时候没死 风捡起花 又丢下 花才死了"
     */
```

情况二：类 :: 静态方法

```java
    //情况二：类 :: 静态方法

    //Comparator中的int compare(T t1,T t2)
    //Integer中的int compare(T t1,T t2)
    //形参列表均为`(T t1,T t2)`，返回值均为`int`，可以使用方法引用
    @org.junit.jupiter.api.Test
    public void test04() {
        //1. Lambda
        Comparator<Integer> comparator01 = (o1, o2) -> Integer.compare(01, 02);
        System.out.println(comparator01.compare(20, 30));

        System.out.println("----------------------------");

        //2. 方法引用
        Comparator<Integer> comparator02 = Integer::compare;
        System.out.println(comparator02.compare(64, 30));

    }

    /**
     * -1
     * ----------------------------
     * 1
     */

    //Function中的R apply(T t)
    //Math中的Long round(Double d)
    //返回值和参数列表为泛型，也可以匹配上，可以使用方法引用
    @org.junit.jupiter.api.Test
    public void test05() {
        //1. Lambda
        Function<Double, Long> function01 = aDouble -> Math.round(aDouble);
        System.out.println(function01.apply(3.1415926));

        System.out.println("----------------------------");

        //2. 方法引用

        Function<Double, Long> function02 = Math::round;
        System.out.println(function02.apply(0.876));
    }

    /**
     * 3
     * ----------------------------
     * 1
     */
```





情况三：类 :: 实例方法

```java
    //情况三：类 :: 实例方法
    // Comparator中的int comapre(T t1,T t2)
    // String中的int t1.compareTo(t2)
    @org.junit.jupiter.api.Test
    public void test06() {
        //1. Lambda
        Comparator<Integer> comparator01 = (o1, o2) -> o1.compareTo(o2);
        System.out.println(comparator01.compare(94, 21));

        System.out.println("---------------------------");

        //2. 方法引用
        Comparator<Integer> comparator02 = Integer::compareTo;
        System.out.println(comparator02.compare(43, 96));
    }


    //BiPredicate中的boolean test(T t1, T t2);
    //String中的boolean t1.equals(t2)
    @org.junit.jupiter.api.Test
    public void test10() {
        //1. Lambda
        BiPredicate<String, String> biPredicate01 = (o1, o2) -> o1.equals(o2);
        System.out.println(biPredicate01.test("Kyle", "Kyle"));

        System.out.println("----------------------------------");

        //2. 方法引用
        BiPredicate<String, String> biPredicate02 = String::equals;
        System.out.println(biPredicate02.test("Viole", "Violet"));
    }
    /**
     * true
     * ----------------------------------
     * false
     */

    // Function中的R apply(T t)
    // Employee中的String toString();
    @org.junit.jupiter.api.Test
    public void test7() {
        Student student = new Student("Kyle", 9527);
        //1. Lambda
        Function<Student, String> function01 = stu -> stu.toString();
        System.out.println(function01.apply(student));

        System.out.println("------------------------------");

        //2. 方法引用
        Function<Student, String> function02 = Student::toString;
        System.out.println(function02.apply(student));
    }
    /**
     * Student(name=Kyle, id=9527)
     * ------------------------------
     * Student(name=Kyle, id=9527)
     */
```

### 构造器引用和数组引用的使用

- 与函数式接口相结合，自动与函数式接口中方法兼容。
- 可以把构造器引用赋值给定义的方法，要求构造器参数列表要与接口中抽象方法的参数列表一致！且方法的返回值即为构造器对应类的对象。

1.构造器引用

- 和方法引用类似，函数式接口的抽象方法的形参列表和构造器的形参列表一致。

- 抽象方法的返回值类型即为构造器所属的类的类型

  ```JAVA
  @org.junit.jupiter.api.Test
      public void test8() {
  
          //1. Lambda
          BiFunction<String, Integer, Student> biFunction01 = (string, integer) -> new Student(string, integer);
          System.out.println(biFunction01.apply("Kyle", 9527));
  
          System.out.println("------------------------------");
  
          //2. 方法引用
          BiFunction<String, Integer, Student> biFunction02 = Student::new;
          System.out.println(biFunction02.apply("Lucy", 9421));
      }
  
      /**
       * Student(name=Kyle, id=9527)
       * ------------------------------
       * Student(name=Lucy, id=9421)
       */
  ```

2.数组引用

- 可以把数组看做是一个特殊的类，则写法与构造器引用一致

  ```JAVA
  
      @org.junit.jupiter.api.Test
      public void test9() {
          //1. Lambda 创建一个指定长度的string数组
          Function<Integer, String[]> function01 = integer -> new String[integer];
          System.out.println(Arrays.toString(function01.apply(5)));
  
          System.out.println("-----------------------------");
          //2. 数组引用
          Function<Integer, String[]> function02 = String[]::new;
          System.out.println(Arrays.toString(function02.apply(7)));
      }
      /**
       * [null, null, null, null, null]
       * -----------------------------
       * [null, null, null, null, null, null, null]
       */
  ```

## Stream API

  ### Stream API概述

  - Java8中有两个最为重要的改变，第一个就是Lambda表达式，另外一个则是Stream API
  - Stream API(java.util.stream)把真正的函数式编程风格引入到Java中，这是目前为止对Java类库最好的补充，因为Stream API可以极大地提高程序员生产力，让程序员写出高效、简洁的代码
  - Stream是Java8 中处理集合的关键抽象概念，它可以指定你希望对集合进行的操作，可以执行非常复杂的查找、过滤和映射数据等操作。
  - 使用Stream API 对集合数据进行操作，就类似于使用SQL 执行的数据库查询，也可以使用Stream API 来并行执行操作。简言之，Stream API 提供了一种高效且易于使用的处理数据的方式
  - 为什么要使用Stream API
    - 实际开发中，项目中多数数据源都是来自MySQL、Oracle 等。但现在数据源可以更多了，有MongDB、Redis等，而这些NoSQL的数据就需要Java层面去处理。我就是学完Redis再来补票的..
    - Stream 和Collection 集合的区别：Collection 是一种静态的内存数据结构，而Stream 是有关计算的。前者是主要面向内存，存储在内存中，后者主要是面向CPU，通过CPU 实现计算（这也就是为什么一旦执行终止操作之后，Stream 就不能被再次使用，得重新创建一个新的流才行）
  - 小结
    1. Stream 关注的是对数据的运算，与CPU 打交道；
       集合关注的是数据的存储，与内存打交道
    2. Stream 自己不会存储数据；
       Stream 不会改变源对象，相反，他们会返回一个持有结果的新Stream
       Stream 操作是延迟执行的，这意味着他们会等到需要结果的时候才执行
    3. Stream 执行流程
       - Stream实例化
       - 一系列中间操作（过滤、映射、..）
       - 终止操作
    4. 说明
       - 一系列中间操作链，对数据源的数据进行处理
       - 一旦执行终止操作，就执行中间操作链，并产生结果，之后，不会再被使用

  ### Stream的实例化

先新建两个类

```java
public class Employee {

    private int id;
    private String name;
    private int age;
    private double salary;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public double getSalary() {
        return salary;
    }

    public void setSalary(double salary) {
        this.salary = salary;
    }

    public Employee() {
        System.out.println("Employee().....");
    }

    public Employee(int id) {
        this.id = id;
        System.out.println("Employee(int id).....");
    }

    public Employee(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public Employee(int id, String name, int age, double salary) {

        this.id = id;
        this.name = name;
        this.age = age;
        this.salary = salary;
    }

    @Override
    public String toString() {
        return "Employee{" + "id=" + id + ", name='" + name + '\'' + ", age=" + age + ", salary=" + salary + '}';
    }

    @Override
    public boolean equals(Object o) {
        if (this == o)
            return true;
        if (o == null || getClass() != o.getClass())
            return false;

        Employee employee = (Employee) o;

        if (id != employee.id)
            return false;
        if (age != employee.age)
            return false;
        if (Double.compare(employee.salary, salary) != 0)
            return false;
        return name != null ? name.equals(employee.name) : employee.name == null;
    }

    @Override
    public int hashCode() {
        int result;
        long temp;
        result = id;
        result = 31 * result + (name != null ? name.hashCode() : 0);
        result = 31 * result + age;
        temp = Double.doubleToLongBits(salary);
        result = 31 * result + (int) (temp ^ (temp >>> 32));
        return result;
    }
}
```

```java
/**
 * 提供用于测试的数据
 */
public class EmployeeData {

    public static List<Employee> getEmployees() {
        List<Employee> list = new ArrayList<>();

        list.add(new Employee(1001, "马化腾", 34, 6000.38));
        list.add(new Employee(1002, "马云", 12, 9876.12));
        list.add(new Employee(1003, "刘强东", 33, 3000.82));
        list.add(new Employee(1004, "雷军", 26, 7657.37));
        list.add(new Employee(1005, "李彦宏", 65, 5555.32));
        list.add(new Employee(1006, "比尔盖茨", 42, 9500.43));
        list.add(new Employee(1007, "任正非", 26, 4333.32));
        list.add(new Employee(1008, "扎克伯格", 35, 2500.32));

        return list;
    }
}
```



- 创建Stream`方式一：`通过集合


  ```java
      @org.junit.jupiter.api.Test
      public void test() {
          List<Employee> employees= EmployeeData.getEmployees();
  
          //default Stream<E> stream()  返回一个顺序流
          Stream<Employee> stream = employees.stream();
  
          //default Stream<E> parallelStream  返回一个并行流
          Stream<Employee> employeeStream=employees.parallelStream();
      }
  ```

- 创建Stream`方式二：`通过数组

 ```java
     @org.junit.jupiter.api.Test
     public void test2() {
         int[] arr = new int[]{1, 3, 5, 7, 9, 2, 4, 6, 8};
         //调用Arrays的static <T> Stream<T> stream(T[] array)   返回一个流
         IntStream stream=Arrays.stream(arr);
 
         Employee kyle=new Employee(9527,"Kyle");
         Employee lucy=new Employee(9527,"Lucy");
         Employee[] employees={kyle,lucy};
 
         Stream<Employee> stream1=Arrays.stream(employees);
 
     }
 ```

- 
  创建Stream`方式三：`通过Stream的of()


  ```java
     @org.junit.jupiter.api.Test
      public void test03() {
          Stream<Integer> stream=Stream.of(9,4,2,8,2,5,2,7);
      }
  ```

- 创建Stream`方式四：`创建无限流

  如果不用limit限制输出，则会一直输出下去，forEach就相当于是终止操作


```java
    @org.junit.jupiter.api.Test
    public void test04() {
        // 迭代
        // 遍历前10个数
        // public static<T> Stream<T> iterate(final T seed, final UnaryOperator<T> f)
        Stream.iterate(0,t->t+1).limit(10).forEach(System.out::println);

        // 生成
        // 10个随机数
        // public static<T> Stream<T> generate(Supplier<T> s)
        Stream.generate(Math::random).limit(10).forEach(System.out::println);
    }

```




### Stream的中间操作：筛选与切片
- 多个中间操作可以连接起来形成一个流水线，除非流水线上触发终止操作，否则中间操作不会执行任何的处理，而在终止操作时一次性全部处理，称为`惰性求值`

  |        方法         |                             描述                             |
  | :-----------------: | :----------------------------------------------------------: |
  | filter(Predicate p) |               接收Lambda ，从流中排除某些元素                |
  |     distinct()      |  筛选，通过流所生成元素的hashCode() 和equals() 去除重复元素  |
  | limit(long maxSize) |                截断流，使其元素不超过给定数量                |
  |    skip(long n)     | 跳过元素，返回一个扔掉了前n 个元素的流。若流中元素不足n 个，则返回一个空流。与limit(n)互补 |
  
  ```java
      @org.junit.jupiter.api.Test
      public void test05() {
  
          List<Employee> employees = EmployeeData.getEmployees();
          //1. filter 查询工资大于7000的员工信息
          employees.stream().filter(employee -> employee.getSalary()>7000).forEach(System.out::println);
          System.out.println("----------------------------");
          //2. limit(n)——截断流，使其元素不超过给定数量。只输出3条员工信息
          employees.stream().limit(3).forEach(System.out::println);
          System.out.println("----------------------------");
          //3.  skip(n) —— 跳过元素，返回一个扔掉了前 n 个元素的流。若流中元素不足 n 个，则返回一个空流。与 limit(n) 互补
          employees.stream().skip(3).forEach(System.out::println);
          System.out.println("----------------------------");
          //4. distinct()——筛选，通过流所生成元素的 hashCode() 和 equals() 去除重复元素
          employees.add(new Employee(9527, "Kyle", 20, 9999));
          employees.add(new Employee(9527, "Kyle", 20, 9999));
          employees.add(new Employee(9527, "Kyle", 20, 9999));
          employees.add(new Employee(9527, "Kyle", 20, 9999));
          employees.add(new Employee(9527, "Kyle", 20, 9999));
          employees.stream().distinct().forEach(System.out::println);
      }
  
      /**
       * Employee{id=1002, name='马云', age=12, salary=9876.12}
       * Employee{id=1004, name='雷军', age=26, salary=7657.37}
       * Employee{id=1006, name='比尔盖茨', age=42, salary=9500.43}
       * ----------------------------
       * Employee{id=1001, name='马化腾', age=34, salary=6000.38}
       * Employee{id=1002, name='马云', age=12, salary=9876.12}
       * Employee{id=1003, name='刘强东', age=33, salary=3000.82}
       * ----------------------------
       * Employee{id=1004, name='雷军', age=26, salary=7657.37}
       * Employee{id=1005, name='李彦宏', age=65, salary=5555.32}
       * Employee{id=1006, name='比尔盖茨', age=42, salary=9500.43}
       * Employee{id=1007, name='任正非', age=26, salary=4333.32}
       * Employee{id=1008, name='扎克伯格', age=35, salary=2500.32}
       * ----------------------------
       * Employee{id=1001, name='马化腾', age=34, salary=6000.38}
       * Employee{id=1002, name='马云', age=12, salary=9876.12}
       * Employee{id=1003, name='刘强东', age=33, salary=3000.82}
       * Employee{id=1004, name='雷军', age=26, salary=7657.37}
       * Employee{id=1005, name='李彦宏', age=65, salary=5555.32}
       * Employee{id=1006, name='比尔盖茨', age=42, salary=9500.43}
       * Employee{id=1007, name='任正非', age=26, salary=4333.32}
       * Employee{id=1008, name='扎克伯格', age=35, salary=2500.32}
       * Employee{id=9527, name='Kyle', age=20, salary=9999.0}
       *
       */
  ```

### Stream的中间操作:映射

|              方法               |                             描述                             |
| :-----------------------------: | :----------------------------------------------------------: |
|         map(Function f)         | 接收一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。 |
| mapToDouble(ToDoubleFunction f) | 接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的DoubleStream。 |
|    mapToInt(ToIntFunction f)    | 接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的IntStream。 |
|   mapToLong(ToLongFunction f)   | 接收一个函数作为参数，该函数会被应用到每个元素上，产生一个新的LongStream。 |
|       flatMap(Function f)       | 接收一个函数作为参数，将流中的每个值都换成另一个流，然后把所有流连接成一个流 |

```java
    @org.junit.jupiter.api.Test
    public void test06() {
        List<String> strings = Arrays.asList("aa", "bb", "cc", "dd");
        List<Employee> employees = EmployeeData.getEmployees();
        //map(Function f)——接收一个函数作为参数，将元素转换成其他形式或提取信息，该函数会被应用到每个元素上，并将其映射成一个新的元素。
        // 练习：将字符串转为大写并输出
        strings.stream().map(String::toUpperCase).forEach(System.out::println);//方法引用
        System.out.println("--------------------------");
        strings.stream().map(s -> s.toUpperCase()).forEach(System.out::println);//lambda

        System.out.println("--------------------------");
        // 练习：获取员工姓名长度大于3的员工的姓名。
        employees.stream().map(Employee::getName).filter(name->name.length()>3).forEach(System.out::println);

        System.out.println("--------------------------");
        // 练习：将字符串中的多个字符构成的集合转换为对应的Stream实例
        strings.stream().map(Test::formStringToStream).forEach(characterStream -> characterStream.forEach(System.out::println));

        // 使用flatMap(Function f)达到同样的效果
        strings.stream().flatMap(Test::formStringToStream).forEach(System.out::println);
    }

    /**
     *AA
     * BB
     * CC
     * DD
     * --------------------------
     * AA
     * BB
     * CC
     * DD
     * --------------------------
     * 比尔盖茨
     * 扎克伯格
     * --------------------------
     * a
     * a
     * b
     * b
     * c
     * c
     * d
     * d
     * a
     * a
     * b
     * b
     * c
     * c
     * d
     * d
     *
     */

    public static Stream<Character> formStringToStream(String str) {
        ArrayList<Character> list = new ArrayList<>();
        for (char c : str.toCharArray()) {
            list.add(c);
        }
        return list.stream();
    }
```

### Stream的中间操作:排序

|          方法          |                描述                |
| :--------------------: | :--------------------------------: |
|        sorted()        |  产生一个新流，其中按自然顺序排序  |
| sorted(Comparator com) | 产生一个新流，其中按比较器顺序排序 |

```java
    @org.junit.jupiter.api.Test
    public void test10() {
        List<Integer> nums = Arrays.asList(13, 54, 97, 52, 43, 64, 27);
        List<Employee> employees = EmployeeData.getEmployees();
        //自然排序
        nums.stream().sorted().forEach(System.out::println);
        //定制排序，先按照年龄升序排，再按照工资降序排
        employees.stream().sorted((o1, o2) -> {

            int result = Integer.compare(o1.getAge(), o2.getAge());
            if (result != 0) {
                return result;
            } else {
                return -Double.compare(o1.getSalary(), o2.getSalary());
            }

        }).forEach(System.out::println);
    }

    /**
     * 13
     * 27
     * 43
     * 52
     * 54
     * 64
     * 97
     * Employee{id=1002, name='马云', age=12, salary=9876.12}
     * Employee{id=1004, name='雷军', age=26, salary=7657.37}
     * Employee{id=1007, name='任正非', age=26, salary=4333.32}
     * Employee{id=1003, name='刘强东', age=33, salary=3000.82}
     * Employee{id=1001, name='马化腾', age=34, salary=6000.38}
     * Employee{id=1008, name='扎克伯格', age=35, salary=2500.32}
     * Employee{id=1006, name='比尔盖茨', age=42, salary=9500.43}
     * Employee{id=1005, name='李彦宏', age=65, salary=5555.32}
     *
     */
```

### Stream的中间操作:匹配与查找

- 终端操作会从流的流水线生成结果。其结果可以是任何不是流的值，例如：List、Integer，甚至是void 。

- 流进行了终止操作后，不能再次使用。

  |          方法          |                             描述                             |
  | :--------------------: | :----------------------------------------------------------: |
  | allMatch(Predicate p)  |                     检查是否匹配所有元素                     |
  | anyMatch(Predicate p)  |                   检查是否至少匹配一个元素                   |
  | noneMatch(Predicate p) |                   检查是否没有匹配所有元素                   |
  |      findFirst()       |                        返回第一个元素                        |
  |       findAny()        |                    返回当前流中的任意元素                    |
  |        count()         |                       返回流中元素总数                       |
  |   max(Comparator c)    |                        返回流中最大值                        |
  |   min(Comparator c)    |                        返回流中最小值                        |
  |  forEach(Consumer c)   | 内部迭代(使用Collection 接口需要用户去做迭代，称为外部迭代。相反，Stream API 使用内部迭代——它帮你把迭代做了) |

```java
    @org.junit.jupiter.api.Test
    public void test7() {
        List<Employee> employees = EmployeeData.getEmployees();
        // allMatch(Predicate p)——检查是否匹配所有元素。
        // 练习：是否所有的员工的工资是否都大于5000
        System.out.println("是否所有的员工的工资是否都大于5000：" + employees.stream().allMatch(employee -> employee.getSalary() > 5000));

        // anyMatch(Predicate p)——检查是否至少匹配一个元素。
        // 练习：是否存在员工年龄小于15
        System.out.println("是否存在员工年龄小于15：" + employees.stream().allMatch(employee -> employee.getAge() < 15));

        // noneMatch(Predicate p)——检查是否没有匹配的元素。
        // 练习：是否不存在员工姓“马”
        System.out.println("是否不存在员工姓马：" + employees.stream().noneMatch(employee -> employee.getName().startsWith("马")));

        //findFirst——返回第一个元素
        System.out.println("返回第一个元素：" + employees.stream().findFirst());

        //findAny——返回当前流中的任意元素
        System.out.println("返回当前流中的任意元素" + employees.stream().findAny());

        //count——返回流中元素的总个数
        System.out.println("返回元素总数：" + employees.stream().count());

        //max(Comparator c)——返回流中最大值
        System.out.println("返回最高工资：" + employees.stream().map(Employee::getSalary).max(Double::compare));

        //min(Comparator c)——返回流中最小值
        System.out.println("返回最小年龄：" + employees.stream().map(Employee::getAge).min(Integer::compare));
        
        //forEach(Consumer c)——内部迭代
        employees.stream().forEach(System.out::println);
        System.out.println("-------------");
        ////使用集合的遍历操作
        employees.forEach(System.out::println);
    }

    /**
     * 是否所有的员工的工资是否都大于5000：false
     * 是否存在员工年龄小于15：false
     * 是否不存在员工姓马：false
     * 返回第一个元素：Optional[Employee{id=1001, name='马化腾', age=34, salary=6000.38}]
     * 返回当前流中的任意元素Optional[Employee{id=1001, name='马化腾', age=34, salary=6000.38}]
     * 返回元素总数：8
     * 返回最高工资：Optional[9876.12]
     * 返回最小年龄：Optional[12]
     * Employee{id=1001, name='马化腾', age=34, salary=6000.38}
     * Employee{id=1002, name='马云', age=12, salary=9876.12}
     * Employee{id=1003, name='刘强东', age=33, salary=3000.82}
     * Employee{id=1004, name='雷军', age=26, salary=7657.37}
     * Employee{id=1005, name='李彦宏', age=65, salary=5555.32}
     * Employee{id=1006, name='比尔盖茨', age=42, salary=9500.43}
     * Employee{id=1007, name='任正非', age=26, salary=4333.32}
     * Employee{id=1008, name='扎克伯格', age=35, salary=2500.32}
     * -------------
     * Employee{id=1001, name='马化腾', age=34, salary=6000.38}
     * Employee{id=1002, name='马云', age=12, salary=9876.12}
     * Employee{id=1003, name='刘强东', age=33, salary=3000.82}
     * Employee{id=1004, name='雷军', age=26, salary=7657.37}
     * Employee{id=1005, name='李彦宏', age=65, salary=5555.32}
     * Employee{id=1006, name='比尔盖茨', age=42, salary=9500.43}
     * Employee{id=1007, name='任正非', age=26, salary=4333.32}
     * Employee{id=1008, name='扎克伯格', age=35, salary=2500.32}
     *
     * Process finished with exit code 0
     */
```

### Stream的终止操作:归约

|               方法               |                         描述                         |
| :------------------------------: | :--------------------------------------------------: |
| reduce(T iden, BinaryOperator b) |    可以将流中元素反复结合起来，得到一个值。返回T     |
|     reduce(BinaryOperator b)     | 可以将流中元素反复结合起来，得到一个值。返回Optional |
```java
    @org.junit.jupiter.api.Test
    public void test8() {

        List<Integer> nums = Arrays.asList(13, 32, 23, 31, 94, 20, 77, 21, 17);
        List<Employee> employees = EmployeeData.getEmployees();

        // reduce(T identity, BinaryOperator)——可以将流中元素反复结合起来，得到一个值。返回 T
        // 练习1：计算1-10的自然数的和
        System.out.println(nums.stream().reduce(0,Integer::sum));

        //reduce(BinaryOperator) ——可以将流中元素反复结合起来，得到一个值。返回 Optional<T>
        // 练习2：计算公司所有员工工资总和
        System.out.println(employees.stream().map(Employee::getSalary).reduce((o1,o2)->o1+o2));

        // 别的写法，计算年龄总和
        System.out.println(employees.stream().map(Employee::getAge).reduce(0,Integer::sum));
    }

    /**
     * 328
     * Optional[48424.08]
     * 273
     */
```
### Stream的终止操作:收集

|         方法         |                             描述                             |
| :------------------: | :----------------------------------------------------------: |
| collect(Collector c) | 将流转换为其他形式。接收一个Collector接口的实现，用于给Stream中元素做汇总的方法 |

```java
    @org.junit.jupiter.api.Test
    public void test9() {
        // collect(Collector c)——将流转换为其他形式。接收一个 Collector接口的实现，用于给Stream中元素做汇总的方法
        // 练习1：查找工资大于6000的员工，结果返回为一个List
        List<Employee> employees = EmployeeData.getEmployees();
        List<Employee> collect = employees.stream().filter(employee -> employee.getSalary() > 6000).collect(Collectors.toList());
        collect.forEach(System.out::println);

        System.out.println("--------------------");
        // 练习2：查找年龄大于20的员工，结果返回为一个List
        List<Employee> collect1 = employees.stream().filter(employee -> employee.getAge() > 20).collect(Collectors.toList());
        collect1.forEach(System.out::println);
    }

    /**
     * Employee{id=1001, name='马化腾', age=34, salary=6000.38}
     * Employee{id=1002, name='马云', age=12, salary=9876.12}
     * Employee{id=1004, name='雷军', age=26, salary=7657.37}
     * Employee{id=1006, name='比尔盖茨', age=42, salary=9500.43}
     * --------------------
     * Employee{id=1001, name='马化腾', age=34, salary=6000.38}
     * Employee{id=1003, name='刘强东', age=33, salary=3000.82}
     * Employee{id=1004, name='雷军', age=26, salary=7657.37}
     * Employee{id=1005, name='李彦宏', age=65, salary=5555.32}
     * Employee{id=1006, name='比尔盖茨', age=42, salary=9500.43}
     * Employee{id=1007, name='任正非', age=26, salary=4333.32}
     * Employee{id=1008, name='扎克伯格', age=35, salary=2500.32}
     *
     * Process finished with exit code 0
     */
```





## Optional类

### 简介

- 到目前为止，臭名昭著的空指针异常是导致Java应用程序失败的最常见原因。以前，为了解决空指针异常，Google公司著名的Guava项目引入了Optional类，Guava通过使用检查空值的方式来防止代码污染，它鼓励程序员写更干净的代码。受到Google Guava的启发，Optional类已经成为Java 8类库的一部分。
- Optional 类(java.util.Optional) 是一个容器类，它可以保存类型T的值，代表这个值存在。或者仅仅保存null，表示这个值不存在。原来用null 表示一个值不存在，现在Optional 可以更好的表达这个概念。并且可以避免空指针异常。
- Optional类的Javadoc描述如下：这是一个可以为null的容器对象。如果值存在则isPresent()方法会返回true，调用get()方法会返回该对象。
- Optional提供很多有用的方法，这样我们就不用显式进行空值检测。
- 创建Optional类对象的方法：
  - Optional.of(T t): 创建一个Optional 实例，t必须非空；
  - Optional.empty() : 创建一个空的Optional 实例
  - Optional.ofNullable(T t)：t可以为null
- 判断Optional容器中是否包含对象：
  - boolean isPresent() : 判断是否包含对象
  - void ifPresent(Consumer<? super T> consumer) ：如果有值，就执行Consumer接口的实现代码，并且该值会作为参数传给它。
- 获取Optional容器的对象：
  - T get(): 如果调用对象包含值，返回该值，否则抛异常
  - T orElse(T other) ：如果有值则将其返回，否则返回指定的other对象。
  - T orElseGet(Supplier<? extends T> other) ：如果有值则将其返回，否则返回由Supplier接口实现提供的对象。
  - T orElseThrow(Supplier<? extends X> exceptionSupplier) ：如果有值则将其返回，否则抛出由Supplier接口实现提供的异常。
  
  
### 使用举例

Boy类

```java
public class Boy {
    private Girl girl;

    public Boy() {
    }

    public Boy(Girl girl) {
        this.girl = girl;
    }

    public Girl getGirl() {
        return girl;
    }

    public void setGirl(Girl girl) {
        this.girl = girl;
    }

    @Override
    public String toString() {
        return "Boy{" +
                "girl=" + girl +
                '}';
    }
}
```

Gril类

```java
public class Girl {
    private String name;

    public Girl() {
    }

    public Girl(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    @Override
    public String toString() {
        return "Girl{" +
                "name='" + name + '\'' +
                '}';
    }
}
```

测试类

```java
@Test
public void test(){
    Girl girl = new Girl();
    // girl = null;
    // of(T t):保证t是非空的
    Optional<Girl> optionalGirl = Optional.of(girl);
}

@Test
public void test25(){
    Girl girl = new Girl();
    // girl = null;
    // ofNullable(T t)：t可以为null
    Optional<Girl> optionalGirl = Optional.ofNullable(girl);
    System.out.println(optionalGirl);
    // orElse(T t1):如果单前的Optional内部封装的t是非空的，则返回内部的t.
    // 如果内部的t是空的，则返回orElse()方法中的参数t1.
    Girl girl1 = optionalGirl.orElse(new Girl(""));
    System.out.println(girl1);
}
```





