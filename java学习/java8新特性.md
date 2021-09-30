## lambda表达式

### 口诀和四个要点
1. 拷贝小括号，写死右箭头，落地大括号。
2. @FunctionalInterface
3. 可以定义多个default方法
4. 可以定义多个static方法

> 1.8之后引入了一个新的操作符 -> ，该操作符为箭头操作符或lambda操作符。它将操作符拆分为两个部分。

> lambda表达式的参数列表的数据类型可以省略不写，因为JVM编译器通过上下文推出。

- 左侧：lambda表达式的参数列表
- 右侧：lambda表达式中所需执行的功能

### 无参数，无返回值
```java
//() -> System.out.println("hello");

Runnable r1 = () -> System.out.println("hello");
```

### 有一个参数，无返回值
```java
//(x) -> System.out.println(x);

Consumer<String> con = (x) -> System.out.println(x);
con.accept("hello");
```

### 若lambda中只有一条语句，return和大括号都可以不写
```java
Comparator<Integer> com = (x, y) -> Integer.compare(x,y);
```

### 有两个以上的参数，有返回值，并且有多个lambda体中有多条语句
```java
Comparator<Integer> com = (x, y) -> {
    System.out.println("hello world");
    return Integer.compare(x,y);
};
```

### lambda表达式需要函数式接口的支持。
> 函数式接口中只有一个抽象方法，加上```@functionalInterface```修饰，可以检查是否是函数式接口。

```java
@FunctionalInterface
public interface Condition<T> {
    public boolean test(T t);
}
```

### 调用collection.sort()方法排序
```java
Collections.sort(list, (e1, e2) -> {
    if (e1.getAge() == e2.getAge()){
        return e1.getName().compareTo(e2.getName());
    }else {
        return Integer.compare(e1.getAge(), e2.getAge());
    }
});
```

## 函数式接口

### Java内置四大函数式接口
- Consumer<T> ：消费型接口
```java
void accept(T t);
```

- Supplier<T> ：供给型接口
```java
T get();
```

- Function<T, R> ：函数型接口
```java
R apply(T t);
````

- Predicate<T> ：断言型接口
```java
boolean test(T t);
```

## 方法引用和构造器引用

> 方法引用：若lambda体中的内容有方法已经实现，我们可以使用“方法引用”。

> 注意：lambda体中调用方法的参数列表与返回值类型，要与函数式接口中抽象方法的函数列表和返回值保持一致。

1. 对象::实例方法名
```java
PrintStream ps = System.out;
Consumer<String> con = ps::println;
```

2. 类::静态方法名
```java
Employee employee = new Employee("zhangsan", 22, 5500);
Supplier<String> sup = employee::getName;
System.out.println(sup);

Comparator<Integer> comparator = Integer::compare;
```

3. 类::实例方法名
```java
BiPredicate<String, String> biPredicate = String::equals;
```

## stream

### stream操作的过程
1. 将数据源（集合、数组等）转换为流。
2. 对流进行一系列的中间操作。
3. 产生一个新的流。

### 注意点
1. stream自己不会存储元素。
2. stream不会改变源对象，他会返回一个持有结果的新stream。
3. stream操作是延迟执行的，当需要结果时才会执行。

### 创建流
```java
//      1.通过collection系列集合提供的stream方法
        List<String> list = new ArrayList<>();
        Stream<String> stream1 = list.stream();
//      2.通过arrays中的静态方法stream
        String[] str = new String[10];
        Stream<String> stream2 = Arrays.stream(str);
//      3.通过stream类中静态方法of
        Stream<String> stream3 = Stream.of("aa", "bb", "cc");
//      4.创建无限流     
//      迭代
        Stream<Integer> stream4 = Stream.iterate(100, x -> x+20);
        stream4.limit(10);
//      生成
        Stream<Integer> stream5 = Stream.generate(() -> (int) Math.random()*100);
        stream5.limit(10);
```

### 中间操作
- 筛选和切片
```java
        List<Employee> list = new ArrayList<>();
        list.add(new Employee("zhangsan", 22, 5900));
        list.add(new Employee("lisi", 39, 5000));
        list.add(new Employee("wangwu", 26, 6800));
        list.add(new Employee("zhaoliu", 19, 4000));
        list.add(new Employee("mazi", 41, 8000));
        list.add(new Employee("mazi", 41, 8000));
        /**
         * 筛选和切片
         * filter--接受lambda，从流中排出某些元素
         * limit--截断流，使其元素不超过给定数量
         * skip(n)--跳过元素，返回一个扔掉前n个元素的流，不足n个，返回一个空的流
         * distinct--筛选，通过流所产生元素的hashcode()和equals()去除重复的元素-----需要重写hashcode()和equals()
         */
        Stream<Employee> stream01 = list.stream()
                                        .distinct()
                                        .skip(1)
                                        .limit(10)
                                        .filter(e -> e.getAge()>35);
        stream01.forEach(System.out::println);
