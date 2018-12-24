---
title: JAVA反射
date: 2018-07-21 21:13:57
tags: 
- JAVA基础
---
在 Java中，反射机制（Reflection）非常重要。

## 介绍 ##
* 定义：Java语言中 一种 动态（运行时）访问、检测 & 修改它本身的能力
* 作用：允许运行中的 Java 程序获取自身的信息，并且可以操作类或对象的内部属性。  

简而言之，通过反射，我们可以在运行时获得程序或程序集中每一个类型的成员和成员的信息。  
程序中一般的对象的类型都是在编译期就确定下来的，而Java反射机制可以动态地创建对象并调用其属性，这样的对象的类型在编译期是未知的。所以我们可以通过反射机制直接创建对象，即使这个对象的类型在编译期是未知的。  
反射的核心是JVM在运行时才动态加载类或调用方法/访问属性，它不需要事先（写代码的时候或编译期）知道运行对象是谁。

  ---

## 优缺点 ##
* 优点  
灵活性高。因为反射属于动态编译，即只有到运行时才动态创建 &获取对象实例。
> 编译方式说明： 
> 1. 静态编译：在编译时确定类型 & 绑定对象。如常见的使用new关键字创建对象 
> 2. 动态编译：运行时确定类型 & 绑定对象。动态编译体现了Java的灵活性、多态特性 & 降低类之间的藕合性

* 缺点  
1. 执行效率低   
因为反射的操作 主要通过JVM执行，所以时间成本会 高于 直接执行相同操作
> 1. 因为接口的通用性，Java的invoke方法是传object和object[]数组的。基本类型参数需要装箱和拆箱，产生大量额外的对象和内存开销，频繁促发GC。
> 2. 编译器难以对动态调用的代码提前做优化，比如方法内联。
> 3. 反射需要按名检索类和方法，有一定的时间开销。

2. 容易破坏类结构   
因为反射操作饶过了源码，容易干扰类原有的内部逻辑

---
## Java Reflection API ###
在JDK中，主要由以下类来实现Java反射机制，这些类都位于java.lang.reflect包中。
* Field类：代表类的成员变量（成员变量也称为类的属性）。
    >表示Class对象所表示的类的成员变量，通过它可以在运行时动态修改成员变量的属性值(包含private)
* Method类：代表类的方法。
    >表示Class对象所表示的类的成员方法，通过它可以动态调用对象的方法(包含private)
* Constructor类：代表类的构造方法。
    >是Class 对象所表示的类的构造方法，利用它可以在运行时动态创建对象。
* Array类：提供了动态创建数组，以及访问数组元素的静态方法。

可见大部分都是和Class类相关，我们在下面就着重说下Class这个类。

### java.lang.Class ###

1. java.lang.Class简介

    >java.lang.Class类是**反射机制的基础**，是Reflection API中的核心类，存放着对应类型对象的运行时信息。

    实际上在Java中每个类都有一个Class对象，每当我们编写并且编译一个新创建的类就会产生一个对应Class对象并且这个Class对象会被保存在同名.class文件里(编译后的字节码文件保存的就是Class对象)，那为什么需要这样一个Class对象呢？是这样的，当我们new一个新对象或者引用静态成员变量时，Java虚拟机(JVM)中的类加载器子系统会将对应Class对象加载到JVM中，然后JVM再根据这个类型信息相关的Class对象创建我们需要实例对象或者提供静态变量的引用值。需要特别注意的是，手动编写的每个class类，无论创建多少个实例对象，在JVM中都只有一个Class对象，即在内存中每个类有且只有一个相对应的Class对象。

    到这我们也就可以得出以下几点信息：

    * Class类也是类的一种，与class关键字是不一样的。

    * 手动编写的类被编译后会产生一个Class对象，其表示的是创建的类的类型信息，而且这个Class对象保存在同名.class的文件中(字节码文件)。

    * Class类只存私有构造函数，因此对应Class对象只能有JVM创建和加载

    * Class类的对象作用是运行时提供或获得某个对象的类型信息，这点对于反射技术很重要

## 反射的使用 ##
### Class对象的获取 ###
- 对象的getClass()方法;  
    ```java
    Class<String> class = "String".getClass();   
    ```
- 类的.class(最安全/性能最好)属性;  
    ```java
    Class<String> class = String.class;  
    ```
- 运用Class.forName(String className)动态加载类,className需要是类的全限定名(最常用)  
    ```java
    Class<?> class = Class.forName("java.lang.String");  
    ```

### 创建实例 ###
- 使用Class对象的newInstance()方法
    ```java
    Class<?> class = String.class;  
    Object str = class.newInstance();    
    ```
