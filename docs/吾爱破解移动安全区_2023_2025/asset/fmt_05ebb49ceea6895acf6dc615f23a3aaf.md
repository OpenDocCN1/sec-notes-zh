
```java
package com.java.method;

public class Methods {
    public static void main(String[] args) {
        /* 方法格式：
        *修饰符 返回值类型 方法名(形参列表){
        *   方法中要执行的代码
        *   return 要返回的值;  // 方法声明了具体的返回值类型，内部必须使用return返回对应的数据类型的数据
        * }
        * 注意：基本类型数据传入到方法中是传入的值，修改传入到方法的值是不会影响到全局变量；
        * 而引用类型数据传入到方法中的是地址，修改方法内的值是会改变全局变量的
        * */
        int s = sum(100,200);
        System.out.println(s);
    }
    public  static int sum(int a,int b) {
        return a+b;
    }
}
```

## 面向对象

面向对象是`Java`的重中之重，之间简单介绍了对象和类的关系，现在正式来讲解一下面向对象相关知识。

### 构造器

每个类都有构造器。如果没有主动的为类定义构造器，`Java`编译器将会为该类提供一个默认的无参构造器。构造器的主要作用是初始化对象的数据成员，在创建一个对象时，构造器会被自动调用来为对象的各个数据成员赋初值。

面向对象基础实例：

#### ClassMethod.java 文件

```java
package com.java.ClassObject;

/*

 * JAVA类声明中关键字public起到什么作用呢？如下ClassMethod类的声明:

 * 按着字面的意思理解是：ClassMethod类是公共的，要求ClassMethod类与类文件名必须保持一致，并且一个java文件中只能有一个public关键字声明的类。

 * public是表示公有的，private是表示私有的，私有的只能在面向对象中访问。

 * */

/*

* Java面向对象底层原理：

* Java面向对象底层是由三个区域组成，分别是方法区、栈内存、堆内存，执行流程大概如下所示：

* 1、将类加载到方法区

* 2、将类中的main方法压入栈内存当中

* 3、执行main方法中的代码，如果创建对象又会将对象对应的类加载到方法区，并且将对象的相对偏移地址压入栈内存中存储，而对象在堆内存中存储。

* 4、堆内存中的对象不仅可以存储属性数据，还可以存储方法（即对象的方法），但方法是以引用地址的形式存储在堆内存当中，方法代码需要通过引用地址去方法区调用。

* 5、方法区中存储了类的结构信息，包括类的属性、方法、常量池等。

* */

public class ClassMethod {

    public static void main(String[] args) {

        // 创建对象：类名 对象名 = new 类名();

        Zoo z = new Zoo("市中心动物园", 10);

        z.n = 1;

        z.main("长颈鹿", 11.4, 0.8);

    }

}
```

#### Zoo.java 文件

```java
package com.java.ClassObject;

public class Zoo {
    public String name;
    public int num;

    // 定义无参数构造器，构造器就是对面向对象属性进行初始化
    public Zoo() {
        System.out.println("这是无参数构造器！");
    }

    // 定义有参数构造器
    public Zoo(String name, int num) {
        this.name = name;  // this关键字代表用于存储当前对象的存储地址
        this.num = num;
        System.out.println("动物园叫" + name + "，动物园有" + num + "动物园区");
        System.out.println("这是有参数构造器！");
    }

    // 定义/初始化变量：修饰符 数据类型 变量名称 = 初始化值;
    public int n;
    public static void dong_wu(String name,double h,double w) {
        System.out.println("在动物园看到了" + name + ",它身高有" + h + "米，它宽度有" + w + "米");
    }
    public void main(String z, double h, double w) {
        for (;n!=0;n--) {
            dong_wu(z, h, w);
        }
    }
}
```

### 面向对象三大特性之封装

封装指的是将数据和操作数据的方法封装在一个单元内部，并通过访问权限控制来限制外部对该单元的访问。

例如，下面是一个简单的`Java`类，它封装了一个学生的姓名和年龄：

```java
public class Student {

    private String name;

    private int age;

    public String getName() {

        return name;

    }

    public void setName(String name) {

        this.name = name;

    }

    public int getAge() {

        return age;

    }

    public void setAge(int age) {

        this.age = age;

    }

}
```

可以看到在这个类中，姓名和年龄都是私有成员变量，只能通过公开的`getter`和`setter`方法来访问和修改。这样，我们就可以在外部控制和限制对这些成员变量的访问。