```

- stream映射
```java
        /**
         * 映射
         * map--接受lambda表达式，将元素转换成其他形式或提取信息。接受一个函数作为参数，该函数会被应用到每个元素上，并将其映射成一个新的元素。
         * flatMap--接受一个函数作为参数，将流中每一个值都换成另一个值，然后把所有的流连成一个流。
         */
        String[] s = {"aa", "bb", "cc", "dd", "ee"};
        List<String> list1 =  Arrays.asList(s);
        list1.stream()
            .map(String::toUpperCase)
            .forEach(System.out::println);

        List<Employee> list = new ArrayList<>();
        list.add(new Employee("zhangsan", 22, 5900));
        list.add(new Employee("lisi", 39, 5000));
        list.add(new Employee("wangwu", 26, 6800));
        list.add(new Employee("zhaoliu", 19, 4000));
        list.add(new Employee("mazi", 41, 8000));
        list.add(new Employee("mazi", 41, 8000));
        list.stream()
            .map(Employee::getAge)
            .forEach(System.out::println);

        Stream<Character> stream = list.stream()
                                       .map(Employee::getName)
                                       .flatMap((str) -> {
                                           List<Character> li = new ArrayList<>();
                                           for (Character ch : str.toCharArray()){
                                               li.add(ch);
                                           }
                                           return li.stream();
                                       });
        stream.forEach(System.out::println);
```

- stream排序
```java
        /**
         * 排序
         * sorted()--自然排序
         * sorted(Comparator com)--定制排序
         */
        List<String> list1 = Arrays.asList("ll", "ss", "mm", "aa");
        list1.stream()
                .sorted()
                .forEach(System.out::println);

        List<Employee> list = new ArrayList<>();
        list.add(new Employee("zhangsan", 22, 5900));
        list.add(new Employee("lisi", 39, 5000));
        list.add(new Employee("wangwu", 26, 6800));
        list.add(new Employee("zhaoliu", 19, 4000));
        list.add(new Employee("mazi", 41, 8000));
        list.stream()
                .sorted((e1, e2) -> {
                    if (e1.getAge() == e2.getAge()){
                        return e1.getName().compareTo(e2.getName()); 
                    }else {
                        return Integer.compare(e1.getAge(), e2.getAge());
                    }
                }).forEach(System.out::println);
```

- stream查找和匹配
```java
        /**
         * 查找与匹配
         * allMatch--检查是否匹配所有的元素
         * anyMatch--检查是否至少匹配一个元素
         * noneMatch--检查是否没有匹配所有元素
         * findFirst--返回第一个元素
         * findAny--返回当前流中的任意元素
         * count--返回流中元素的总个数
         * max--返回流中最大值
         * min--返回流中最小值
         */
        List<Employee> list = new ArrayList<>();
        list.add(new Employee("zhangsan", 22, 5900));
        list.add(new Employee("lisi", 39, 5000));
        list.add(new Employee("wangwu", 26, 6800));
        list.add(new Employee("zhaoliu", 19, 4000));
        list.add(new Employee("mazi", 41, 8000));
        boolean flag = list.stream()
            .allMatch((e) -> e.getSalary()>3000);
