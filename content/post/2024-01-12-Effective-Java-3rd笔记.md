---
title : 'Effective Java 3rd笔记'
description: ' Effective Java 3rd笔记'
date : '2024-01-12T21:59:45+08:00'
author:     'Dylan'
tags:        ['Java']
categories:  ['编程','笔记']
---

记录《Effectice Java 第三版》

<!--more-->
# 1. 对象创建和销毁
## 1.1 使用静态工厂方法替代构造函数

- 优势
   - 静态工厂方法的第三个优点是，与构造方法不同，它们可以返回其返回类型的任何子类型的对象
   - 静态工厂的第四个优点是返回对象的类可以根据输入参数的不同而不同
   - 在编写包含该方法的类时，返回的对象的类不需要存在。
- 限制
   - 没有Public或protected构造方法的类不能被子类化
   - 静态工厂方法的第二个缺点是，程序员很难找到它们
## 1.2 参数过多时使用Builder模式
JavaBeans 模式本身有严重的缺陷。由于构造方法在多次调用中被分割，所以在构造过程中 JavaBean 可能处于不一致的状态
## 1.3 使用私有构造方法或枚类实现 Singleton 属性
### public final field
```java
// Singleton with public final field
public class Elvis {
    public static final Elvis INSTANCE = new Elvis();
    private Elvis() { ... }
    public void leaveTheBuilding() { ... }
}
```
###  static factory
```java
// Singleton with static factory
public class Elvis {
private static final Elvis INSTANCE = new Elvis();
private Elvis() { ... }
public static Elvis getInstance() { return INSTANCE; }
public void leaveTheBuilding() { ... }
}
```
### Enum singleton 
```java
// Enum singleton - the preferred approach
public enum Elvis {
INSTANCE;
public void leaveTheBuilding() { ... }
}
```
## 1.4 使用私有构造方法执行非实例化
> 偶尔你会想写一个类，它只是一组静态方法和静态属性。 这样的类获得了不好的名声，因为有些人滥用这些类 
> 而避免以面向对象方式思考，但是它们确实有着特殊的用途。 它们可以用来按照 java.lang.Math 或 
> java.util.Arrays 的方式，在基本类型的数值或数组上组织相关的方法。

## 1.5 使用依赖注入取代硬连接资源
> 当一个类A依赖另一个类B时 不要直接在A中new B ,以参数形式注入B.如果B有多种 可提供接口

## 1.6 避免创建不必要的对象

- 尽可能重用对象
- 优先使用基础类型而不是自动装箱
- 不要使用 String s = new String("bikini");
## 1.7 消除过期的对象引用

- 当一个类自己管理内存时，程序员应该警惕内存泄漏问题；每当一个元素被释放时，元素中包含的 任何对象引用都应该被清除
- 另一个常见的内存泄漏来源是缓存
   - WeakHashMap
- 第三个常见的内存泄漏来源是监听器和其他回调
## 1.8 避免使用 Finalizer 和 Cleaner 机制

- 弊端
   - Finalizer 和 Cleaner 机制的一个缺点是不能保证他们能够及时执行
   - 使用 finalizer 和 cleaner 机制会导致严重的性能损失
   - finalizer 机制有一个严重的安全问题
- 合法用途
   - 一个是作为一个安全网（safety net），以防资源的拥有者忽略了它的 close 方法
   - Cleaner 机制的方法与本地对等类（native peers）有关本地对等类是一个由普通对象委托的本 地 (非 Java) 对象。由于本地对等类不是普通的 Java 对象，所以垃圾收集器并不知道它，当它的 Java 对等对象被回 收时，本地对等类也不会回收。假设性能是可以接受的，并且本地对等类没有关键的资源，那么 Finalizer 和 Cleaner 机制可能是这项任务的合适的工具
- 总结：
   - 除了作为一个安全网或者终止非关键的本地资源，不要使用 Cleaner 机制，或者是在 Java 9 发布之前的 finalizers 机制。即使是这样，也要当心不确定性和性能影响
## 1.9 使用 try-with-resources 语句替代 try-finally 语句

- 简单明确
# 2. 所有对象通用方法
> 虽然 Object 是一个具体的类，但它主要是为继承而设计的

## 2.1 重写 equals 方法时遵守通用约定
### 不需要重写equals情况

- 每个类的实例都是固有唯一的
- 类不需要提供一个“逻辑相等（logical equality）”的测试功能
- 父类已经重写了 equals 方法，则父类行为完全适合于该子类
- 类是私有的或包级私有的，可以确定它的 equals 方法永远不会被调用
### 等价关系

- 自反性
   - 对于任何非空引用 x， x.equals(x) 必须返回 true
- 对称性
   - 对于任何非空引用 x 和 y，如果且仅当 y.equals(x) 返回 true 时 x.equals(y) 必须返回 true
- 传递性
   - 对于任何非空引用 x、y、z，如果 x.equals(y) 返回 true， y.equals(z) 返回 true，则 x.equals(z) 必须返回 true。
- 一致性
   - 对于任何非空引用 x 和 y，如果在 equals 比较中使用的信息没有修改，则 x.equals(y) 的多次调用 必须始终返回 true 或始终返回 false
- 非空性
   - 对于任何非空引用 x， x.equals(null) 必须返回 false
### 如何写高质量equals

- 1.使用 == 运算符检查参数是否为该对象的引用。如果是，返回 true。这只是一种性能优化，但是如果这种比较可 能很昂贵的话，那就值得去做
- 2.使用 instanceof 运算符来检查参数是否具有正确的类型。 如果不是，则返回 false。 通常，正确的类型是 equals 方法所在的那个类。 有时候，改类实现了一些接口。 如果类实现了一个接口，该接口可以改进 equals 约 定以允许实现接口的类进行比较，那么使用接口。 集合接口（如 Set，List，Map 和 Map.Entry）具有此特性
- 3.参数转换为正确的类型。因为转换操作在 instanceof 中已经处理过，所以它肯定会成功
- 4.对于类中的每个“重要”的属性，请检查该参数属性是否与该对象对应的属性相匹配。如果所有这些测试成功， 返回 true，否则返回 false。如果步骤 2 中的类型是一个接口，那么必须通过接口方法访问参数的属性;如果类型 是类，则可以直接访问属性，这取决于属性的访问权限
### 重要

- 当重写 equals 方法时，同时也要重写 hashCode 方法
   - 相等的对象必须具有相等的哈希码
- 不要让 equals 方法试图太聪明
   - 过于激进
- 在 equal 时方法声明中，不要将参数 Object 替换成其他类型
## 2.2  重写 equals 方法时同时也要重写 hashcode 方法
>  如果不这样做，你的类违反了 hashCode 的通用约定，这会阻止它在 HashMap 和 HashSet 这样的集合中正常工作

Object规范

- 当在一个应用程序执行过程中，如果在 equals 方法比较中没有修改任何信息，在一个对象上重复调用 hashCode 方法时，它必须始终返回相同的值。从一个应用程序到另一个应用程序的每一次执行返回的值可以是 不一致的
- 如果两个对象根据 equals(Object) 方法比较是相等的，那么在两个对象上调用 hashCode 就必须产生的结果是相 同的整数。
- 如果两个对象根据 equals(Object) 方法比较并不相等，则不要求在每个对象上调用 hashCode 都必须产生不同的 结果。 但是，程序员应该意识到，为不相等的对象生成不同的结果可能会提高散列表（hash tables）的性能
## 2.3 始终重写 toString 方法
> 它使得类更加舒适 
> 地使用和协助调试。 toString 方法应该以一种美观的格式返回对象的简明有用的描述

## 2.4 谨慎地重写 clone 方法

