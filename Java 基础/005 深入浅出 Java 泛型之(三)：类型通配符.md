# 深入浅出 Java 泛型之(三)：类型通配符

>本文出自伯特的《[LoulanPlan](https://github.com/ruicbAndroid/LoulanPlan)》，转载务必注明作者及出处。

本文讲一讲泛型中比较独立的一块：类型通配符，掌握其有利于我们写出更加简洁、优雅的代码。其对泛型基础有一定的要求，尤其是泛型方法，这块还没有掌握的建议先看一下前两篇文章：

1. 《[深入浅出 Java 泛型之(一)：前生今世](https://github.com/ruicbAndroid/LoulanPlan/blob/master/Java%20基础/003%20深入浅出%20Java%20泛型之(一)：前生今世.md)》
2. 《[深入浅出 Java 泛型之(二)：泛型详解](https://github.com/ruicbAndroid/LoulanPlan/blob/master/Java%20基础/004%20深入浅出%20Java%20泛型之(二)：泛型详解.md)》

## 1. 背景

上文讲解了泛型在方法中的运用，现在我们定义一个泛型方法，支持传入 `List` 集合，判断其是否合法（不为空且存在元素）：

```java
public <E> boolean validList(List<E> list) {
    return list != null && !list.isEmpty();
}
```

观察方法的实现，发现其与元素类型没有任何关系。因此，**只要限定方法的形参为任意 `List` 集合的父类即可**。尝试优化如下：

```java
public boolean validList(List<Object> list) {
    return list != null && !list.isEmpty();
}
```

改造后的方法只是普通方法，不再是泛型方法了。少了类型参数 `<E>` 的声明，方法看起来确实更精简一些。更重要是的，使用 `Object` 限定 `List` 元素类型，这样就可以传入任意类型的 `List` 了，因为 `Object` 是所有类的直接或间接父类。真的是这样吗？

实则不然，如果我们调用方法并传入 `List<String>` 类型的数据，编译器会报错，提示我们 **无法将 `List<String>` 传入 `validList(List<Object>)` 方法**。区别于数组，在集合框架中，**即便类型 `String` 是 `Object` 的子类，但 `List<String>` 并不是 `List<Object>` 的子类**，所以上述入参显然是不正确的，这一点需要格外注意。

那怎么才能表示任意 `List` 集合的父类呢？那就是本文所讨论的 **类型通配符** 了，如下：

```java
public boolean validList(List<?> list) {
    return list != null && !list.isEmpty();
}
```

此时，再调用方法并传入 `List<String>` 类型的数据，程序就能够正常运行并符合预期。

## 2. 类型通配符

上述形如 `List<?>` 的写法表示的就是类型通配符，那么其与 `List<E>` 有什么区别呢？

`List<E>` 中的 `E` 是声明类型参数，通常放在类名之后和方法返回值之前，以支持外部在实例化对象或调用方法时传递类型实参；

而 `List<?>` 中的 `?` 用于实例化类型参数，只是具体的实例类型是未知的，可以是任意类型。

在 Java 规范文档中，**建议我们在声明方法时，优先使用类型通配符**，能够让代码更加简洁。同时，由于类型未知，所以只能对数据进行类型无关的操作，使得代码意图更明确、可读性更强。

通常称 `?` 为无限定通配符，下面我们就接着讲一讲类型通配符的限定。

## 3. 有限定通配符

正如可以对类型形参进行限定一样，我们也可以对类型通配符进行限定。区别是，类型形参仅支持设定上限，而类型通配符除了设定上限，还可以设定下限。

### 3.1 类型通配符的上限

和限定类型形参的上限一样，使用 `extends` 关键字即可，上限可以是类或接口，形如 `List<? extends AClass>`。

更多知识点就不再展开了，建议大家直接参考《[深入浅出 Java 泛型之(二)：泛型详解#类型形参的上限](https://github.com/ruicbAndroid/LoulanPlan/blob/master/Java%20基础/004%20深入浅出%20Java%20泛型之(二)：泛型详解.md#5-类型形参的上限)》。

### 3.2 类型通配符的下限

上限，可以限定通配符为指定类型或其子类。反之，下限可以限定通配符为指定类型或其超类。

我们通过 `super` 关键字类限定类型通配符的下限，一般称之为 **超类通配符**，形如 `List<? super AClass>`。

超类通配符有什么作用呢？先来看下面的代码：

```java
public class MainClass {
    //定义内部类 Father
    static class Father implements Comparable<Father> {
        @Override
        public int compareTo(@NonNull Father a) { return 0;}
    }

    //定义内部类 Son，继承自 Father
    static class Son extends Father { }

    //声明方法 max，用于获取集合中的最大元素
    <T extends Comparable<T>> void max(List<T> list) { }

    //程序执行入口
    public static void main(String[] args) {
        //创建可执行程序实例
        MainClass mainClass = new MainClass();

        //获取 List<Father> 中的最大元素
        List<Father> fathers = new ArrayList<>();
        fathers.add(new Father());
        fathers.add(new Father());
        mainClass.max(fathers);

        //获取 List<Son> 中的最大元素
        List<Son> sons = new ArrayList<>();
        sons.add(new Son());
        sons.add(new Son());
        mainClass.max(sons);
    }
}
```

交代下上述代码做了什么：在 `main()` 方法中通过调用 `max()` 方法，比较并获取集合中的最大元素。一个是 `List<Father>` 集合，另一个是 `List<Son>` 集合，其中 `Son extends Father`。

然而，上面的代码是无法编译通过的，错误出在最后一行代码 `mainClass.max(sons)`，我们一起来分析下为什么。

`max()` 方法通过 `T extends Comparable<T>` 对形参进行限定，这就要求参与比较的元素 `T` 必须实现 `Comparable<T>` 接口，且只能相同类型的数据进行比较。

我们看到 `Father` 类的声明是 `Father implements Comparable<Father>`，所以 `Son extends Father` 等价于 `Son extends Father implements Comparable<Father>`，也就是说 `Son` 实现的是 `Comparable<Father>` 接口，显然 `Son` 的定义不符合 `max()`  方法对形参的限定，从而引发编译报错。

事实上，在本例中 `Son` 复用 `Father` 的 `Comparable` 接口和 `compareTo` 方法是合理的需求，符合程序设计的原则。此时，如何使得类  `Son` 能够参与比较呢？这就需要借助超类通配符来修改 `max()` 方法的声明：

```java
<T extends Comparable<? super T>> void max(List<T> list) { }
```

修改后，将 `Son` 代入上述类型形参的声明，`<Son extends Comparable<? super Son>>` 显然符合 `Son implements Comparable<Father>`，所以程序得以正确编译并运行。

## 4. 类型通配符的局限

类型通配符能够使得代码更加精简，但也存在诸多局限性。

### 4.1 只能读不能写

这一点是使用类型通配符必须要注意的，举个例子：

```java
List<?> list = new ArrayList<>();
String str = "LoulanPlan";
list.add(str);//编译报错
```

我们尝试将 `String` 类型数据写入 `List<?>` 集合中，看似合理，但实际是无法编译通过的。

原因：`List<?>` 表示集合元素类型未知，如果允许写入必将引发类型安全的问题，所以 Java 禁止进行写入操作。

反过来分析下，如果允许写入会有什么问题：

```java
List<?> listA;
List<String> listB = new ArrayList<>();
//listA 指向 ListB
listA = listB;
//向集合中添加 Integer 类型数数据（实际上 Java 不允许这么做）
listA.add(1);
```

通过 `listA = listB` 将 `listA` 指向了 `List<String>` 类型的 `ListB`，如果 Java 允许执行 `listA.add(1)` 的操作，就意味着可以向 `List<String>` 中添加 `Integer` 类型的数据，这显然违背了 Java 强类型的原则。

### 4.2 无法声明依赖关系

使用类型形参，可以通过如下方式声明参数间的依赖关系：

```java
<E, S extends E> void copy(List<S> src, List<E> dest) {}
```

这种参数间的依赖关系，是无法使用类型通配符完全替代的。这点很好理解，大家都是未知类型，谈何依赖？

但有一点，使用类型通配符能够简化上述方法的声明：

```java
<E> void copy(List<? extends E> src, List<E> dest) {}
```

### 4.3 无法作为方法的返回值

这一点是没什么好说的，方法的返回值必须是明确的数据类型，类型未知的通配符显然无法作为方法的返回值。

## 5. 小结

有关类型通配符的介绍就到这了，至此我们已经掌握了 Java 泛型中类型参数和通配符的是使用。下一篇将作为 Java 泛型的完结篇，分析下泛型的实现原理及局限性，以帮助大家在开发中使用泛型时，有的放矢且游刃有余。