```

- stream归约和收集
```java
        /**
         * 归约
         * reduce(T identity, BinaryOperator) / reduce(BinaryOperator)--可以将流中元素反复结合起来，得到一个值
         */
        List<Integer> list1 = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9);
        Integer sum = list1.stream().reduce(0, (x,y) -> x+y);

        List<Employee> list = new ArrayList<>();
        list.add(new Employee("zhangsan", 22, 5900));
        list.add(new Employee("lisi", 39, 5000));
        list.add(new Employee("wangwu", 26, 6800));
        list.add(new Employee("zhaoliu", 19, 4000));
        list.add(new Employee("mazi", 41, 8000));
        Optional<Float> s = list.stream().map(Employee::getSalary)
                     .reduce(Float::sum);
        System.out.println(s);
        
        /**
         * 收集
         * collect--将流转换为其他形式，接收一个collector接口的实现，用于给stream中元素做汇总的方法。
         */
        Set<String> list3 = list.stream()
                                .map(Employee::getName)
                                .collect(Collectors.toSet());
        List<String> list4 = list.stream()
                                 .map(Employee::getName)
                                 .collect(Collectors.toList());
        HashSet<String> list5 = list.stream()
                                    .map(Employee::getName)
                                    .collect(Collectors.toCollection(HashSet::new));
        //总数
        Long count = list.stream()
                         .collect(Collectors.counting());
        //平均值
        Double avg = list.stream()
                         .collect(Collectors.averagingDouble(Employee::getSalary));
        //总和
        Double zonghe = list.stream()
                            .collect(Collectors.summingDouble(Employee::getSalary));
        //最大值
        Optional<Employee> max = list.stream()
                                     .collect(Collectors.maxBy((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary())));
        //最小值
        Optional<Employee> min = list.stream()
                                     .collect(Collectors.minBy((e1, e2) -> Double.compare(e1.getSalary(), e2.getSalary())));
        //分组
        Map<Integer, List<Employee>> m = list.stream()
                                             .collect(Collectors.groupingBy(Employee::getAge));
        Map<Integer, Map<String, List<Employee>>> m1 = list.stream()
                                                           .collect(Collectors.groupingBy(Employee::getAge, Collectors.groupingBy(Employee::getName)));
        //分区
        Map<Boolean, List<Employee>> m2 = list.stream()
                                              .collect(Collectors.partitioningBy((e) -> e.getSalary()>5000));
        //另外一种获取方式
        DoubleSummaryStatistics dss = list.stream()
                                          .collect(Collectors.summarizingDouble(Employee::getSalary));
        System.out.println(dss.getMax() + dss.getAverage() + dss.getCount() + dss.getMin());
        //将元素全部以字符串的方式拼接起来起来
        String str = list.stream()
                         .map(Employee::getName).collect(Collectors.joining());
        
```

## optional类
> optional<T>类是一个容器类，代表一个值存在或不存在，原来null表示一个值不存在，optional可以更好地表达这个概念，并且可以避免空指针异常。

### 常用方法
- ```Optional.of(T t)```:创建一个Optional实例
- ```Optional.empty()```:创建一个空的Optional实例
- ```Optional.ofNullable(T t)```:若t不为null，创建Optional实例，否则创建空实例。
- ```isPresent()```:判断是否包含值
- ```orElse(T t)```:如果调用对象包含值，返回该值，否则返回t
- ```orElseGrt(Supplier s)```:如果调用对象包含值，返回该值，否则返回s获取的值。
- ```map(Function f)```:如果有值对其处理，并返回处理后的Optional，否则返回optional.empty()
- ```flatMap(Function mapper)```:与map类似，要求返回值必须是Optional


## 接口中的默认方法和静态方法
- 接口中可以有默认方法
```java
public interface Moren{
    default void test(){
        System.out.println("默认方法");
    }
}
```

- 接口中可以有静态方法
```java
public interface Moren{
    public static void test(){
        System.out.println("静态方法");
    }
}
```

## 新时间日期API
> 都是不可变的，线程安全的
1. ```LocalDate```：日期、```LocalTime```：时间、```LocalDateTime```：日期和时间

```java
LocalDateTime localDateTime = LocalDateTime.now();
System.out.println(localDateTime);

LocalDateTime localDateTime1 = LocalDateTime.of(2020, 4, 13, 11, 13, 44);
System.out.println(localDateTime1);

LocalDateTime localDateTime2 = localDateTime.plusDays(1);       //加一天
LocalDateTime localDateTime3 = localDateTime.plusMonths(1);     //加一个月
LocalDateTime localDateTime4 = localDateTime.minusDays(1);      //减一天
LocalDateTime localDateTime5 = localDateTime.minusMonths(1);    //减一个月

localDateTime.getYear();
localDateTime.getMonth();
localDateTime.getMonthValue();
```

2. Instant:时间戳（1970年1月1日00：00：00到某个时间之间的毫秒值）
```java
Instant instant = Instant.now();        //默认获取UTC时区
OffsetDateTime offsetDateTime = instant.atOffset(ZoneOffset.ofHours(8));
System.out.println(offsetDateTime.toEpochSecond());

Instant instant1 = Instant.now();
Thread.sleep(1000);
Instant instant2 = Instant.now();
System.out.println(Duration.between(instant1, instant2).getNano());
```