### 面向对象三大特性之继承

继承就是子类继承父类的特征和行为，使得子类对象（实例）具有父类的实例域和方法，或子类从父类继承方法，使得子类具有父类相同的行为。

`Java`中的继承是单继承的，也就是说每个子类只能继承一个父类。`Java`中通过使用关键字`"extends"`来实现继承。

继承的语法格式如下所示：

```java
class SubClass extends SuperClass {
    // SubClass 类的属性和方法
}
```

其中，`SubClass`是子类，`SuperClass`是父类。子类可以继承父类的属性和方法，同时可以添加自己的属性和方法。

`Java`中的所有类都继承自`Object`类。因此，如果没有指定父类，`Java`中的类默认继承自`Object`类。

`Java`中的继承关系可以形成继承层次结构，也就是说一个子类可以成为另一个子类的父类。`Java`中的继承层次结构可以使用继承关系图来表示。

在子类继承父类后，构造器会有以下特点：

- 子类中所有的构造器默认都会先访问父类中无参的构造器，再执行自己。这是因为子类在初始化的时候，有可能会使用到父类中的数据，如果父类没有完成初始化，子类将无法使用父类的数据。
- 如果父类中没有无参的构造器，子类必须使用`super`关键字显式调用父类的构造器，并传入相应的参数。例如：

```java
public class SubClass extends SuperClass {

  public SubClass(int x, int y) {

     super(x); // 调用父类的有参构造器

     // 子类的构造器逻辑

  }

}
```

- 子类通过`this(...)`去调用本类的其他构造器，本类其他构造器会通过`super`去手动调用父类的构造器，最终还是会调用父类构造器的。例如：

```java
public class SubClass extends SuperClass {
  public SubClass(int x) {
     this(x, 0); // 调用本类的有参构造器
  }

  public SubClass(int x, int y) {
     super(x); // 调用父类的有参构造器
     // 子类的构造器逻辑
  }
}
```

- 注意：`this(...)`和`super(...)`都只能放在构造器的第一行，所以二者不能共存在同一个构造器中。

继承的优点在于它可以提高代码的重用性和可维护性，同时可以使代码更加灵活和可扩展。通过继承，子类可以继承父类的属性和方法，从而减少了代码的编写量。此外，继承也可以使代码更加灵活，能够更好地适应需求的变化。

继承的缺点在于它可能会引入过多的复杂性，使程序难以维护和扩展。如果继承关系设计不当，会导致代码的耦合性过高，增加了代码的复杂度和维护成本。因此，在设计继承关系时，需要仔细考虑继承的层次，避免出现过多的继承关系。

### 面向对象三大特性之多态

在 `Java` 中，多态是一种基于继承、多态和重载的特性。它允许同一种类型的对象在不同的场景下表现出不同的行为。

`Java` 实现多态的方式主要有两种：继承和接口。

- 继承实现多态

子类可以继承父类的方法，并且可以重写父类的方法，从而实现多态。当子类重写了父类的方法后，当通过父类对象调用该方法时，实际上会调用子类重写后的方法。

示例代码：

```java
class Animal {

    public void makeSound() {

        System.out.println("未知的叫声");

    }

}

class Dog extends Animal {

    public void makeSound() {

        System.out.println("汪汪汪");

    }

}

class Cat extends Animal {

    public void makeSound() {

        System.out.println("喵喵喵");

    }

}

public class Test {

    public static void main(String[] args) {

        Animal animal = new Animal();

        Animal dog = new Dog();

        Animal cat = new Cat();

        animal.makeSound();

        dog.makeSound();

        cat.makeSound();

    }

}
```

输出结果：

```
未知的叫声
汪汪汪
喵喵喵
```

- 接口实现多态

接口是一种规范，它定义了一组方法，但并不提供方法的具体实现。不同的类可以实现同一个接口，并且每个类都可以根据自己的实际情况来实现接口中的方法，从而实现多态。

示例代码：

```java
interface Animal {

    void makeSound();

}

class Dog implements Animal {

    public void makeSound() {

        System.out.println("汪汪汪");

    }

}

class Cat implements Animal {

    public void makeSound() {

        System.out.println("喵喵喵");

    }

}

public class Test {

    public static void main(String[] args) {

        Animal dog = new Dog();

        Animal cat = new Cat();

        dog.makeSound();

        cat.makeSound();

    }

}
```

