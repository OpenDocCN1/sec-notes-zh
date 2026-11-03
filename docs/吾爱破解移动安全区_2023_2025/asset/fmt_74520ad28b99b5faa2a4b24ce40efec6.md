
## 静态代码块

```
package com.java.day2_static_code;

public class StaticCode {
    // 静态代码块：如果在启动系统时对静态资源进行初始化，则建议使用静态代码块完成数据的初始化作用
    public static String username;

    static {
        System.out.println("静态代码块对静态资源进行初始化啦！");
        username = "张三";
    }

    public static void main(String[] args) {
        System.out.println("main方法开始执行了！");
    }
}
```

## 抽象类

抽象类是不能被实例化的类，如果你是新手，那你是不是很疑惑，一个类不能被实例化那还有什么卵用？

别急嘛！抽象类的存在主要是为了被子类继承和实现。抽象类中可以包含抽象方法，抽象方法是一种没有实现的方法，只有方法的声明，没有方法体。

抽象类的特点如下：

- 抽象类不能被实例化，只能被继承。
- 抽象类可以包含抽象方法和非抽象方法。
- 抽象方法必须被子类通过方法重写实现，否则子类也必须声明为抽象类。
- 抽象类可以拥有构造方法，但抽象类不能被用来创建对象。

下面是一个抽象类的例子：

```
public abstract class Animal {  // abstract表示该类是抽象类

    private String name;

    public Animal(String name) {
        this.name = name;
    }

    public abstract void move();

    public void eat() {
        System.out.println(name + "在吃东西");
    }

    public String getName() {
        return name;
    }

}

public class Dog extends Animal {

    public Dog(String name) {
        super(name);
    }

    @Override
    public void move() {
        System.out.println(super.getName() + "在奔跑");
    }

}

public class Test {

    public static void main(String[] args) {
        Animal animal = new Dog("小狗");
        animal.move();
        animal.eat();
    }

}
```

在上面的例子中，`Animal` 类是一个抽象类，它包含了一个抽象方法 `move()`、一个非抽象方法 `eat()` 和一个非抽象方法 `getName()`。在 `Dog` 类中继承了 `Animal` 类，并实现了 `move()` 方法。在 `Test` 类中，我们创建了一个 `Animal` 类型的对象，实际上是一个 `Dog` 对象，然后调用了它的 `move()`、`eat()` 和 `getName()` 方法。由于 `Dog` 类重写了 `move()` 方法，因此输出的是“小狗在奔跑”，而 `eat()` 方法是从父类继承而来，输出的是“小狗在吃东西”，`getName()` 方法也是从父类继承而来，用来获取动物的名称。

## 接口

接口是一种特殊的抽象类，它只包含了抽象方法和常量，没有任何实现。接口中的所有方法都是公共的，不能包含实例域或构造器，因此不能被实例化。

一个类可以实现多个接口，但只能继承一个类。

接口的定义格式如下：

```
public interface 接口名 {
    // 常量定义
    // 方法声明
}
```

接口中的方法默认为 `public abstract` 类型，可以省略这两个关键字。接口中的常量必须是 `public static final` 类型的(这三个修饰符表示该变量是公共的、静态的、不可改变的，也叫该变量为常量)，可以省略这三个关键字。接口中的方法不能包含方法体，必须由实现类去实现。

下面是一个接口的例子：

```
public interface Animal {

    int LEGS = 4;

    void move();

}

public class Dog implements Animal {

    @Override
    public void move() {
        System.out.println("狗在奔跑");
    }

}

public class Test {

    public static void main(String[] args) {
        Animal animal = new Dog();
        animal.move();
        System.out.println("狗有" + Animal.LEGS + "条腿");
    }

}
```

在上面的例子中，`Animal` 是一个接口，包含了一个常量 `LEGS` 和一个抽象方法 `move()`。`Dog` 类实现了 `Animal` 接口，并实现了 `move()` 方法。在 `Test` 类中，我们创建了一个 `Animal` 类型的对象，实际上是一个 `Dog` 对象，然后调用了它的 `move()` 方法。由于 `Dog` 类实现了 `Animal` 接口，因此可以使用 `Animal` 类型来引用 `Dog` 对象，从而实现了多态性。输出的结果为“狗在奔跑，狗有4条腿”。

## 内部类

内部类其实就是在一个类里面再定义一个类，内部类可以访问其外部类的所有成员，包括私有成员。内部类可以分为成员内部类、静态内部类、局部内部类和匿名内部类。

### 成员内部类

成员内部类就是在一个类的内部定义的另一个类，它可以访问外部类的所有成员，包括私有成员，并且可以使用外部类的引用来访问外部类的成员。成员内部类的定义格式如下：

```
public class Outer {
    private int x = 10;
    public class Inner {
        public void print() {
            System.out.println("x = " + x);
        }
    }
}
```

在上面的例子中，`Inner` 是 `Outer` 的成员内部类，它可以访问 `Outer` 的私有成员 `x`。在外部类中创建 `Inner` 对象的方法如下：

```
Outer outer = new Outer();
Outer.Inner inner = outer.new Inner();
inner.print();
```

### 静态内部类

