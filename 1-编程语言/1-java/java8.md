[TOC]

#### 1、Lambda 表达式

##### 1.1、基本语法

java语法糖 lambda表达式 底层通过内部类实现，可以自动推导参数类型

```java
//1、一个参数时 可以省略小括号
Comparator<Integer> comparator = (x,y) -> {
    System.out.println("相加结果是:"+(x+y));
    return Integer.compare(x,y);
};
//2、Lambda 体只有一条语句时，return/大括号可以省略
Comparator<Integer> comparator = (x,y) -> Integer.compare(x,y);
```

##### 1.2、函数式接口

只包含一个抽象方法的接口，称为函数式接口

lambda表达式只能用在函数式接口上

##### 1.3、常用内置函数式接口

可以将lambda作为参数使用

```java
public interface Supplier<T> {
    T get();
}

public class Test {
    public static void main(String[] args) {
        int[] arr={1,9,2,7,5};
      	//lambda做参数 简化代码
        int maxValue=getMax(()->{
            int max=arr[0];
            for(int i=1;i<arr.length;i++){
                if(arr[i]>max){
                    max=arr[i];
                }
            }
            return max;
        });
        System.out.println(maxValue);
    }
		//Supplier做参数
    private static int getMax(Supplier<Integer> sup){
        return sup.get();
    }
}
```

```java
@FunctionalInterface
public interface Consumer<T> {  //传参运算
    void accept(T t);
}

@FunctionalInterface
public interface Supplier<T> {	//运算返回数据
   T get();
}

@FunctionalInterface
public interface Function<T, R> {	//传参-运算-返回
    R apply(T t);
}

@FunctionalInterface
public interface Predicate<T> {	//传参-返回判断
    boolean test(T t);
}
```

##### 1.4、方法参考

> 引用静态方法						ContainingClass::staticMethodName
> 引用特定对象的实例方法	containsObject::instanceMethodName
> 引用特定类型的任意对象的实例方法	ContainingType::methodName
> 对构造函数的引用				ClassName::new

#### 2、流

- 源：表示将数据项提供给流程的数据源,[IntStream只能用于int类型数据的操作]。常见的来源是：
  - Collection.stream() — 从集合的元素创建一个流
  - Stream.of(T...) — 从传递给工厂方法的参数创建一个流
  - Stream.of(T[]) — 从数组的元素创建一个流
  - Stream.empty() — 创建一个空流
  - IntStream.range(lower, upper) — 创建一个由从下到上的元素组成的 IntStream，互斥
  - IntStream.rangeClosed(lower, upper) — 创建一个由从下到上的元素组成的 IntStream，包括在内。
  - Stream stream = List.stream(); -- 通过集合转为流
- 零个或多个中间操作：中间操作返回一个流，因此我们可以链接多个中间操作。中间操作的一个重要特征是惰性。只有存在终端操作时才会执行中间操作。常见的中间操作有：
  - filter(Predicate<T>) — 匹配谓词的流元素
  - map(Function<T, U>) — 将提供的函数应用于流元素的结果
  - flatMap(Function<T, Stream<U>> — 通过将提供的流承载函数应用于流的元素而产生的流的元素
  - distinct() — 流的元素，已删除重复项
  - sorted() — 流的元素，按自然顺序排序
  - Sorted(Comparator<T>) — 流的元素，按提供的比较器排序
  - limit(long) - 流的元素，截断到提供的长度
  - skip(long) — 流的元素，丢弃前 N 个元素
- 单终端操作：终端操作要么无效，要么返回非流结果。它们使流管道执行并返回所有应用操作的执行结果。常见的终端操作有：
  - forEach(Consumer<T> action) — 将提供的操作应用于流的每个元素
  - toArray() — 从流的元素创建一个数组
  - reduce(...) — 将流的元素聚合成一个汇总值
  - collect(...) — 将流的元素聚合到一个汇总结果容器中
  - min(Comparator<T>) — 根据比较器返回流的最小元素
  - max(Comparator<T>) — 根据比较器返回流的最大元素
  - count() — 返回流的大小
  - [any,all,none]Match(Predicate<T>) — 返回流的任何/所有/没有元素是否与提供的谓词匹配。这是一个短路操作
  - findFirst() — 返回流的第一个元素（如果存在）。这是一个短路操作
  - findAny() — 返回流的任何元素（如果存在）

流实例demo: https://blog.csdn.net/wang0907/article/details/123851730

#### 3、接口增强

java8前 接口中只能定义抽象方法

java8接口中使用default修饰且可以实现方法体  也可以进行重写

#### 4、Optional

判空 主要配合流使用

#### 5、时间

LocalDate：表示日期，包含：年月日。格式为：2020-01-13
LocalTime：表示时间，包含：时分秒。格式为：16:39:09.307
LocalDateTime：表示日期时间，包含：年月日 时分秒。格式为：2020-01-13T16:40:59.138
DateTimeFormatter：日期时间格式化类
Instant：时间戳类
Duration：用于计算 2 个时间(LocalTime，时分秒)之间的差距
Period：用于计算 2 个日期(LocalDate，年月日)之间的差距
ZonedDateTime：包含时区的时间
ChronoUnit: 可用于日期计算时指定日期单位



参考: https://blog.csdn.net/lzb348110175/article/details/103955158