- Object 的 clone 方法是受保护的
- Cloneable 接口的目的是作为一个 mixin 接口  不包含任何方法
- Cloneable 接口不包含任何方法，那它用来做什么
   - 它决定了 Object 的受保护的 clone 方法实现的行为：如 果一个类实现了 Cloneable 接口，那么 Object 的 clone 方法将返回该对象的逐个属性（field-by-field）拷贝；否则会抛 出 CloneNotSupportedException 异常
## 2.5 考虑实现 Comparable 接口
> 实现具有合理排序的值类，你都应该让该类实现 Comparable 接口，以便在基于比较的集合中轻松对其实例进行排序，搜索和使用

# 3 类和接口
## 3.1 使类和成员的可访问性最小化
> 一个设计良好的组件隐藏了它的所有实现细节，干净地将它的 API 与它的实现分离开来 ;组件只通过它们的 API 进行通信，并且对彼此的内部工作一无所知.
> 这一概念，被称为信息隐藏或封装，是软件设计的基本原则

### 隐藏信息优势

1. 组件分离开来，允许它们被独立地开发，测试，优化，使用，理解和修改
2. 增加了软件重用
3. 藏信息降低了构建大型系统的风险，因为即使系统不能运行，各个独立的组件也可能是可用的

使用尽可能低的访问级别
## 3.2 在公共类中使用访问方法而不是公共属性
对于可修改public 类的属性时 提供方法而不是使用public修饰属性.
## 3.3 最小化可变性
> 不可变类简单来说是它的实例不能被修改的类

### 不可变类规则

- 不要提供修改对象状态的方法
- 确保这个类不能被继承
- 把所有属性设置为 final
- 把所有的属性设置为 private
- 确保对任何可变组件的互斥访问
### 优点

- 不可变对象本质上是线程安全的; 它们不需要同步
- 不仅可以共享不可变的对象，而且可以共享内部信息
- 不可变对象为创建复杂对象提供便利(Immutable objects make great building blocks for other objects)
   - 无论是可变的还是不可变的。 如果知道一个复杂组件的内部对象不会发生改变，那么维护复杂对象的不变量就容易多了。这一原则的特例是，不可变对象可以构成Map 对象的键和 Set 的元素，一旦不可变对象作为 Map 的键或 Set 里的元素，即使破坏了 Map 和 Set的不可变性，但不用担心它们的值会发生变化
- 不可变对象提供了免费的原子失败机制
### 缺点
不可变类的主要缺点是对于每个不同的值都需要一个单独的对象
### 总结

1. 坚决不要为每个属性编写一个 get 方法后再编写一个对应的 set 方法。 除非有充分的理由使类成为可变类，否则类应该是不可变的
2. 如果一个类不能设计为不可变类，那么也要尽可能地限制它的可变性 ，除非有充分的理由不这样做，否则应该把每个属性声明为私有 final 的
3. 构造方法应该创建完全初始化的对象，并建立所有的不变性
## 3.4 组合优于继承
> 与方法调用不同，继承打破了封装  换句话说，一个子类依赖于其父类的实现细节来保证其正确的功能。 父类的实现可能会从发布版本不断变化，如果是这样，子类可能会被破坏，即使它的代码没有任何改变


> 只有在子类真的是父类的子类型的情况下，继承才是合适的。 换句话说，只有在两个类之间存在“is-a”关系的情况下，B 类才能继承 A 类。 如果你试图让 B 类继承 A 类时，
> 问自己这个问题：每个 B 都是 A 吗？ 如果你不能如实回答这个问题，那么 B 就不应该继承 A

### 总结
继承是强大的，但它是有问题的，因为它违反封装。 只有在子类和父类之间存在真正的子类型关系时才适用。 即使如此，如果子类与父类不在同一个包中，并且父类不是为继承而设计的，继承可能会导致脆弱性。 为了
避免这种脆弱性，使用合成和转发代替继承，特别是如果存在一个合适的接口来实现包装类。 包装类不仅比子类更健壮，而且更强大.
## 3.5 如使用继承则设计，应当文档说明，否则不该使用
这个类必须准确地描述重写这个方法带来的影响 ，用可重写方法的方法在文档注释结束时包含对这些调用的描述  这些描述在规范中特定部分，标记
为“Implementation Requirements”，由 Javadoc 标签 @implSpec 生成
```java
/**
* {@inheritDoc}
*
* @implSpec
* This implementation iterates over the collection looking for the
* specified element.  If it finds the element, it removes the element
* from the collection using the iterator's remove method.
*
* <p>Note that this implementation throws an
* {@code UnsupportedOperationException} if the iterator returned by this
* collection's iterator method does not implement the {@code remove}
* method and this collection contains the specified object.
*
* @throws UnsupportedOperationException {@inheritDoc}
* @throws ClassCastException            {@inheritDoc}
* @throws NullPointerException          {@inheritDoc}
*/
public boolean remove(Object o) {}
```
 除非你知道有一个真正的子类需要，否则你可能最好是通过声明你的类为 final 禁止继承，或者确保没有可访问的构造方法
## 3.6 接口优于抽象类
> Java 有两种机制来定义允许多个实现的类型：接口和抽象类。 由于在 Java 8 [JLS 9.4.3] 中引入了接口的默认方
> 法（default methods ），因此这两种机制都允许为某些实例方法提供实现。 一个主要的区别是要实现由抽象类定义
> 的类型，类必须是抽象类的子类。 因为 Java 只允许单一继承，所以对抽象类的这种限制严格限制了它们作为类型定
> 义的使用。 任何定义所有必需方法并服从通用约定的类都可以实现一个接口，而不管类在类层次结构中的位置


接口是定义混合类型（mixin）的理想选择。 厨师和歌手两个歌手 。一个人可能既是厨师又是歌手
接口比抽象类更容易扩展
接口默认方法的限制、接口接口不允许包含实例属性或非公共静态成员（私有静态方法除外）
考虑骨架的设计 抽象类配合接口  例如 java中 ArrayList 和 HashMap等

### 总结
总而言之，一个接口通常是定义允许多个实现的类型的最佳方式。 如果你导出一个重要的接口，应该强烈考虑
提供一个骨架的实现类。 在可能的情况下，应该通过接口上的默认方法提供骨架实现，以便接口的所有实现者都可
以使用它。 也就是说，对接口的限制通常要求骨架实现类采用抽象类的形式

## 3.7 为后代设计接口
> 在 Java 8 之前，不可能在不破坏现有实现的情况下为接口添加方法。 如果向接口添加了一个新方法，现有的实
> 现通常会缺少该方法，从而导致编译时错误。 在 Java 8 中，添加了默认方法（default method）构造[JLS 9.4]，目的
> 是允许将方法添加到现有的接口

默认方法的声明包含一个默认实现，该方法允许实现接口的类直接使用，而不必实现默认方法
**default接口特性**

1. 实现类会继承接口中的default方法,并且不强制实现类重写此方法
2. 当一个实现类实现了多个接口，多个接口里都有相同的默认方法时，实现类必须重写该默认方法，否则编译错误
3. 类优先于接口 如果子类继承父类，父类中有b方法，该子类同时实现的接口中也有b方法（被default修饰），那么子类会继承父类的b方法而不是继承接口中的b方法
## 3.8 接口仅用来定义类型
接口只能用于定义类型。 它们不应该仅用于导出常量
## 3.9 优先类继承而不是tagged classes
tagged classes
```java
// Tagged class - vastly inferior to a class hierarchy!
class Figure {
enum Shape { RECTANGLE, CIRCLE };
// Tag field - the shape of this figure
final Shape shape;
// These fields are used only if shape is RECTANGLE
double length;
double width;
// This field is used only if shape is CIRCLE
double radius;
// Constructor for circle
Figure(double radius) {
shape = Shape.CIRCLE;
this.radius = radius;
}
// Constructor for rectangle
Figure(double length, double width) {
shape = Shape.RECTANGLE;
this.length = length;
this.width = width;
}
double area() {
switch(shape) {
case RECTANGLE:
return length * width;
case CIRCLE:
return Math.PI * (radius * radius);
default:
throw new AssertionError(shape);
}
}
}
```