静态内部类是在一个类的内部定义的静态类，它和普通的类一样，不依赖于外部类的实例，因此可以直接通过类名来访问。静态内部类不能访问外部类的非静态成员，只能访问外部类的静态成员。静态内部类的定义格式如下：

```
public class Outer {
    private static int x = 10;
    public static class Inner {
        public void print() {
            System.out.println("x = " + x);
        }
    }
}
```

在上面的例子中，`Inner` 是 `Outer` 的静态内部类，它可以访问 `Outer` 的静态成员 `x`。在外部类中创建 `Inner` 对象的方法如下：

```
Outer.Inner inner = new Outer.Inner();
inner.print();
```

### 局部内部类

局部内部类是定义在方法内部的类，它只能在该方法内部使用，对外部不可见。局部内部类可以访问外部类的所有成员，包括私有成员。局部内部类的定义格式如下：

```
public class Outer {
    private int x = 10;
    public void test() {
        class Inner {
            public void print() {
                System.out.println("x = " + x);
            }
        }
        Inner inner = new Inner();
        inner.print();
    }
}
```

在上面的例子中，`Inner` 是 `Outer` 方法内部的局部内部类，它可以访问 `Outer` 的私有成员 `x`。在方法内部创建 `Inner` 对象的方法如下：

```
Outer outer = new Outer();
outer.test();
```

### 匿名内部类

匿名内部类是没有名字的内部类，它通常用于创建一个只需要使用一次的类。匿名内部类可以继承一个类或者实现一个接口，它没有构造方法，但可以使用构造代码块进行初始化。匿名内部类的定义格式如下：

```
new 父类构造器/接口() {
    // 匿名内部类的内容
}
```

在上面的例子中，父类构造器可以是有参数的构造器，也可以是无参数的构造器；接口可以是有多个方法的接口，也可以是只有一个方法的接口。下面是一个使用匿名内部类实现接口的例子：

```
public interface Animal {

    void move();

}

public class Test {

    public static void main(String[] args) {
        Animal animal = new Animal() {
            @Override
            public void move() {
                System.out.println("动物在移动");
            }
        };
        animal.move();
    }

}
```

在上面的例子中，我们创建了一个实现 `Animal` 接口的匿名内部类，并实现了 `move()` 方法。在 `main()` 方法中，我们创建了一个 `Animal` 类型的对象，实际上是一个匿名内部类的对象，然后调用了它的 `move()` 方法。输出的结果为“动物在移动”。

## 异常

Java中的异常指程序在运行过程中出现的错误或异常情况，例如类型错误、数组下标越界等。Java提供了异常处理机制来处理这些异常情况。

### 异常的体系

- JAVA设计的所有异常类都是有一个祖宗的，这个祖宗类叫 `Java.lang.Throwable`，代表这些异常对象都是可以向外抛出去的。
- 这个祖宗类有两个子类分支，分别是 `Error` 和 `Exception`。
- `Error`：代表系统级别的错误，属于严重的错误，比如系统出现问题，甲骨文公司就会把这些问题封装成 `Error` 对象给抛出来。所以说这个不是给开发者用的。
- `Exception`：这个才是我们熟知的异常，才是代表开发者可能会出现的问题，开发者一般都是使用该类以及其子类来封装程序出现的问题。
- `Exception` 的子类有两类异常，分别是运行时异常和编译时异常。
- 运行时异常：`RuntimeException` 类以及其子类，编译时(写代码的时候)不会出现错误提醒，运行时才会抛出异常。
- 编译时异常：写代码时会出现错误提醒。

### 自定义异常并简单处理异常

Java能提供的异常类是有限的，不可能为全世界出现的问题提供异常类来代表出现的问题。如果某个企业出现自己的问题，想要通过异常类来表示，那就需要自己来定义异常类了。接下来我们自己自定义异常类，假设我们需要收集每个用户合理的年龄，如果检测到某个用户输入的年龄不合理就抛出我们自定义的异常。

自定义运行时异常类：

```
package com.java.day5_exception;

// 1、自定义运行时异常类必须让这个类继承RuntimeException，这样才可以成为一个运行时异常类
// 2、需要重写构造器，当我们创建异常对象时可能需要将参数传给有参构造器进行初始化
public class AgeUnreasonableRuntimeException extends RuntimeException{
    public AgeUnreasonableRuntimeException() {
    }

    public AgeUnreasonableRuntimeException(String message) {  // message参数是用来封装我们自定义异常出现的原因
        // 将封装我们自定义异常出现的原因的参数传给父类的有参构造器，最后由父类来封装我们自定义异常出现的原因
        super(message);
    }
}
```

自定义编译时异常类：

```
package com.java.day5_exception;

// 1、自定义编译时异常类必须让这个类继承Exception，这样才可以成为一个编译时异常类
// 2、需要重写构造器，当我们创建异常对象时可能需要将参数传给有参构造器进行初始化
public class AgeUnreasonableException extends Exception{
    public AgeUnreasonableException() {
    }

    public AgeUnreasonableException(String message) {  // message参数是用来封装我们自定义异常出现的原因
        // 将封装我们自定义异常出现的原因的参数传给父类的有参构造器，最后由父类来封装我们自定义异常出现的原因
        super(message);
    }
}
```

创建一个Java类，用于实现收集每个用户合理的年龄，如果检测到某个用户输入的年龄不合理就抛出我们自定义的异常：
