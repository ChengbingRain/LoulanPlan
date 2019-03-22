# 深入浅出 Java 泛型之：用武之地篇

在介绍泛型的使用之前，我们先简单回顾下相关概念，之后会向大家介绍泛型的命名规范，然后通过实例讲解泛型在类、接口和方法中的运用，以及泛型的类型上限和派生。

## 1. 概念回顾

泛型，**即参数化类型**，支持将可变的类型抽象为形参，在使用时动态指定类型实参。使用时，通过 `<>` 声明泛型，形参可以是任意需要抽象的类型。

更多有关泛型的基础概念，可以看上一篇文章 《[深入浅出 Java 泛型之：前生今世篇](https://github.com/ruicbAndroid/LoulanPlan/blob/master/Java%20%E5%9F%BA%E7%A1%80/003%20%E6%B7%B1%E5%85%A5%E6%B5%85%E5%87%BA%20Java%20%E6%B3%9B%E5%9E%8B%E4%B9%8B%EF%BC%9A%E5%89%8D%E7%94%9F%E4%BB%8A%E4%B8%96%E7%AF%87.md)》。下文主要介绍泛型的命名规范和运用场景。

## 2. 命名规范

在 Java 官方文档中，其建议我们使用 **单个大写字母** 来表示类型形参，这与我们所了解的类或接口的命名规范是相违背的。为什么？

因为只有这样，我们才能识别出某一类型是泛型还是类或接口，否则就像下面的代码:

```java
public class Test<Param> {

    public void doSth(Param param) {
        //下意识会觉得 Param 是一个类或接口，实则为泛型的类型形参
    }
}
```

使用 `Param` 作为泛型参数没有任何问题，但显然会使人迷惑，所以遵循**单个大写字母**的命名规范才是最佳选择。

当然了，大写字母也不是随便用的，最好能使用大家达成共识的常用字母（我写了几年代码了，还没见到过使用 A,B,C 作为泛型类型的）。Java 官方文档中给出了常用的大写字母，及其代表的含义:

1. E - Element：元素。 被大量使用在 Java 集合框架中
2. K - Key：键
3. V - Value：值
4. N - Number：数字
5. T - Type：类型
6. S,U 等常用字母

有关泛型的命名规则就介绍到这儿，下面一起来了解下泛型在类、接口和方法中的运用。

## 3. 泛型在类和接口中的运用

定义类或接口时，可以在名称之后追加 `<>` 来声明类型形参，多个形参之间可以使用 `,` 分隔。下面通过定义泛型类来举例（定义泛型接口类似）：

```java
public class GenericClass<K, V> {
    //使用泛型定义变量
    K mKey;
    V mValue;

    //使用泛型定义方法参数
    private void set(K key, V value) {
        mKey = key;
        mValue = value;
    }

    //使用泛型作为方法返回值
    private K getKey() {
        return mKey;
    }

    private V getValue() {
        return mValue;
    }
}
```

如上，类型形参和普通数据类型一样，可以用于定义类变量，用于方法参数定义、方法返回值等。

在程序中使用泛型类 `GenericClass`：

```java
//创建实例
GenericClass<String, Integer> gClass = new GenericClass<>();
//赋值
gclass.set("key", 666);
//取值
String key = gClass.getKey();
int value = gClass.getValue();
```

使用上除了在创建实例时需要制定类型，其它地方与普通类没有区别，这就是泛型的意义所在。

最后，需要注意的是，**类型形参不可用于声明静态变量、定义静态方法**。因为类型形参需要在创建类实例时指定，而静态变量或函数属于类而非类实例。

## 4. 泛型在方法中的运用

我们回到上面定义的泛型类 `GenericClass<K, V>`，其包含了如下方法：

```java
private K getKey() {
    return mKey;
}
```

该方法是泛型方法吗？答案很显然：不是。如果是的话，我何必多此一举将泛型在方法中的运用拎出来单独讲呢？

先说结论，下文展开介绍。

> **泛型方法与其所在的类或接口是否是使用了泛型无关，其拥有独立的泛型作用域**。

泛型方法如下面的 `eMerge()` 方法：

```java
public class Test<T> {

    //普通方法
    public List<T> tMerge(@NonNull List<T> from, @NonNull List<T> to) {
        for (T t : from) {
            to.add(t);
        }
        return to;
    }

    //泛型方法
    public <E> List<E> eMerge(@NonNull List<E> from, @NonNull List<E> to) {
        for (E e : from) {
            to.add(e);
        }
        return to;
    }
}
```

`tMerge` 方法虽使用泛型 T 定义参数，但其本质上仍未普通方法，因为此处的 T 是 `Test` 类定义的。

`eMerge()` 同样使用了泛型 E 定义参数，却是泛型方法。因为我们**在方法的修饰符和返回值之间通过 `<>` 将其定义为泛型方法**，这里的类型 E 与 `Test` 类无关，其属于方法本身。

在泛型方法中，类型形参的作用域仅为当前方法，包括返回值、声明方法参数、定义局部变量。

下面一起看看普通方法和泛型方法在使用上的差异：

```java
//准备数据,省略初始化并填充数据的过程...
List<String> strList_1, strList_2;
List<Integer> intList_1, intList_2;

//定义泛型类
Test<String> test = new Test<>();

//调用普通方法
List<String> mergeStrList = test.tMerge(strList_1, strList_2);
//调用泛型方法
List<Integer> mergeIntList = test.eMerge(intList_1, intList_2);
```

可以看到，泛型方法与普通方法在使用上没有任何区别。

有一点需要注意的是，在调用泛型方法时，我们没有像使用泛型类那样显示指定实际类型，但系统依然可以识别参数类型，这是因为**编译器会根据传入的实参推断出泛型形参的类型**。

至此，我们学习了泛型在类（或接口）和方法中的运用，算是掌握了泛型的基本用法，下面继续深入学习泛型的相关知识。

## 5. 类型形参的上限

在前面定义泛型类或方法时，我们通过 `<E>` 的形式定义类型形参。现在，我们希望为形参指定上限，限定外部传入指定类型及其子类，该如何实现呢？

泛型支持通过 `extends` 关键字限定类型形参的上限，上限可以是类或接口。`extends` 在一般意义上用于表示 `extends`（上限为类）或 `implements`（上限为接口）。下面一起通过示例巩固下：

```java
public <E extends Number> E max(E num_1, E num_2) {

}
```

`max()` 是泛型方法，支持传入两个数值并返回较大值。在定义类型形参时，我们通过 `<E extends Number>` 限定上限为 `Number`，使得外部只能传入 `Nunber` 及其子类的类型，如下：

```java
//返回 3 和 5 的较大值
int value = max(3, 5);

//错误的调用，不支持 String 类型参数
String value = max("Hello", "World");
```

可以看出，类型上限能够起到很好的限制作用，合理利用能够帮助我们写出更优雅、健壮的 API。

我们继续完善上述方法，发现无法对两个数进行比较。可能有些同学会觉得，既然是 `Number` 类型了，直接比较不就行了吗？

```java
public <E extends Number> E max(E num_1, E num_2) {
    return num_1 > num_2 ? num_1 : num_2;
}
```

显然不可以，因为 `Number` 类型的数据是不支持进行比较的。

实际上，`Integer` 和 `Long` 等类 `extends Number` 的同时，还 `implements Compare<T>`，并实现其接口方法 `compareTo(T t)` 用于进行比较。借助于这一特点，我们自然想到为形参添加 `Compare<T>` 上限，姿势如下：

```java
public <E extends Number & Comparable<E>> E max(E num_1, E num_2) {
    //通过 Compare 接口的 compareTo() 方法进行比较
    if (num_1.compareTo(num_2) > 0) {
        return num_1;
    }
    return num_2;
}
```

在声明类型参数 `<E extends Number & Comparable<E>>` 时，我们通过 `&` 为形参 `E` 设定了多个上限，既限制传入的数据类型，又满足我们对数据进行比较的需求。

补充一点，`<E extends Number & Comparable<E>>` 涉及了泛型接口 `Compare<T>` 的派生，这个知识点将在下一节介绍。

最后，需要强调的是，通过 `&` 设定多上限时，不可以违背 Java 的 "单 extends 多 implements" 的规则。即 **最多只可以限定一个父类上限，但可以限定多个接口上限，且父类上限（如果有的话）必须作为第一个上限类型**。

## 6. 泛型类的派生

和普通类或接口一样，泛型类或接口同样支持派生，即被子类继承或实现。

不同的是，由于泛型类或接口包含类型形参，当派生子类时，其不可以再包含类型形参，必须指定类型实参。这和使用泛型类定义对象时必须指定类型实参是一个道理。

定义泛型类 `Father`：

```java
class Father<E> {

}
```

基于 `Father` 派生子类 `Son`：

```java
//case 1：正确
class Son extends Father<String> {}

//case 2：错误
class Son extends Father<E> {}
```

如 “case 1” 为父类 `Father` 传递类型 `String` 一样，子类必须为父类传入类型实参。当然，这只是概括性的说法，有两种特殊的场景需要拎出来单独讲一下：**其一是不显示指定父类的类型实参，其二是将子类的类型形参作为实参传递给父类**。

### 6.1 不显示指定父类的类型实参

不显示指定，是指从代码上看是没有指定的：

```java
class Son extends Father {
    //...
}
```

注意上面并没有为父类 `Father` 指定类型，甚至连泛型符号 `<>` 都没有，但程序依然可以正常运行，只是编译阶段编译器会提示“使用了未经检查或不安全的操作”（先忽略）。

如此看来，岂不是与前面说的“必须指明父类形参类型”相违背吗？非也非也。如果在使用泛型类时，没有显示指定类型，在本例中系统会默认为父类传递 `Object` 作为实参（标记①）。所以，上述代码等价于：

```java
class Son extends Father<Object> {}
```

需要解释一下，并非不显示指定类型，就一律默认为 `Object`。严格来说，系统会将类型实参当做**声明该参数时指定的第一个上限类型**去处理。

在 Java 中任意 `class` 类都是 `Object` 类的直接或间接子类，所以 `Father<E>` 的类型参数 `E` 存在隐形上限 `Object`，即 `Father<E extends Object>`。这就解释了上面的例子中，为什么会默认传递 `Object` 作为实参。

再举个例子，将 `Father<E>` 的声明改为 `Father<E extends String>`，则上述不显示指定父类类型的代码等价于：

```java
class Son extends Father<String> {}
```

这里默认传递 `String` 就很好理解了，因为 `Father` 类在声明类型形参 `E` 时的第一个上限为 `String` 类型。

### 6.2 将子类的类型形参作为实参传递给父类

这种场景其实很好理解，直接上代码：

```java
class Son<E> extends Father<E> {}
```

看到 `extends Father<E>`，会让人理解为没有为父类传递类型实参。实则不然，**因为 `Father` 的 `E` 并非类型形参，而是实参**。

在前面介绍泛型在类中的使用时说过，“泛型类的类型形参和普通数据类型一样，可以在类的任意位置使用”。所以在这里，就是**将子类的类型形参作为实参传递给父类**，同样遵循泛型类派生的规则。

现在，回顾上一节的 `<E extends Number & Comparable<E>>` 就很好理解了，在为方法定义类型参数时涉及了泛型接口的派生，将形参 `E` 作为实参传递给 `Comparable` 接口。

## 7. 总结

至此，我们已经了解了泛型是什么，解决了什么问题，以及如何使用泛型。只要加以练习加深理解，就能够满足企业级开发的需求了。

下篇文章（对，还有第三篇），会讲解泛型的一些高级内容，敬请期待...