- 可读性差
- 容易出错、效率低下

继承类
```java
// Class hierarchy replacement for a tagged class
abstract class Figure {
abstract double area();
}
class Circle extends Figure {
final double radius;
Circle(double radius) { this.radius = radius; }
    @Override double area() { return Math.PI * (radius * radius); }
}
class Rectangle extends Figure {
final double length;
final double width;
Rectangle(double length, double width) {
this.length = length;
this.width = width;
}
@Override double area() { return length * width; }
}
```
## 3.10 优先考虑静态成员类
>  有四种嵌套类：静态成员类，非静态成员类，匿名类和局部类。 除了第一种以外，剩下的三种都被称为内部类（inner class）

### 静态成员类
静态成员类是最简单的嵌套类。
 最好把它看作是一个普通的类，恰好在另一个类中声明，并且可以访问所有宿主类的成员，甚至是那些被声明为私有类的成员。 静态成员类是其宿主类的静态成员，并遵循与其他静态成员相同的可访问性规则.
> 如果你声明了一个不需要访问宿主实例的成员类，总是把 static 修饰符放在它的声明中，使它成为一个静态成员
> 类，而不是非静态的成员类

### 非静态成员类
非静态成员类的每个实例都隐含地与其包含的类的宿主实例相关联。 
在非静态成员类的实例方法中，可以调用宿主实例上的方法。
如果嵌套类的实例可以与其宿主类的实例隔离存在，那么嵌套类必须是静态成员类：不可能在没有宿主实例的情况下创建非静态成员类的实例
非静态成员类实例和其宿主实例之间的关联是在创建成员类实例时建立的，并且之后不能被修改
### 匿名类
 它不是其宿主类的成员,
不能实例化它们 
不能执行 instanceof 方法测试或者做任何其他需要你命名的类
 不能声明一个匿名类来实现多个接口，或者继承一个类并同时实现一个接口。
匿名类的客户端不能调用除父类型继承的成员以外的任何成员
### 局部类
```java
public void function(){
            class abc {
                void abc(){}
            }
            new abc().abc();
        }
```
### 总结
> 有四种不同的嵌套类，每个都有它的用途。 如果一个嵌套的类需要在一个方法之外可见，或者太长而不能很好地适应一个方法，使用一个成员类。 
> 如果一个成员类的每个实例都需要一个对其宿主实例的引用，使其成为非静态的; 否则，使其静态。
>  假设这个类属于一个方法内部，如果你只需要从一个地方创建实例，并且存在一个
> 预置类型来说明这个类的特征，那么把它作为一个匿名类; 否则，把它变成局部类

## 3.11 将源文件限制为单个顶级类
> Java 编译器允许在单个源文件中定义多个顶级类，但这样做没有任何好处，并且存在重大风险。 风险源于在源文件中定义多个顶级类使得为类提供多个定义成为可能。 使用哪个定义会受到源文件传递给编译器的顺序的影响


永远不要将多个顶级类或接口放在一个源文件中。 遵循这个规则保证在编译时不能有多个定义。 这又保证了编译生成的类文件以及生成的程序的行为与源文件传递给编译器的顺序无关
```java
// 同一个文件定义两个类
class A {

}
class B{

}

```
# 4 泛型
java 5 泛型为集合提供便利。编译器会自动插入强制转换，并在编译时告诉你是否尝试插入错误类型的对象。 这样做的结果是既安全又清晰的程序，但这些益处，不限于集合，是有代价的
## 4.1  不要使用原始类型
没有泛型的集合
```java
// Raw collection type - don't do this!
// My stamp collection. Contains only Stamp instances.
private final Collection stamps = ... ;
```
> 如果你使用原始类型，则会丧失泛型的所有安全性和表达上的优势

使用原始类型可能导致运行时异常，所以不要使用它们。 它们仅用于与泛型引入之前的传统代码的兼容
性和互操作性。
 Set<Object> 是一个参数化类型，表示一个可以包含任何类型对象的集合，
Set<?> 是一个通配符类型，表示一个只能包含某些未知类型对象的集合，
 Set 是一个原始类型，它不在泛型类型系统之列。 前两个类型是安全的，最后一个不是