输出结果：

```
汪汪汪
喵喵喵
```

无论是通过继承还是接口实现多态，都可以使代码更加灵活、可扩展性更强。

### 虚方法

在`Java`中，虚方法的实现依赖于方法表和虚方法表。方法表是每个类的一部分，它包含了该类所有的方法的信息，包括方法名、参数类型、返回值类型等。虚方法表是每个类的一个隐藏的数据结构，它包含了该类所有的虚方法的信息，包括方法的地址、偏移量等。每个对象在内存中都有一个指向其所属类的虚方法表的指针，称为`vtable`指针。

当调用一个虚方法时，`Java`虚拟机首先根据对象的实际类型找到其对应的虚方法表，然后根据方法的偏移量找到要调用的方法的地址，并执行该方法。如果子类重写了父类的虚方法，则子类的虚方法表中会覆盖父类相应的方法地址，从而实现了多态。

以下是一个示例代码：

```java
class Animal {

    public void eat() {

        System.out.println("Animal is eating");

    }

}

class Cat extends Animal {

    public void eat() {

        System.out.println("Cat is eating");

    }

}

class Dog extends Animal {

    public void eat() {

        System.out.println("Dog is eating");

    }

}

public class Main {

    public static void main(String[] args) {

        Animal a = new Animal();

        Animal b = new Cat();

        Animal c = new Dog();

        a.eat(); // Animal is eating

        b.eat(); // Cat is eating

        c.eat(); // Dog is eating

    }

}
```

在上面的代码中，`Animal`、`Cat`和`Dog`都有一个`eat`方法，但是它们的行为不同。在`main`方法中，分别创建了一个`Animal`、一个`Cat`和一个`Dog`对象，并调用它们的`eat`方法。由于`eat`方法是虚方法，因此在运行时会根据对象的实际类型来确定要调用哪个方法，从而实现了多态。

### 方法重写

方法重写指的是在子类中定义一个与父类方法名、返回类型、参数列表都相同的方法，但是方法体不同的过程。

当子类对象调用被重写的方法时，将优先调用子类中的方法，而不是父类中的方法。

方法重写的条件为：

- 方法名、返回类型、参数列表必须与父类中被重写的方法相同。
- 访问权限不能低于父类中被重写的方法的访问权限。
- 子类方法抛出的异常不能大于父类方法抛出的异常。
- 静态方法不能被重写，但是可以被隐藏。

下面是一个方法重写的例子：

```java
public class Animal {
    public void move() {
        System.out.println("动物在移动");
    }
}

public class Dog extends Animal {
    @Override
    public void move() {
        System.out.println("狗在奔跑");
    }
}

public class Test {
    public static void main(String[] args) {
        Animal animal = new Animal();
        animal.move(); // 输出：动物在移动

        Dog dog = new Dog();
        dog.move(); // 输出：狗在奔跑
    }
}
```

在上面的例子中，`Animal`类中定义了一个`move()`方法，输出“动物在移动”。在`Dog`类中重写了这个方法，并输出“狗在奔跑”。当我们调用`animal.move()`时，输出的是父类`Animal`中的`move()`方法的内容；当我们调用`dog.move()`时，输出的是子类`Dog`中重写的`move()`方法的内容。

### static 修饰符

`static`是静态的意思，可以修饰成员变量和成员方法；`static`修饰的成员变量和成员方法在内存中只用存储一份，因为可以被类和对象共享访问、修改。

`static`注意事项：

1、静态方法只能访问静态成员(静态变量或者静态方法)，不能直接访问实例成员，因为静态成员是属于类的，而不是直接属于对象的，静态成员会同类一同创建。

2、实例方法可以访问静态成员，也可以访问实例成员，因为静态成员是可以被共享访问的。

3、静态方法中不可以出现`this`关键字，因为`this`只能代表当前对象，而静态方法不一定使用对象调用。

4、静态方法中也不能使用`super`关键字，因为`super`关键字代表父类对象，而静态方法没有对象的概念。

5、静态方法也不能被子类重写，因为静态方法是属于类的，而不是属于对象的。

```java
package com.java.day1_static;

public class User {

    public static int usernum;

    public static int Val(int num1, int num2) {

        return Math.max(num1, num2);

    }

}
```

### 静态代码块