- 通过Class对象获取指定的Constructor对象，再调用Constructor对象的newInstance()方法
    ```java
    // 获取String所对应的Class对象
    Class<?> class = String.class;  
    // 获取String类带一个String参数的构造器
    Constructor constructor = class.getConstructor(String.class);  
    // 根据构造器创建实例
    Object obj = constructor.newInstance("string");   
    ```

### 获取方法 ###
- `getDeclaredMethods()`方法返回类或接口声明的所有方法，包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法。
    ```java
    public Method[] getDeclaredMethods() throws SecurityException    
    ```

- `getMethods()`方法返回某个类的所有公用（public）方法，包括其继承类的公用方法。
    ```java
    public Method[] getMethods() throws SecurityException   
    ```
- `getMethod()`方法返回一个特定的方法，其中第一个参数为方法名称，后面的参数为方法的参数对应Class的对象。
    ```java
    public Method getMethod(String name, Class<?>... parameterTypes)   
    ```
- 代码示例：  
    ```java
    public class test {
        public static void main(String[] args){

            Class clazz = Circle.class;

            //根据参数获取public的Method,包含继承自父类的方法
            Method method = clazz.
            getMethod("draw", int.class, String.class);

            System.out.println("method:" + method);

            //获取所有public的方法:
            Method[] methods = clazz.getMethods();
            for (Method m : methods) {
                System.out.println("m::" + m);
            }

            System.out.printl
            ("=========================================");
        //获取当前类的方法包含private,该方法无法获取继承自父类的method
            Method method1 = clazz.getDeclaredMethod("drawCircle");
            System.out.println("method1::" + method1);
        //获取当前类的所有方法包含private,该方法无法获取继承自父类的method
            Method[] methods1 = clazz.getDeclaredMethods();
            for (Method m : methods1) {
                System.out.println("m1::" + m);
            }
        }
    }

    class Shape {
        public void draw() {
            System.out.println("draw");
        }

        public void draw(int count, String name) {
            System.out.println("draw " + name + ",count=" + count);
        }

    }

    class Circle extends Shape {

        private void drawCircle() {
            System.out.println("drawCircle");
        }

        public int getAllCount() {
            return 100;
        }
    }  
     ```
- 输出结果
> method:public void reflect.Shape.draw(int,java.lang.String) 
m::public int reflect.Circle.getAllCount()   
m::public void reflect.Shape.draw()  
m::public void reflect.Shape.draw(int,java.lang.String)  
m::public final void java.lang.Object.wait(long,int)  
m::public final native void java.lang.Object.wait(long)   
m::public final void java.lang.Object.wait()   
m::public boolean java.lang.Object.equals(java.lang.Object)  
m::public java.lang.String java.lang.Object.toString()  
m::public native int java.lang.Object.hashCode()  
m::public final native java.lang.Class java.lang.Object.getClass()  
m::public final native void java.lang.Object.notify()  
m::public final native void java.lang.Object.notifyAll()  

 > =========================================  
 > method1::private void reflect.Circle.drawCircle()  
 m1::public int reflect.Circle.getAllCount()  
 m1::private void reflect.Circle.drawCircle() 
 
> `注：Method类还有个很重要的方法invoke(),关于invoke()方法的详解我们找机会详解。`

### 获取构造方法 ###
- `getConstructors()`返回所有具有public访问权限的构造函数的Constructor数组。
    ```java
    public Constructor<?>[] getConstructors() throws SecurityException 
    ```

- `getDeclaredConstructors()`返回所有声明的（包括private）构造函数数组。
    ```java
    public Constructor<?>[] getDeclaredConstructors() throws SecurityException   
    ```
- `getConstructor()`返回指定参数类型、具有public访问权限的构造函数。
    ```java
    public Constructor<T> getConstructor(Class<?>... parameterTypes)  
    ```
- `getDeclaredConstructor()`返回指定参数类型、所有声明的（包括private）构造函数。
    ```java
    public Constructor<T> getDeclaredConstructor(Class<?>... parameterTypes) throws NoSuchMethodException, SecurityException   
    ```