![image.png](https://cdn.nlark.com/yuque/0/2022/png/22342229/1665472326335-7d8bcd71-a8b3-4f63-9862-bba6298b8d7d.png#clientId=u0c30c3fe-9225-4&from=paste&height=443&id=u34dc7e9c&originHeight=443&originWidth=604&originalType=binary&ratio=1&rotation=0&showTitle=false&size=72042&status=done&style=none&taskId=ue695784a-4da6-4159-9a57-5ccad6b609f&title=&width=604)
## 4.2 消除非检查警告
> 每个未经检查的警告代表在运行时出现ClassCastException 异常的可能性。 尽你所能消除这些警告。 如果无法消除未经检查的警告，并且可以证明引
> 发该警告的代码是安全类型的，则可以在尽可能小的范围内使用 @SuppressWarnings(“unchecked”) 注解来禁止警告。 记录你决定在注释中抑制此警告的理由。

## 4.3 列表优于数组
> 数组和泛型具有非常不同的类型规则。 数组是协变和具体化的; 泛型是不变的，类型擦除的。 因此，数组
> 提供运行时类型的安全性，但不提供编译时类型的安全性，反之亦然。 一般来说，数组和泛型不能很好地混合工
> 作。 如果你发现把它们混合在一起，得到编译时错误或者警告，你的第一个冲动应该是用列表来替换数组

```java
// 编译通过 运行时异常
Object[] objectArray = new Long[1];
objectArray[0] = "I don't fit in"; // Throws ArrayStoreException


// 无法编译
List<Object> ol = new ArrayList<Long>(); // Incompatible types
ol.add("I don't fit in");
```
## 4.4  优先考虑泛型
> 泛型类型比需要在客户端代码中强制转换的类型更安全，更易于使用。 当你设计新的类型时，确保它们
> 可以在没有这种强制转换的情况下使用。 这通常意味着使类型泛型化。 如果你有任何现有的类型，应该是泛型的但
> 实际上却不是，那么把它们泛型化

## 4.5 优先考虑方法
定义工具方法时优先考虑使用泛型方法
## 4.6 用限定通配符来增加 API 的灵活性

```java
   boolean addAll(Collection<? extends E> c);
 boolean removeAll(Collection<?> c);
```
>  为了获得最大的灵活性，对代表生产者或消费者的输入参数使用通配符类型 
>  PECS:  producer-extends，consumer-super

> 换句话说，如果一个参数化类型代表一个 T 生产者，使用 <? extends T> ；
> 如果它代表 T 消费者，则使用 <? super T> 。 
> 在我们的 Stack 示例中， pushAll 方法的 src 参数生成栈使用的 E 实例，因此 src的合适类型为 Iterable<? extends E> ； 
> popAll 方法的 dst 参数消费 Stack 中的 E 实例，因此 dst的合适类型是 C ollection <? super E> 。 PECS 助记符抓住了使用通配符类型的基本原则

## 4.7 合理地结合泛型和可变参数
```java
//Possible heap pollution from parameterized vararg type 
     public static <T> void test(T... list){
        System.out.println(list.length);
    }
```
> 可变参数和泛型不能很好地交互，因为可变参数机制是在数组上面构建的脆弱的抽象，并且数组具有与泛型不同的类型规则。 
> 虽然泛型可变参数不是类型安全的，但它们是合法的。 如果选择使用泛型（或参数化）可变参数编写方法，请首先确保该方法是类型安全的，然后使用 @SafeVarargs 注解对其进行标注，以免造成使用不愉快.

## 4.8 优先考虑类型安全的异构容器
异构容器可以存放不同的类型
类型安全的异构容器
```java
// Typesafe heterogeneous container pattern - implementation
public class Favorites {
private Map<Class<?>, Object> favorites = new HashMap<>();
public<T> void putFavorite(Class<T> type, T instance) {
favorites.put(Objects.requireNonNull(type), instance);
}
public<T> T getFavorite(Class<T> type) {
return type.cast(favorites.get(type));
}
}
```
> 泛型 API 的通常用法（以集合 API 为例）限制了每个容器的固定数量的类型参数。 你可以通过将类型参数
> 放在键上而不是容器上来解决此限制。 可以使用 Class 对象作为此类型安全异构容器的键。 以这种方式使用的
> Class 对象称为类型令牌。 也可以使用自定义键类型。 例如，可以有一个表示数据库行（容器）的
> DatabaseRow 类型和一个泛型类型 Column<T> 作为其键

# 5 枚举和注解
## 5.1  Use enums instead of int constants
int 枚举模式的技术有许多缺点。 它没有提供类型安全的方式，也没有提供任何表达力
```java
// The int enum pattern - severely deficient!
public static final int APPLE_FUJI = 0;
public static final int APPLE_PIPPIN = 1;
public static final int APPLE_GRANNY_SMITH = 2;

public static final int ORANGE_NAVEL = 0;
public static final int ORANGE_TEMPLE = 1;
public static final int ORANGE_BLOOD = 2;
```

>  Java 的枚举类型是完整的类，比其他语言中的其他语言更强大，其枚举本质本上是 int 值
> 

枚举更具可读性，更安全，更强大
```java
public enum Apple { FUJI, PIPPIN, GRANNY_SMITH }
public enum Orange { NAVEL, TEMPLE, BLOOD }
```
> Java 枚举类型背后的基本思想很简单：它们是通过公共静态 final 属性为每个枚举常量导出一个实例的类。
> 由于没有可访问的构造方法，枚举类型实际上是 final 的。 由于客户既不能创建枚举类型的实例也不能继承它，
> 除了声明的枚举常量外，不能有任何实例。 换句话说，枚举类型是实例控制的（第 6 页）。 它们是单例（条目 3）
> 的泛型化，基本上是单元素的枚举


```java
// Enum type that switches on its own value - questionable
public enum Operation {
 PLUS, MINUS, TIMES, DIVIDE;
// Do the arithmetic operation represented by this constant
public double apply(double x, double y) {
switch(this) {
case PLUS: return x + y;
case MINUS: return x - y;
case TIMES: return x * y;
case DIVIDE: return x / y;
}
throw new AssertionError("Unknown op: " + this);
}
}



// Enum type with constant-specific method implementations
public enum Operation {
PLUS {public double apply(double x, double y){return x + y;}},
MINUS {public double apply(double x, double y){return x - y;}},
TIMES {public double apply(double x, double y){return x * y;}},
DIVIDE{public double apply(double x, double y){return x / y;}};
public abstract double apply(double x, double y);
}
```
## 5.2 Use instance fields instead of ordinals
> 许多枚举通常与单个 int 值关联。所有枚举都有一个 ordinal 方法，它返回每个枚举常量类型的数值位
> 置。public abstract class Enum<E extends Enum<E>> 中的 ordinal

**永远不要从枚举的序号中得出与它相关的值; 请将其保存在实例属性中**
> 枚举规范对此 ordinal 方法说道：“大多数程序员对这种方法没有用处。 它被设计用于基于枚举的通用数据结
> 构，如 EnumSet 和 EnumMap 。“除非你在编写这样数据结构的代码，否则最好避免使用 ordinal 方法。

```java
    /**
     * The ordinal of this enumeration constant (its position
     * in the enum declaration, where the initial constant is assigned
     * an ordinal of zero).
     *
     * Most programmers will have no use for this field.  It is designed
     * for use by sophisticated enum-based data structures, such as
     * {@link java.util.EnumSet} and {@link java.util.EnumMap}.
     */
    private final int ordinal;
```
## 5.3 Use EnumSet instead of bit fields
> 如果枚举类型的元素主要用于集合中

### int枚举模式 实现集合操作
```java
// Bit field enumeration constants - OBSOLETE!
public class Text {
public static final int STYLE_BOLD = 1 << 0; // 1
public static final int STYLE_ITALIC = 1 << 1; // 2
public static final int STYLE_UNDERLINE = 1 << 2; // 4
public static final int STYLE_STRIKETHROUGH = 1 << 3; // 8
// Parameter is bitwise OR of zero or more STYLE_ constants
public void applyStyles(int styles) { ... }
}

//
text.applyStyles(STYLE_BOLD | STYLE_ITALIC);
```
### EnumSet
```java
// EnumSet - a modern replacement for bit fields
public class Text {
public enum Style { BOLD, ITALIC, UNDERLINE, STRIKETHROUGH }
// Any Set could be passed in, but EnumSet is clearly best
public void applyStyles(Set<Style> styles) { ... }
}
```
## 5.4 Use EnumMap instead of ordinal indexing
## 5.5 使用接口模拟可扩展的枚举
枚举类不能扩展 但是接口类可以
```java
// Emulated extensible enum using an interface
public interface Operation {
    double apply(double x, double y);
}

public enum BasicOperation implements Operation {
    PLUS("+") {
        public double apply(double x, double y) {
            return x + y;
        }
    },
    MINUS("-") {
        public double apply(double x, double y) {
            return x - y;
        }
    },
    TIMES("*") {
        public double apply(double x, double y) {
            return x * y;
        }
    },
    DIVIDE("/") {
        public double apply(double x, double y) {
            return x / y;
        }
    };
    private final String symbol;

    BasicOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}


// Emulated extension enum
public enum ExtendedOperation implements Operation {
    EXP("^") {
        public double apply(double x, double y) {
            return Math.pow(x, y);
        }
    },
    REMAINDER("%") {
        public double apply(double x, double y) {
            return x % y;
        }
    };
    private final String symbol;

    ExtendedOperation(String symbol) {
        this.symbol = symbol;
    }

    @Override
    public String toString() {
        return symbol;
    }
}
```
## 5.6 注解优于命名模式
> 通常使用命名模式（naming patterns）来指示某些程序元素需要通过工具或框架进行特殊处理
> JUnit 3测试框架要求其用户通过以 test 开头的方法名来指定测试方法

## 5.7  始终使用 Override 注解
> 此注解只能在方法声明上使用，它表明带此注解的方法声明重写了父类的声明 如果始终使用这个注解，它将避免产生大量的恶意 bug

如下代码 
目的重写Object的  equals hashCode 使得主程序输出26
```java
public class Bigram {
    private final char first;
    private final char second;

    public Bigram(char first, char second) {
        this.first = first;
        this.second = second;
    }
    public boolean equals(Bigram b) {
        return b.first == first && b.second == second;
    }
    public int hashCode() {
        return 31 * first + second;
    }
// 修改后的代码
//    @Override
//    public boolean equals(Object b) {
//        if (!(b instanceof Bigram)) {
//            return false;
//        }
//        Bigram bb = (Bigram) b;
//        return bb.first == first && bb.second == second;
//    }
//
//    @Override
//    public int hashCode() {
//        return 31 * first + second;
//    }

    public static void main(String[] args) {
        Set<Bigram> s = new HashSet<>();
        for (int i = 0; i < 10; i++)
            for (char ch = 'a'; ch <= 'z'; ch++)
                s.add(new Bigram(ch, ch));
        System.out.println(s.size());
    }
}
```
## 5.8 使用标记接口定义类型
> 标记接口（marker interface），不包含方法声明，只是指定（或“标记”）一个类实现了具有某些属性的接口。 例如 Serializable 

# 6 Lambdas and Streams
## 6.1 lambda 表达式优于匿名类
> 在 Java 8 中，添加了函数式接口， lambda 表达式和方法引用，以便更容易地创建函数对象
> @FunctionalInterface

lambda 优于匿名类的主要优点是它更简洁
lambda 是表示小函数对象的最佳方式
```java
// Anonymous class instance as a function object - obsolete!
Collections.sort(words, new Comparator<String>() {
public int compare(String s1, String s2) {
return Integer.compare(s1.length(), s2.length());
}
});
// Lambda expression as function object (replaces anonymous class)
Collections.sort(words,
(s1, s2) -> Integer.compare(s1.length(), s2.length()));
```
**lambda 没有名称和文档; 如果计算不是自解释的，或者超过几行，则不要将其放入 lambda表达式中**
 一行代码对于 lambda 说是理想的，三行代码是合理的最大值。 如果违反这一规定，可能会严重损害程序的可读性
### 不适用lamdba
 Lambda 仅限于函数式接口

1. 如果你想创建一个抽象类的实例，你可以使用匿名类来实现，但不能使用 lambda
2. 匿名类来创建具有多个抽象方法的接口实例
3. lambda 不能获得对自身的引用；在 lambda 中， this 关键字引用封闭实例
## 6.2 方法引用优于 lambda 表达式
```java
Map<String, Integer> hashMap = new HashMap<>();
hashMap.merge("1",1,(count, incr) -> count + incr);
hashMap.merge("1",1,(count, incr) -> count + incr);
System.out.println(hashMap);
// 替换
map.merge(key, 1, Integer::sum);
```
## 6.3 优先使用标准的函数式接口
> 自定义@FunctionalInterface时优先考虑 java.util.function 包下已经定义的函数接口

java.util.function 下的六类基本接口
Operator 接口表示方法的结果和参数类型相同。
Predicate 接口表示其方法接受一个参数并返回一个布尔值。
Function 接口表示方法其参数和返回类型不同。
Supplier 接口表示一个不接受参数和返回值 (或“供应”) 的方法。
 Consumer 表示该方法接受一个参数而不返回任何东西，本质上就是使用它的参数。
![image.png](https://cdn.nlark.com/yuque/0/2022/png/22342229/1665545197598-a4ac7aa6-7ee0-4a5f-9472-76b6b029291b.png#clientId=u0c30c3fe-9225-4&from=paste&height=278&id=u4bad7de6&originHeight=278&originWidth=717&originalType=binary&ratio=1&rotation=0&showTitle=false&size=33949&status=done&style=none&taskId=uc814a41c-2b3f-4a29-825f-3faa140d5ce&title=&width=717)
## 6.4 明智审慎地使用 Stream
>  Java 8 中添加了 Stream API，以简化顺序或并行执行批量操作的任务

如果使用得当，流可以使程序更短更清晰；如果使用不当，它们会使程序难以阅读和维护。
## 6.5 优先考虑流中无副作用的函数
> 如果你是一个刚开始使用流的新手，那么很难掌握它们。仅仅将计算表示为流管道是很困难的。当你成功时，你的程序将运行，但对你来说可能没有意识到任何好处。流不仅仅是一个 API，它是基于函数式编程的范式(paradigm）。为了获得流提供的可表达性、速度和某些情况下的并行性，你必须采用范式和 API。


> 流范式中最重要的部分是将计算结构化为一系列转换，其中每个阶段的结果尽可能接近前一阶段结果的纯函数（pure function）。 纯函数的结果仅取决于其输入：它不依赖于任何可变状态，也不更新任何状态。 为了实现这一点，你传递给流操作的任何函数对象（中间操作和终结操作）都应该没有副作用

```java
// 伪流操作
Map<String, Long> freq = new HashMap<>();
try (Stream<String> words = new Scanner(file).tokens()) {
words.forEach(word -> {
freq.merge(word.toLowerCase(), 1L, Long::sum);
});
}
// 正常的流操作
Map<String, Long> freq;
try (Stream<String> words = new Scanner(file).tokens()) {
freq = words
.collect(groupingBy(String::toLowerCase, counting()));
}
```
**编程流管道的本质是无副作用的函数对象。 这适用于传递给流和相关对象的所有许多函数对象。 终结操作 forEach 仅应用于报告流执行的计算结果，而不是用于执行计算。 为了正确使用流，必须了解收集器。 最重要的收集器工厂是 toList ， toSet ， toMap ， groupingBy 和 join 。**
## 6.6  优先使用 Collection 而不是 Stream 来作为方法的返回类型
> 在 Java 8 之前，通常方法的返回类型是 Collection ， Set 和 List这些接口；还包括 Iterable 和数组类型

Stream 是一次性
## 6.7 谨慎使用流并行
Java 一直处于提供简化并发编程任务的工具的最前沿
Java 5 引入了 java.util.concurrent 类库，带有并发集合和执行器框架
Java 7 引入了 fork-join 包，这是一个用于并行分解的高性能框架
Java 8 引入了流，可以通过对parallel 方法的单个调用来并行化
> 通常，并行性带来的性能收益在 ArrayList 、 HashMap 、 HashSet 和 ConcurrentHashMap 实例、数
> 组、 int 类型范围和 long 类型的范围的流上最好。 这些数据结构的共同之处在于，它们都可以精确而廉价地分
> 割成任意大小的子程序，这使得在并行线程之间划分工作变得很容易

# 7 Methods
## 7.1  检查参数有效性
> 大多数方法和构造方法对可以将哪些值传递到其对应参数中有一些限制
> 例如：索引值必须是非负数，对象引用必须为非 null 。 
> 你应该清楚地在文档中记载所有这些限制，并在方法主体的开头用检查来强制执行。
>  应该尝试在错误发生后尽快检测到错误，这是一般原则的特殊情况。 
> 如果不这样做，则不太可能检测到错误，并且一旦检测到错误就更难确定错误的来源

每次编写方法或构造方法时，都应该考虑对其参数存在哪些限制。 应该记在这些限制，并在方法体
的开头使用显式检查来强制执行这些限制。 养成这样做的习惯很重要。 在第一次有效性检查失败时，它所需要的少
量工作将会得到对应的回报
## 7.2 必要时进行防御性拷贝
> 愉快使用 Java 的原因，它是一种安全的语言（safe language）。 这意味着在缺少本地方法（native methods）的情况下，它不受缓冲区溢出，数组溢出，野指针以及其他困扰 C 和 C++ 等不安全语言的内存损坏错误的影响。 在一种安全的语言中，无论系统的任何其他部分发生什么，都可以编写类并确切地知道它们的不变量会保持不变。 在将所有内存视为一个巨大数组的语言中，这是不可能的

**必须防御性地编写程序，假定类的客户端尽力摧毁类其不变量**
```java

// Broken "immutable" time period class
public final class Period {
    private final Date start;
    private final Date end;

    /**
     * @param start the beginning of the period
     * @param end   the end of the period; must not precede start
     * @throws IllegalArgumentException if start is after end
     * @throws NullPointerException     if start or end is null
     */
    public Period(Date start, Date end) {
        if (start.compareTo(end) > 0)
            throw new IllegalArgumentException(
                    start + " after " + end);
        this.start = start;
        this.end = end;
    }

    public Date start() {
        return start;
    }

    public Date end() {
        return end;
    }
   Date start = new Date();
   Date end = new Date();
   Period p = new Period(start, end);
   end.setYear(78); // Modifies internals of p




//修改后
// Repaired constructor - makes defensive copies of parameters
    public Period(Date start, Date end) {
        this.start = new Date(start.getTime());
        this.end = new Date(end.getTime());
        if (this.start.compareTo(this.end) > 0)
            throw new IllegalArgumentException(
                    this.start + " after " + this.end);
    }

    public Date start() {
        return new Date(start.getTime());
    }

    public Date end() {
        return new Date(end.getTime());
    }
}

```
## 7.3 仔细设计方法签名

1. **仔细选择方法名名称**  名称应始终遵守标准命名约定  选择与更广泛的共识一致的名称
2. **不要过分地提供方便的方法	**多的方法使得类难以学习、使用、文档化、测试和维护。对于接口更是如此，在接口中，太多的方法使实现者和用户的工作变得复杂
3. **避免过长的参数列表  **目标是四个或更少的参数 相同类型参数的长序列尤其有害  有三种技术可以缩短过长的参数列表
   1. 一种方法是将方法分解为多个方法
   2. 创建辅助类来保存参数组  这些辅助类通常是静态成员类
   3. 从对象构造到方法调用采用 Builder 模式 。如果你有一个方法有许多参数，特别是其中一些是可选的，那么可以定义一个对象来表示所有的参数，并允许客户端在这个对象上进行多个"setter" 调用，每次设置一个参数或较小相关的组。设置好所需的参数后，客户端调用对象的 "execute" 方法，该方法对参数进行最后的有效性检查，并执行实际的计算。
4. **对于参数类型，优先选择接口而不是类**    如果有一个合适的接口来定义一个参数，那么使用它来支持一个实现该接口的类 没有理由在编写方法时使用 HashMap 作为输入参数，相反，而是使用 Map 作为参数，这允许传入 HashMap、TreeMap、ConcurrentHashMap、TreeMap 的子 Map（submap）或任何尚未编写的 Map 实现。通过使用的类而不是接口，就把客户端限制在特定的实现中，如果输入数据碰巧以其他形式存在，则强制执行不必要的、代价高昂的复制操作
5. **与布尔型参数相比，优先使用两个元素枚举类型 **
## 7.4 明智审慎地使用重载
> 在编译时选择要调用哪个重载方法

**因为重载（overloaded）方法之间的选择是静态的，而重写（overridden）方法之间的选择是动态的**
```java

// Broken! - What does this program print?
public class CollectionClassifier {
    public static String classify(Set<?> s) {
        return "Set";
    }

    public static String classify(List<?> lst) {
        return "List";
    }

    public static String classify(Collection<?> c) {
        return "Unknown Collection";
    }

    public static void main(String[] args) {
        Collection<?>[] collections = {
                new HashSet<String>(),
                new ArrayList<BigInteger>(),
                new HashMap<String, String>().values()
        };
        for (Collection<?> c : collections)
            System.out.println(classify(c));
    }
}
```
**一个安全和保守的策略是永远不要导出两个具有相同参数数量的重载 或者采用不同名称（不进行重载）**

## 7.5 明智审慎地使用可变参数
> 可变参数方法正式名称称为可变的参数数量方法『variable arity methods』，接受零个或多个指定类型的参数。 可变参数机制首先创建一个数组，其大小是在调用位置传递的参数数量，然后将参数值放入数组中，最
> 后将数组传递给方法

在性能关键的情况下使用可变参数时要小心。每次调用可变参数方法都会导致数组分配和初始化
> 当需要使用可变数量的参数定义方法时，可变参数非常有用。 在使用可变参数前加上任何必需的参数，并注意使用可变参数的性能后果

## 7.6 返回空的数组或集合，不要返回 null
```java
 Collections.emptyList();
 List.of();
```
## 7.7 Return optionals judiciously
Optional 是java8为尽量避免数据空指针异常提供的 工具类。

1. 永远不要通过返回 Optional 的方法返回一个空值 它破坏 Optional 设计的初衷
2. 容器类型，包括集合、映射、Stream、数组和Optional，不应该封装在 Optional 中
3. 除了作为返回值之外，不应该在任何其他地方中使用 Optional
## 7.8 为所有已公开的 API 元素编写文档注释
> 文档注释是记录 API 的最佳、最有效的方法。对于所有导出的 API 元素，它们的使用应被视为必需的。 采用符合标准惯例的一致风格

# 8 General Programming(通用程序设计)
## 8.1 最小化局部变量的作用域
> 通过最小化局部变量的作用域，可以提高代码的可读性和可维护性，并降低出错的可能性
> 最小化局部变量作用域的最终技术是保持方法小而集中

## 8.2 for-each 循环优于传统 for 循环
```java
// Not the best way to iterate over a collection!
for (Iterator<Element> i = c.iterator(); i.hasNext(); ) {
Element e = i.next();
... // Do something with e
}
// Not the best way to iterate over an array!
for (int i = 0; i < a.length; i++) {
... // Do something with a[i]
}
```
> 但是它们并不完美。迭代器和索引变量都很混乱——你只需要元素而已。此外，它们也代表了出错的机会。迭代器在每个循环中出现三次，索引变量出现四次，这使你有很多机会使用错误的变量。

### 不能使用for-each 情况

- **有损过滤**   如果需要遍历集合，并删除指定选元素，则需要使用显式迭代器，以便可以调用其 remove 方法。 通常可以使用在 Java 8 中添加的 Collection 类中的 removeIf 方法，来避免显式遍历
- **转换  **如果需要遍历一个列表或数组并替换其元素的部分或全部值
- **并行迭代 ** 如果需要并行地遍历多个集合，那么需要显式地控制迭代器或索引变量
## 8.3 了解并使用库
java类库是由专家设计编写并经过测试的

1. 通过使用标准库，你可以利用编写它的专家的知识和以前使用它的人的经验
2. 你不必浪费时间为那些与你的工作无关的问题编写专门的解决方案
3. 使用标准库的第三个优点是，随着时间的推移，它们的性能会不断提高，而你无需付出任何努力
4. 这样的代码更容易被开发人员阅读、维护和重用
> 总而言之，不要白费力气重新发明轮子。如果你需要做一些看起来相当常见的事情，那么库中可能已经有一个工
> 具可以做你想做的事情

## 8.4 若需要精确数值就应避免使用 float 和 double 类型
[浮点数精度为什么会丢失(Java示例)](http://abingsky37.github.io/float_lost.html)
float 和double 类型特别不适合进行货币计算，因为不可能将 0.1（或 10 的任意负次幂）精确地表示为 float 或 double
正确方法是 使用 BigDecimal、int 或 long 进行货币计算
```java
System.out.println(1.00 - 9 * 0.10);
```
## 8.5  基本数据类型优于包装类

1. 基本类型只有它们的值，而包装类型具有与其值不同的标识
2. 基本类型只有全功能值，而每个包装类型除了对应的基本类型的所有功能值外，还有一个非功能值，即 null。
3. 基本类型比包装类型更节省时间和空间
4. 程序执行拆箱时，将抛出 NullPointerException
## 8.6 当使用其他类型更合适时应避免使用字符串
> 当存在或可以编写更好的数据类型时，应避免将字符串用来表示对象。如果使用不当，字符串比其他类型
> 更麻烦、灵活性更差、速度更慢、更容易出错。字符串经常被误用的类型包括基本类型、枚举和聚合类型

## 8.7 当心字符串连接引起的性能问题
> 字符串连接操作符 (+) 是将几个字符串组合成一个字符串的简便方法。对于生成单行输出或构造一个小的、固
> 定大小的对象的字符串表示形式，它是可以的，但是它不能伸缩。使用 字符串串联运算符重复串联 n 个字符串需要
> n 的平方级时间

要获得能接受的性能，请使用 StringBuilder 代替 String+
## 8.8 通过接口引用对象
> 如果存在合适的接口类型，那么应该使用接口类型声明参数、返回值、变量和字段
> 如果你养成了使用接口作为类型的习惯，那么你的程序将更加灵活

```java
// Good - uses interface as type
Set<Son> sonSet = new LinkedHashSet<>();
// Bad - uses class as type!
LinkedHashSet<Son> sonSet = new LinkedHashSet<>();
//换
Set<Son> sonSet = new HashSet<>();
```
**注意：**如果原实现提供了接口的通用约定不需要的一些**特殊功能**，并且代码依赖于该功能，那么新实现提供相同的功能就非常重要。
例如，如果围绕第一个声明的代码依赖于** LinkedHashSet 的排序策略**，那么在声明中
将 HashSet 替换为 LinkedHashSet 将是不正确的，因为 HashSet 不保证迭代顺序
## 8.9  接口优于反射(Prefer interfaces to reflection)
**使用反射时尽可能在编译时已知接口或者超类**
反射的弊端

1. 你失去了编译时类型检查的所有好处
2. 执行反射访问所需的代码既笨拙又冗长
3. 性能降低
> 通过非常有限的形式使用反射，你可以获得反射的许多好处，同时花费的代价很少。 对于许多程序，它们必须用到在编译时无法获取的类，在编译时存在一个适当的接口或超类来引用该类）。如果是这种情况，
> 可以用反射方式创建实例，并通过它们的接口或超类正常地访问它们

```java
public static void main(String[] args) {
        test("java.util.HashSet");
        test("java.util.TreeSet");
    }

    private static void test(String name) {
        Class<? extends Set<String>> cla = null;
        try {
            cla = (Class<? extends Set<String>>) Class.forName(name);
            Constructor<? extends Set<String>> constructor = cla.getConstructor();
            Set<String> set = constructor.newInstance();
            set.add("");
        } catch (Exception e) {

        }
    }
```
> 总之，反射是一种功能强大的工具，对于某些复杂的系统编程任务是必需的，但是它有很多缺点。如果编写的程
> 序必须在编译时处理未知的类，则应该尽可能只使用反射实例化对象，并使用在编译时已知的接口或超类访问对象

## 8.10 明智审慎地本地方法
> Java 本地接口（JNI）允许 Java 程序调用本地方法，这些方法是用 C 或 C++ 等本地编程语言编写的

1. 为了提高性能，很少建议使用本地方法
2. 安全问题
## 8.11 明智审慎地进行优化
格言
> 不要去计较效率上的一些小小的得失，在 97% 的情况下，不成熟的优化才是一切问题的根源。
> —Donald E. Knuth 

> 在优化方面，我们应该遵守两条规则：
> 规则 1：不要进行优化。
> 规则 2 （仅针对专家）：还是不要进行优化，也就是说，在你还没有绝对清晰的未优化方案之前，请不要进行优化。
> —M. A. Jackson 

> More computing sins are committed in the name of efficiency(without necessarily achieving it) than for any other singlereason—including blind stupidity.
> —William A. Wulf 

**不要过早进行优化**
**解耦**
> 不要为了性能而牺牲合理的架构。努力编写 好的程序，而不是快速的程序。 如果一个好的程序不够快，它的架构将允许它被优化。好的程序体现了信息隐藏的原则：在可能的情况下，它们在单个组件中本地化设计决策，因此可
> 以在不影响系统其余部分的情况下更改单个决策.


1. **尽量避免限制性能的设计决策  **，组件设计中最困难的是与外部有关联或者交互的组件  。这些组件主要是在设计API、线路层协议和持久数据格式。而且所有这些组件都可能对系统能够达到的性能造成重大限制
2. **API设计影响性能 **   在 API 中使用实现类而不是接口将你绑定到特定的实现会影响后续优化。
## 8.12 遵守被广泛认可的命名约定

1. 包名和模块名应该是分层的，组件之间用句点分隔
2. 组件应该由小写字母组成，很少使用数字
3. 类和接口名称，包括枚举和注释类型名称，应该由一个或多个单词组成，每个单词的首字母大写
4. 方法和字段名遵循与类和接口名相同的排版约定 方法或字段名的第一个字母应该是小写
5. 常量字段，它的名称应该由一个或多个大写单词组成，由下划线分隔
6. 局部变量名与成员名具有类似命名约定，但允许使用缩写，也允许使用单个字符和短字符序列，它们的含义取决于它们出现的上下文
7. 泛型参数 T 表示任意类型，E 表示集合的元素类型，K和 V 表示 Map 的键和值类型，X 表示异常。函数的返回类型通常为 R。任意类型的序列可以是 T、U、V 或 T1、T2、T3

![image.png](https://cdn.nlark.com/yuque/0/2022/png/22342229/1665670643099-8a722054-0e2b-4779-8e2d-8545b5a7f814.png#clientId=u367e17a8-c53d-4&from=paste&height=257&id=u6f066fb2&originHeight=257&originWidth=700&originalType=binary&ratio=1&rotation=0&showTitle=false&size=30489&status=done&style=none&taskId=u5e06421d-cdd2-4166-9946-ab206dac67f&title=&width=700)
# 9 异常
> 当使用最佳优势时，异常能够提高程序的 可读性 可靠性、可维护性

## 9.1 只针对异常的情况下才使用异常
> 异常应该只用于异常的情况下；他们永远不应该用于正常的程序控制流程

异常

- 因为异常设计的初衷适用于不正常的情形，所有几乎没有 JVM 实现试图对他们进行优化，使它们与显式的测试一样快
- 把代码放在 try-catch 块中反而阻止了现代 JVM 实现本可能执行的某些特定优化
## 9.2 对可恢复的情况使用受检异常，对编程错误使用运行时异常
> java 程序设计语言提供了三种 throwable：
> 受检异常（checked exceptions）
> 运行时异常（runtime exceptions）
> 和错误（errors）

[Chapter 11. Exceptions](https://docs.oracle.com/javase/specs/jls/se7/html/jls-11.html)
**在决定使用受检异常还是非受检异常时，主要的原则是： 如果期望调用者能够合理的恢复程序运行，对于这种情况就应该使用受检异常**
![image.png](https://cdn.nlark.com/yuque/0/2022/png/22342229/1665736169829-4ee30ea3-e8b6-41bf-9f22-58b510a75ed7.png#clientId=u1aa94c71-4800-4&from=paste&height=327&id=uecb3f435&originHeight=327&originWidth=850&originalType=binary&ratio=1&rotation=0&showTitle=false&size=20605&status=done&style=none&taskId=uce5452c5-4476-447f-93fe-45bb2a2d3d4&title=&width=850)
## 9.3 避免不必要的使用受检异常
> 过分使用受检异常会使 API 使用起来非常不方便。

> 在谨慎使用的前提之下，受检异常可以提升程序的可读性；
> 如果过度使用，将会使 API 使用起来非常痛苦。如果调用者无法恢复失败，就应该抛出未受检异常。
> 如果可以恢复，并且想要迫使调用者处理异常的条件，首选应该返回一个 optional 值。当且仅当万一失败时，这些无法提供足够的信息，才应该抛出受检异常

## 9.4 优先使用标准的异常
> Java 平台类库提供了一组基本的未受检异常，它们满足了绝大多数 API 的异常抛出需求

![image.png](https://cdn.nlark.com/yuque/0/2022/png/22342229/1665738342689-affa52cd-de54-4996-8fdd-f0dab348d9b5.png#clientId=u1aa94c71-4800-4&from=paste&height=288&id=u9299e418&originHeight=288&originWidth=694&originalType=binary&ratio=1&rotation=0&showTitle=false&size=46821&status=done&style=none&taskId=udc85bdac-ee85-474d-9bf6-d5a944f7855&title=&width=694)
## 9.5 抛出与抽象对应的异常
> 如果方法抛出的异常与它所执行的任务没有明显的联系，这种情形将会使人不知所措。当方法传递由低层抽象抛
> 出的异常时，往往会发生这种情况.

更高层的实现应该捕获低层的异常，同时抛出可以按照高层抽象进行解释的异常。这种做法称为异常转译 (exception translation)
```java
    /* Exception Translation */
try {
     ...
            /* Use lower-level abstraction to do our bidding */
        } catch (LowerLevelException e) {
            throw new HigherLevelException(...);
        }

public E get( int index ) {
ListIterator<E> i = listIterator( index );
try {
	return(i.next() );
}
catch ( NoSuchElementException e ) 
{
	throw new IndexOutOfBoundsException( "Index: " + index );
}
}
```
## 9.6 每个方法抛出的异常都需要创建文档
> 描述一个方法所抛出的异常，是正确使用这个方法时所需文档的重要组成部分。因此，花点时间仔细地为每个方
> 法抛出的异常建立文档是特别重要的

单独地声明受检异常  准确地记录下抛出每个异常的条件
## 9.7 在细节消息中包含失败一捕获信息

1. 为了捕获失败，异常的细节信息应该包含“对该异常定位有帮助”的所有参数和字段的值。
2. 千万不要在细节消息中包含密码、密钥以及类似的信息
## 9.8 保持失败原子性
> 一般而言，失败的方法调用应该使对象保持在被调用之前的状态。 具有这种属性的方法被称为具有失败原子性 (failure atomic) 

有几种途径可以实现这种效果

1. 最简单的办法莫过于设计一个不可变的对象 
2. 对于在可变对象上执行操作的方法，获得失败原子性最常见的办法是，在执行操作之前检查参数的有效性   这可以使得在对象的状态被修改之前，先抛出适当的异常
   1. 一种类似的获得失败原子性的办法是，调整计算处理过程的顺序，使得任何可能会失败的计算部分都在对象状态被修改之前发生
3. 第三种获得失败原子性的办法是，在对象的一份临时拷贝上执行操作，当操作完成之后再用临时拷贝中的结果代替对象的内容
## 9.9 不要忽略异常
```java
// Empty catch block ignores exception - Highly suspect!
try {
...
} catch ( SomeException e ) {
}
```
# 10 并发
## 10.1  同步访问共享的可变数据
> 关键字 synchronized 可以保证在同一时刻，只有一个线程可以执行某一个方法，或者某一个代码块.

> 当一个对象被一个线程修改的时候，可以阻止另一个线程观察到对象内部不一致的状态


**为了在线程之间进行可靠的通信，也为了互斥访问，同步是必要的  **这归因于 Java 语言规范中的内存模型（memory model），它规定了一个线程所做的变化何时以及如何变成对其他线程可见。
>  当多个线程共享可变数据的时候，每个读或者写数据的线程都必须执行同步

## 10.2 避免过度同步
> 过度同步则可能导致性能降低、死锁，甚至不确定的行为

通常来说，应该在同步区域内做尽可能少的工作。
## 10.3 executor 、task 和 stream 优先于线程
当直接使用线程时， Thread 是既充当工作单元，又是执行机制。在 Executor Framework 中，工作单元和执行机制是分开的
## 10.4 相比 wait 和 notify 优先使用并发工具
ava.util.concurrent 中更高级的工具分成三类：
 Executor Framework 
并发集合（ Concurrent Collection ）
同步器（ Synchronizer）

- CountDownLatch
- Semaphore
- CyclicBarrier
## 10.5 文档应包含线程安全属性
> 每个类都应该措辞严谨的描述或使用线程安全注解清楚地记录其线程安全属性

## 10.6  明智审慎的使用延迟初始化（懒加载）
延迟初始化是延迟字段的初始化，直到需要它的值。如果不需要该值，则不会初始化字段
> 延迟初始化是一把双刃剑。它降低了初始化类或创建实例的成本，代价是增加了访问延迟初始化字段的成本

**延迟初始化也有它的用途。如果一个字段只在类的一小部分实例上访问，并且初始化该字段的代价很高，那么延**
**迟初始化可能是值得的。唯一确定的方法是以使用和不使用延迟初始化的效果对比来度量类的性能**

## 10.7 不要依赖线程调度器
当许多线程可以运行时，线程调度器决定哪些线程可以运行以及运行多长时间。任何合理的操作系统都会尝试**公**
**平地做出这个决定**，但是策略可能会有所不同。因此，编写良好的程序不应该依赖于此策略的细节。任何依赖线程调
度器来保证正确性或性能的程序都可能是不可移植的
Thread.yield 
Thread.stop

# 11 序列化
## 11.1 优先选择 Java 序列化的替代方案
> 序列化的一个根本问题是它的可攻击范围太大，且难以保护，而且问题还在不断增多：通过调用

> ObjectInputStream 上的 readObject 方法反序列化对象图。**这个方法本质上是一个构造函数**，可以用来实例化
> 类路径上几乎任何类型的对象，只要该类型实现 Serializable 接口。在反序列化字节流的过程中，此方法可以执行来
> 自任何这些类型的代码，因此所有这些类型的代码都在攻击范围内


> Java 反序列化是一个明显且真实的危险源，因为它被应用程序直接和间接地广泛使用，比如 RMI（远程方法调
> 用）、JMX（Java 管理扩展）和 JMS（Java 消息传递系统）。不可信流的反序列化可能导致远程代码执行
> （RCE）、拒绝服务（DoS）和一系列其他攻击。应用程序很容易受到这些攻击，即使它们本身没有错误

领先的跨平台结构化数据表示是 JSON 和 Protocol Buffers，也称为 protobuf。JSON 由 Douglas Crockford 设计用
于浏览器与服务器通信，Protocol Buffers 由谷歌设计用于在其服务器之间存储和交换结构化数据。尽管这些技术有时
被称为「中性语言」，但 JSON 最初是为 JavaScript 开发的，而 protobuf 是为 c++ 开发的；这两种技术都保留了其起
源的痕迹
## 11.2 非常谨慎地实现 Serializable
> 使类的实例可序列化非常简单，只需实现 Serializable 接口即可

**实现 Serializable 接口的一个主要代价是，一旦类的实现被发布，它就会降低更改该类实现的灵活性**
字段变更后兼容性
**实现 Serializable 接口的第二个代价是，增加了出现 bug 和安全漏洞的可能性**
**实现 Serializable 接口的第三个代价是，它增加了与发布类的新版本相关的测试负担**
所需的测试量与可序列化类的数量及版本的数量成正比，工作量可能很大。你必须确保「序列化-反序列化」过程成功，并确保
它生成原始对象的无差错副本
## 11.3 考虑使用自定义的序列化形式

- **transient**
- void writeObject(ObjectOutputStream out) throws IOException
- void readObject(ObjectInputStream in)throws IOException，ClassNotFoundException
```java
private void writeObject(ObjectOutputStream out) throws IOException {  
        //invoke default serialization method
        out.defaultWriteObject(); 
 
        if(school == null)
            school = new School();
        out.writeObject(school.sName);  
        out.writeObject(school.sId);  
    }  
 
private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {  
        //invoke default serialization method
        in.defaultReadObject();
  
        String name = (String) in.readObject();  
        String id = (String) in.readObject(); 
        school = new School(name, id);
    }  
```
## 11.4  保护性的编写 readObject 方法
## 11.5 对于实例控制，枚举类型优于 readResolve
```java
// readResolve for instance control - you can do better!
private Object readResolve() {
// Return the one true Elvis and let the garbage collector
// take care of the Elvis impersonator.
return INSTANCE;
}
```
## 11.6  考虑用序列化代理代替序列化实例
