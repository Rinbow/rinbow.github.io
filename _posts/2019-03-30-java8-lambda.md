---
title: Java8 中 lambda 表达式的使用
tags: [java]
---

### Java8 lambda 表达式实例

#### 1. 用 lambda 表达式实现 Runnable

看一下 Java 8 之前的 runnable 实现方法，需要 4 行代码，而使用 lambda 表达式只需要一行代码。我们在这里做了什么呢？那就是用 () -> {} 代码块替代了整个匿名类。

```java
// Java 8 之前：
new Thread(new Runnable() {
    @Override
    public void run() {
    System.out.println("Before Java8, too much code for too little to do");
    }
}).start();

//Java 8 方式：
new Thread( () -> System.out.println("In Java8, Lambda expression rocks !!") ).start();
```

#### 2. 用 lambda 表达式进行事件处理

````java
// Java 8 之前：
JButton show =  new JButton("Show");
show.addActionListener(new ActionListener() {
    @Override
    public void actionPerformed(ActionEvent e) {
    System.out.println("Event handling without lambda expression is boring");
    }
});

// Java 8 方式：
show.addActionListener((e) -> {
    System.out.println("Light, Camera, Action !! Lambda expressions Rocks");
});
````

#### 3. 用 lambda 表达式对列表进行迭代

```java
// Java 8 之前：
List features = Arrays.asList("Lambdas", "Default Method", "Stream API", "Date and Time API");
for (String feature : features) {
    System.out.println(feature);
}

// Java 8 方式：
List features = Arrays.asList("Lambdas", "Default Method", "Stream API", "Date and Time API");
features.forEach(n -> System.out.println(n));
 
// 使用Java 8的方法引用更方便，方法引用由::双冒号操作符标示，
// 看起来像C++的作用域解析运算符
features.forEach(System.out::println);
```

#### 4. 使用 lambda 表达式和函数式接口 Predicate

除了在语言层面支持函数式编程风格，Java 8 也添加了一个包，叫做 java.util.function。它包含了很多类，用来支持 Java 的函数式编程。其中一个便是 Predicate，使用 java.util.function.Predicate 函数式接口以及 lambda 表达式，可以向 API 方法添加逻辑，用更少的代码支持更多的动态行为。下面是 Java 8 Predicate 的例子，展示了过滤集合数据的多种常用方法。**Predicate 接口非常适用于做过滤**。

```java
public static void main(args[]){
    List languages = Arrays.asList("Java", "Scala", "C++", "Haskell", "Lisp");
 
    System.out.println("Languages which starts with J :");
    filter(languages, (str)->str.startsWith("J"));
 
    System.out.println("Languages which ends with a ");
    filter(languages, (str)->str.endsWith("a"));
 
    System.out.println("Print all languages :");
    filter(languages, (str)->true);
 
    System.out.println("Print no language : ");
    filter(languages, (str)->false);
 
    System.out.println("Print language whose length greater than 4:");
    filter(languages, (str)->str.length() > 4);
}

private static void filter(List names, Predicate condition) {
    for(String name: names)  {
        if(condition.test(name)) {
            System.out.println(name + " ");
        }
    }
}

// 更好的办法
private static void filter(List names, Predicate condition) {
    names.stream().filter(name -> condition.test(name)).forEach(name -> {
        System.out.println(name + " ");
    });
    // 可进一步精简为如下形式
    names.stream().filter(condition).forEach(System.out::println);
}
```

可以看到，Stream API 的过滤方法也接受一个 Predicate，这意味着可以将我们定制的 filter() 方法替换成写在里面的内联代码，这就是 lambda 表达式的魔力。

#### 5. Predicate 的组合使用

java.util.function.Predicate 允许将两个或更多的 Predicate 合成一个。它提供类似于逻辑操作符 AND 和 OR 的方法，名字叫做 and()、or() 和 xor()，用于将传入 filter() 方法的条件合并起来。例如，要得到所有以J开始，长度为四个字母的语言，可以定义两个独立的 Predicate 示例分别表示每一个条件，然后用  Predicate.and() 方法将它们合并起来，如下所示：

