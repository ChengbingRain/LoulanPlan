# 深入浅出 Java 泛型之(四)：类型擦除

作为泛型系列文章的最后一篇，将围绕泛型的实现原理讲讲类型擦除，以及类型擦除带来的的局限性，帮助大家加深对泛型的理解，从而更好的运用泛型。

还没有看之前文章的同学，建议先看一下：

1. 《[深入浅出 Java 泛型之(一)：前生今世](https://github.com/ruicbAndroid/LoulanPlan/blob/master/Java%20基础/003%20深入浅出%20Java%20泛型之(一)：前生今世.md)》
2. 《[深入浅出 Java 泛型之(二)：泛型详解](https://github.com/ruicbAndroid/LoulanPlan/blob/master/Java%20基础/004%20深入浅出%20Java%20泛型之(二)：泛型详解.md)》
3. 《[深入浅出 Java 泛型之(三)：类型通配符](https://github.com/ruicbAndroid/LoulanPlan/blob/master/Java%20基础/004%20深入浅出%20Java%20泛型之(三)：类型通配符.md)》

## 1. 泛型实现原理之类型擦除

关于泛型，有一种说法是：Java 泛型是伪泛型。为什么这么说呢？

因为 Java 泛型的类型信息仅在编译时用于类型检查，之后就会被编译器擦除，在最终生成的 Java 字节码中是不包含泛型的类型信息的，只有原始数据类型。这一过程我们称之为：类型擦除（Type Erasure）。

为什么要在编译期间将类型擦除呢？

根本上是为了兼容历史版本的代码。我们知道泛型是在 Java 5 中引入的，为了兼容使用 Java 5 之前的 JDK 编写的程序代码，使用类型擦除是一种不得已而为之的方案。只有这样，才能保证泛型代码与遗留代码之间的完全互操作性，而开发者也无需因引入 Java 5 而修改遗留代码。

下面通过实例加深对类型擦除的理解，编写 LoulanPlan.java 如下：

```java
public class LoulanPlan<U, V extends Number> {

    private U name;
    private V year;

    public LoulanPlan(U u, V v) {
        name = u;
        year = v;
    }

    public U getName() {
        return name;
    }

    public V getYear() {
        return year;
    }

    //入口函数
    public static void main(String... args) {
        LoulanPlan<String, Integer> loulanPlan = new LoulanPlan<>("LoulanPlan", 2019);
        String name = loulanPlan.getName();
        int year = loulanPlan.getYear();
    }
}
```

通过 `javac LoulanPlan.java` 编译得到 `LoulanPlan.class`，然后执行 `javap -v LoulanPlan.class` 反编译得到字节码。

为增加阅读体验对实际字节码进行了删减，重点看几处注释即可：

```java
Classfile .../src/main/java/com/bote/LoulanPlan.class
  //...
public class com.bote.LoulanPlan<U extends java.lang.Object, V extends java.lang.Number> extends java.lang.Object
  //...
Constant pool:
   //！！！！！！！ 注释一 ！！！！！！！！
   #1 = Methodref          #13.#37        // java/lang/Object."<init>":()V
   #2 = Fieldref           #4.#38         // com/bote/LoulanPlan.name:Ljava/lang/Object;
   #3 = Fieldref           #4.#39         // com/bote/LoulanPlan.year:Ljava/lang/Number;
   //...
{
  public com.bote.LoulanPlan(U, V);
    descriptor: (Ljava/lang/Object;Ljava/lang/Number;)V
    //...

  public U getName();
    descriptor: ()Ljava/lang/Object;
    //...

  public V getYear();
    descriptor: ()Ljava/lang/Number;
    //...

  public static void main(java.lang.String...);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC, ACC_VARARGS
    Code:
      stack=4, locals=4, args_size=1
        //...
        //！！！！！！！ 注释二 ！！！！！！！！
        17: invokevirtual #8                  // Method getName:()Ljava/lang/Object;
        20: checkcast     #9                  // class java/lang/String
        23: astore_2
        24: aload_1

        25: invokevirtual #10                 // Method getYear:()Ljava/lang/Number;
        28: checkcast     #11                 // class java/lang/Integer
        31: invokevirtual #12                 // Method java/lang/Integer.intValue:()I
        34: istore_3
        35: return
        //...
}
Signature: #34                          // <U:Ljava/lang/Object;V:Ljava/lang/Number;>Ljava/lang/Object;
SourceFile: "LoulanPlan.java"
```

下面通过三处注释来看下类型擦除在字节码中的体现。

- 注释一：全局变量 `name` 和 `key` 的实际类型为 `Object` 和 `Number`，分别对应类型参数 `U` 和 `V` 的上限。

```java
#2 = Fieldref           #4.#38         // com/bote/LoulanPlan.name:Ljava/lang/Object;
#3 = Fieldref           #4.#39         // com/bote/LoulanPlan.year:Ljava/lang/Number;
```

- 注释二：通过 `LoulanPlan<String, Integer> loulanPlan` 创建实例时，声明 `name` 为 `String` 类型，但 `getName` 的返回值 `name` 为 `Object` 类型，所以需要通过 `checkcast` 指令检查返回值是否为 `String` 类型，若检查失败则直接抛出 `ClassCastException` 异常。

```java
17: invokevirtual #8                  // Method getName:()Ljava/lang/Object;
20: checkcast     #9                  // class java/lang/String
```

相信通过观察字节码，能够进一步加深对类型擦除的理解。记住，类型擦除是在编译时进行的，所以在运行时对于 JVM 而言，泛型类和普通类没有任何区别，更准确的说 JVM 根本不知道泛型的存在，从字节码中就可以看出这一点。

## 2. 类型擦除带来的局限性

类型擦除的主要作用在于兼容历史代码，但同时也引入了一些局限性，简单列举几点常见的局限。

### 2.1 无法在运行时区分类型

类型擦除使得泛型信息在编译阶段被擦除，所以我们无法在运行时区分不同参数类型的泛型类信息。

```java
ArrayList<String> strList = new ArrayList<>();
ArrayList<Integer> intList = new ArrayList<>();

System.out.print(ArrayList.class === strList.getCalss());
System.out.print(ArrayList.class === intList.getCalss());
```

因为每个类有且仅有一份 `Class` 信息，对于泛型类而言，与其具体的类型参数无关。所以，上述输出均为 `true`。

同理，由于类型无关，我们无法通过下述方式访问类的 `Class` 信息：

```java
List<String>.class; //错误，正确写法：List.class
```

### 2.2 不能使用类型参数创建实例

```java
T t = new T();
```

上述代码是无法编译通过的，因为擦除擦除，编译期间将失去类型信息，无法知道类类型。其次，无法获知 T 类型是否包含无参构造函数。

### 2.3 不能用于静态变量

静态变量和静态方法属于类级别，也就是说不需要通过实例对象进行访问，而泛型类中的类型参数只能在实例化时指定类型实参。显然，泛型无法作用于静态变量。

## 3. 最后

至此，有关泛型的系列文章就告一段落了。对于本系列文章有任何交代不清楚或者错误的地方，欢迎指正。
