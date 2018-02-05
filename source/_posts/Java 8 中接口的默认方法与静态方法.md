---
title: Java 8 中接口的默认方法与静态方法
date: 2018-02-01 15:48:19
categories:
    - Java
tags: 
    - Java
    - Java 8
---

> 在 Java 8 中，接口引入了一些新的语言特性：默认方法（Default Methods）以及静态方法（Static Methods）。本篇文章就来了解下这两个特性。

<!-- more -->

## 默认方法
### 默认方法的定义
默认方法的定义需要用到 default 关键字，例如：
``` java
public interface A {
    default String getName(){
        return "A";
    }
}
```
默认方法不可以被 private 修饰，默认就是 public 的，public 可以省略不写。

### 默认方法的使用
当有类实现了该接口后，可以直接调用该默认方法，例如：
``` java
public class C implements A{
    
}
```
``` java
public class Main{
    public static void main(String[] args) {
        System.out.println(new C().getName());// A
    }
}
```
同时，在具体的接口实现类中也可以对接口中的默认方法进行覆写。例如：
``` java
public class C implements A{
    @Override
    public String getName() {
        return "C";
    }
}
```
``` java
public class Main{
    public static void main(String[] args) {
        System.out.println(new C().getName());// C
    }
}
```

### 默认方法的继承
大家都知道，接口与接口之间是可以继承的。那么在有默认方法的情况下，接口的继承又是什么样子的呢？
我们来看看下面的继承例子：
``` java
public interface A {
    default String getName(){
        return "A";
    }
}
```
``` java
public interface B extends A{

}
```
``` java
public interface C extends B{
    @Override
    default String getName() {
        return "C";
    }
}
```
``` java
public interface D extends C{
    @Override
    String getName();
}
```
``` java
public class Main{
    public static void main(String[] args) {
        System.out.println(new A(){}.getName());// A
        System.out.println(new B(){}.getName());// A
        System.out.println(new C(){}.getName());// C
        System.out.println(new D(){
            @Override
            public String getName() {
                return "D";
            }
        }.getName());// D
    }
}
```
通过打印结果，我们可以发现其继承关系有这么几种情况：

- 不覆写父接口的默认方法，直接用父接口的方法实现。（参考 B 接口）
- 覆写父接口的默认方法，用自己的实现。（参考 C 接口）
- 覆写父接口的默认方法实现，并将其更改为抽象方法。（参考 D 接口）

前两个很好理解，就跟类的继承是一样的。第三个呢，当把默认方法改为抽象方法后，那么实现类就必须去实现该抽象方法了。

### 为什么要有默认方法
想象一个场景，假如在没有默认方法的情况下我们需要对集合框架增加一个新的迭代功能（forEach），那么我们就需要对其 Iterable 接口增加一个新的抽象方法，但是这样做的后果就是该接口下的所有实现类都必须去实现该方法。可是由于集合框架体系的庞大，实现类有非常多个，一改动全身，影响实在是太大，并不可取。

但是当有默认方法时，就解决了这个问题，就算在接口中增加了新的默认方法，具体的实现类也无需改动就有了新功能。

默认方法的出现就是为了保证原有代码兼容性的同时在接口中增加新的功能。

### 默认方法带来的问题
默认方法虽好，但是我们也不可以滥用。因为默认方法会在复杂的继承关系中给我们的程序带来歧义以及编译错误。

由于接口是可以被多继承的，所以我们来看下面这种情况：
``` java
public interface A {
    default String getName(){
        return "A";
    }
}
```
``` java
public interface B {
    default String getName(){
        return "B";
    }
}
```
``` java
public class C implements A, B{

}
```
当 C 类同时实现 A、B 接口时，编译是无法通过的。很明显，因为 A、B 接口中都有 getName() 方法，要是能编译通过的话，程序怎么会知道调用哪个呢？所以出现这种情况，C 类必须去覆写 getName() 方法，然后给出自己的想要的 “结果”，例如，用 A 的 “结果”：
``` java
@Override
public String getName() {
    return A.super.getName();
}
```
或者说是自己给一个 "结果"：
``` java
@Override
public String getName() {
    return "C";
}
```
当最终调用的时候，打印结果就是：A、C 了。

另外还有一种稍复杂的情况，来看下这个例子：
``` java
public interface A {
    default String getName(){
        return "A";
    }
}
```
``` java
public interface B extends A{
    @Override
    default String getName() {
        return "B";
    }
}
```
``` java
public class Main{

    static class C implements A, B{

    }

    static class D implements A, B{
        @Override
        public String getName() {
//            return A.super.getName();// 报错
            return B.super.getName();// 正确
        }
    }

    public static void main(String[] args) {
        System.out.println(new C().getName());// B
        System.out.println(new D().getName());// B
    }
}
```
C 这个类同时实现了 A 、B 接口，怎么就不报错让其实现 getName() 方法了呢？仔细看，B 和 A 是继承的关系。当 B 继承 A 后并覆写了 getName() 方法，那么 A 的 getName() 其实已经被屏蔽了。当同时实现这两个接口时，打印出的自然是 B 的 "结果" 了。同理，D 类中不可用 A 接口的实现也是如此。

> 这是当接口继承行为发生冲突时的规则之一，即 被其他类型所覆盖的方法会被忽略。

但是，我们就是想要 A 接口的结果，这怎么办？我们可以来新定义一个接口来解决该问题：
``` java
public interface A {
    default String getName(){
        return "A";
    }
}
```
``` java
public interface B extends A{
    @Override
    default String getName() {
        return "B";
    }
}
```
``` java
public interface C extends A{
    @Override
    default String getName() {
        return A.super.getName();// 注意，这里必须去调用 A 接口的实现
    }
}
```
``` java
public class Main{

    static class D implements B, C{
        @Override
        public String getName() {
            return C.super.getName();
        }
    }

    public static void main(String[] args) {
        System.out.println(new D().getName());// A
    }
}
```
绕了一圈，总算是得到 A 的结果了。在该例子中，我们需要注意的是，在新定义的接口中，必须去调用 A 接口的实现，如果不调用的话，A 接口中的 getName() 方法依然会被 B 接口覆写导致屏蔽。切记切记！

## 默认方法总结
默认方法给予我们修改接口并且不破坏原有实现类提供了便利。但是同时也带来了一些问题，比如上文中提到了继承冲突。以及接口与类的关系由约束及实现而变得更加模糊。因此，我们应该在定义默认方法之前思考下是否真的必须这样做。

## 静态方法
从 Java 8 开始，接口除了可以定义默认方法外，还可以定义静态方法。

接口中静态方法的定义和类中的定义差不多：
``` java
public interface A {
    static void print(){
        System.out.println("A");
    };   
}
```
只不过在接口中，默认是由 public 所修饰的，并且同默认方法一样，是不可以更改为 private 的。

调用时，直接接口名.方法名：
``` java
A.print();
```