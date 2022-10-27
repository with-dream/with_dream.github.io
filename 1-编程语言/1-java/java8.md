#### 1、Lambda 表达式

##### 1.1、基本语法

java语法糖 底层通过内部类实现，可以自动推导参数类型

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

Supplier

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

Consumer