- 代码示例:   
     ```java
    public class test {
        public static void main(String[] args) throws Exception {

            Class<?> userClass = User.class;

        //第一种方法，实例化默认构造方法，User必须无参构造函数,否则将抛异常
            User user = (User) userClass.newInstance();
            user.setAge(20);
            user.setName("showtime");
            System.out.println("user" + user.toString());

            System.out.println
            ("--------------------------------------------");

            //获取带String参数的public构造函数
            Constructor constructor = userClass.
            getConstructor(String.class);
            //创建User
            User user1 = (User) constructor.newInstance("philipfry");
            user1.setAge(22);
            System.out.println("user1:" + user1.toString());

            System.out.println
            ("--------------------------------------------");

            //取得指定带int和String参数构造函数,该方法是私有构造private
            Constructor declaredConstructor =
            userClass.getDeclaredConstructor(int.class, String.class);
            //由于是private必须设置可访问
            declaredConstructor.setAccessible(true);
            //创建user对象
            User user2 = (User) declaredConstructor.
            newInstance(25, "yiran");
            System.out.println("user2:" + user2.toString());

            System.out.println
            ("--------------------------------------------");

            //获取所有构造包含private
            Constructor<?> cons[] = userClass.
            getDeclaredConstructors();
            // 查看每个构造方法需要的参数
            for (int i = 0; i < cons.length; i++) {
                //获取构造函数参数类型
                Class<?> clazzs[] = cons[i].getParameterTypes();
                System.out.println("构造函数[" + i + "]:" + 
                cons[i].toString());
                System.out.print("参数类型[" + i + "]:(");
                for (int j = 0; j < clazzs.length; j++) {
                    if (j == clazzs.length - 1)
                        System.out.print(clazzs[j].getName());
                    else
                        System.out.print(clazzs[j].getName() + ",");
                }
                System.out.println(")");
            }
        }
    }

    class User {
        private int age;
        private String name;

        public User() {
        }

        public User(String name) {
            this.name = name;
        }

        /**
        * 私有构造
        *
        * @param age
        * @param name
        */
        private User(int age, String name) {
            this.age = age;
            this.name = name;
        }

        public int getAge() {
            return age;
        }

        public void setAge(int age) {
            this.age = age;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

    }   
     ```
- 运行结果
> User [age=20, name=showtime]    
user1:User [age=22, name=philipfry]
user2:User [age=25, name=yiran]  
构造函数[0]:private reflect.User(int,java.lang.String)  
参数类型[0]:(int,java.lang.String)  
构造函数[1]:public reflect.User(java.lang.String)  
参数类型[1]:(java.lang.String)  
构造函数[2]:public reflect.User()  
参数类型[2]:() 
 
### 获取属性 ###
- `getFields()`获取修饰符为public的字段，包含继承属性。
    ```java
    public Field[] getFields() throws SecurityException    
    ```

- `getDeclaredFields()`获取Class对象所表示的类或接口的所有(包含private修饰的)字段,不包括继承的属性。
    ```java
    public Field[] getDeclaredFields() throws 
    SecurityException   
    ```
- `getField(String name)`返回指定参数类型、具有public访问权限的构造函数。
    ```java
    public Field getField(String name) throws 
    NoSuchFieldException, SecurityException   
    ```
- `getDeclaredField(String name)`获取指定name名称的(包含private修饰的)字段，不包括继承的属性。
    ```java
    public Field getDeclaredField(String name) throws  NoSuchFieldException, SecurityException   
    ```
- 代码示例:
    ```java
    public class test {
        public static void main(String[] args) throws ClassNotFoundException, NoSuchFieldException {
            Class<?> clazz = Student.class;
            //获取指定字段名称的Field类,
            //注意字段修饰符必须为public而且存在该字段,
            // 否则抛NoSuchFieldException
            Field field = clazz.getField("age");
            System.out.println("field:"+field);

            //获取所有修饰符为public的字段,包含父类字段,
            //注意修饰符为public才会获取
            Field fields[] = clazz.getFields();
            for (Field f:fields) {
                System.out.println("f:"+f.getDeclaringClass());
            }

            System.out.println("================getDeclaredFields====================");
            //获取当前类所字段(包含private字段),注意不包含父类的字段
            Field fields2[] = clazz.getDeclaredFields();
            for (Field f:fields2) {
                System.out.println("f2:"+f.getDeclaringClass());
            }
            //获取指定字段名称的Field类,可以是任意修饰符的自动,
            //注意不包含父类的字段
            Field field2 = clazz.getDeclaredField("desc");
            System.out.println("field2:"+field2);
        }
    }

    class Person{
        public int age;
        public String name;

        public int getAge() {
            return age;
        }

        public void setAge(int age) {
            this.age = age;
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }
    }

    class Student extends Person{
        public String desc;
        private int score;

        public String getDesc() {
            return desc;
        }

        public void setDesc(String desc) {
            this.desc = desc;
        }

        public int getScore() {
            return score;
        }

        public void setScore(int score) {
            this.score = score;
        }
    }
    ```
- 输出结果
> field:public int reflect.Person.age  
f:public java.lang.String reflect.Student.desc  
f:public int reflect.Person.age  
f:public java.lang.String reflect.Person.name  
================getDeclaredFields====================  
f2:public java.lang.String reflect.Student.desc  
f2:private int reflect.Student.score  
field2:public java.lang.String reflect.Student.desc  