```java
// 甚至可以用 and()、or() 和 xor() 逻辑函数来合并 Predicate，
// 例如要找到所有以 J 开始，长度为四个字母的名字，你可以合并两个 Predicate 并传入
Predicate<String> startsWithJ = (n) -> n.startsWith("J");
Predicate<String> fourLetterLong = (n) -> n.length() == 4;
names.stream()
    .filter(startsWithJ.and(fourLetterLong))
    .forEach((n) -> System.out.print("nName, which starts with 'J' and four letter long is : " + n));
```

#### 6. 使用 lambda 表达式的 Map 和 Reduce

```java
// 不使用lambda表达式为每个订单加上12%的税
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
for (Integer cost : costBeforeTax) {
    double price = cost + .12*cost;
    System.out.println(price);
}
 
// 使用lambda表达式
List costBeforeTax = Arrays.asList(100, 200, 300, 400, 500);
costBeforeTax.stream().map((cost) -> cost + .12*cost).forEach(System.out::println);
```

#### 7. 通过滤过创建 List

过滤是 Java 开发者在大规模集合上的一个常用操作，而现在使用 lambda 表达式和流 API 过滤大规模数据集合是惊人的简单。流提供了一个 filter() 方法，接受一个 Predicate 对象，即可以传入一个 lambda 表达式作为过滤逻辑。下面的例子是用 lambda 表达式过滤 Java 集合，将帮助理解。

```java
// 创建一个字符串列表，每个字符串长度大于2
List<String> filtered = strList.stream().filter(x -> x.length() > 2).collect(Collectors.toList());
System.out.printf("Original List : %s, filtered list : %s %n", strList, filtered);
```

另外，关于 filter() 方法有个常见误解。在现实生活中，做过滤的时候，通常会丢弃部分，但使用 filter() 方法则是获得一个新的列表，且其每个元素符合过滤原则。

#### 8. 对列表的每个元素应用函数

我们通常需要对列表的每个元素使用某个函数，例如逐一乘以某个数、除以某个数或者做其它操作。这些操作都很适合用 map() 方法，可以将转换逻辑以 lambda 表达式的形式放在 map() 方法里，就可以对集合的各个元素进行转换了，如下所示。

```java
// 将字符串换成大写并用逗号链接起来
List<String> G7 = Arrays.asList("USA", "Japan", "France", "Germany", "Italy", "U.K.","Canada");
String G7Countries = G7.stream().map(x -> x.toUpperCase()).collect(Collectors.joining(", "));
System.out.println(G7Countries);
```

#### 9. 复制不同的值，创建一个子列表

本例展示了如何利用流的 distinct() 方法来对集合进行去重。

```java
// 用所有不同的数字创建一个正方形列表
List<Integer> numbers = Arrays.asList(9, 10, 3, 4, 7, 3, 4);
List<Integer> distinct = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList());
System.out.printf("Original List : %s,  Square Without duplicates : %s %n", numbers, distinct);
```

#### 10. 计算集合元素的最大值、最小值、总和以及平均值

IntStream、LongStream 和 DoubleStream 等流的类中，有个非常有用的方法叫做 summaryStatistics() 。可以返回 IntSummaryStatistics、LongSummaryStatistics 或者 DoubleSummaryStatistics，描述流中元素的各种摘要数据。在本例中，我们用这个方法来计算列表的最大值和最小值。它也有 getSum() 和 getAverage() 方法来获得列表的所有元素的总和及平均值。

```java
//获取数字的个数、最小值、最大值、总和以及平均值
List<Integer> primes = Arrays.asList(2, 3, 5, 7, 11, 13, 17, 19, 23, 29);
IntSummaryStatistics stats = primes.stream().mapToInt((x) -> x).summaryStatistics();
System.out.println("Highest prime number in List : " + stats.getMax());
System.out.println("Lowest prime number in List : " + stats.getMin());
System.out.println("Sum of all prime numbers : " + stats.getSum());
System.out.println("Average of all prime numbers : " + stats.getAverage());
```

### lambda 表达式 vs 匿名类

 既然 lambda 表达式即将正式取代 Java 代码中的匿名内部类，那么有必要对二者做一个比较分析。一个关键的不同点就是关键字 this。匿名类的 this 关键字指向匿名类，而 lambda 表达式的 this 关键字指向包围 lambda 表达式的类。另一个不同点是二者的编译方式。Java编译器将 lambda 表达式编译成类的私有方法。使用了 Java 7的 invokedynamic 字节码指令来动态绑定这个方法。