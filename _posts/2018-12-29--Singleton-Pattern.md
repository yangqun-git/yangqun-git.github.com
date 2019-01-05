---
layout: post
title: 学习笔记 - 单例模式
image: /public/images/yunshan.jpg
tags: ['学习笔记','设计模式']

---

#### 定义：

> 保证一个类**仅有**一个实例，并提供一个全局访问点。

它的定义还是比较简单的，但是单例模式即简单又复杂。简单是指它上手容易一看就懂，至于为什么复杂请看下文。

#### 适用场景：

>  想确保**任何情况**下都**绝对**只有一个实例。

单例模式的适用场景也很容易理解，而且它的实际应用也有很多，比如线程池、数据库的连接池、应用配置等等。

#### 优点：

* 在内存里只有一个实例，减少了内存开销。
* 设置全局访问点，严格控制访问。

#### 缺点：

* 扩展困难，只能修改代码。

#### 重点：

* 私有构造器：为了禁止从外部调用构造器创建这个对象，所以必须把构造器访问权限设置为`private`。
* 线程安全
* 延迟加载
* 序列化和反序列化安全：单例对象一旦序列化或反序列化就会产生不一样的对象从而破坏单例模式。
* 反射攻击



现在我们来开始学习单例模式，单例模式也分很多种，我们通过演进的方式一点点体会它们的不同以及优缺点。

#### 懒汉式

```java
public class LazySingleton {
    /**
     * 构造器一定是private的
     */
    private LazySingleton(){}

    // 声明一个静态的要被单例的这个对象
    // 懒汉式可以理解为它比较懒,在初始化的时候它是没有创建的,而是做一个延迟加载
    private static LazySingleton lazySingleton = null;

    /**
     * 声明一个获取单例对象的方法
     */
    public static LazySingleton getInstance(){
        if (null == lazySingleton){
            lazySingleton = new LazySingleton();
        }
        return lazySingleton;
    }

}
```

这种方式它是线程不安全的，我们看一下代码，在单线程的时候这种写法是ok的，但是一旦多线程情况下使用这个单例的话就会出问题。

我们假想现在有两个线程，Thread-0执行到代码的`lazySingleton = new LazySingleton()`这一行但是还没有执行完，这样的话`lazySingleton`对象还没有被正确赋值，这个时候Thread-1执行到了`if (null == lazySingleton)`，因为Thread-0没有赋值上，Thread-1判断结果为`true`，Thread-1也会执行实例化代码，这样就产生了两个不同的对象。如果是更多个线程呢？

所以在第一次初始化的时候就会创建很多个单例对象，如果这个单例对象很消耗资源，那很有可能引发系统故障，这个就是非常危险的。在我们这个例子中被没有什么消耗资源的地方，但是场景是一样的。

既然有隐患，那我们就一定要消除它，有几种改进方案，我们一起来看一下：

##### 同步方法

```java
    public synchronized static LazySingleton getInstance() {
    	...    
    }
```

我们在方法声明上加一个`synchronized`关键字，使这个方法变成一个静态方法。我们将`synchronized`关键字放在静态方法上，实际上锁的是这个类的`.class`文件，如果不是静态方法，相当于锁的堆内存中生成的对象，	也就是说我们现在是锁了整个类。它和下面这个写法的意义是一样的：

```java
    public static LazySingleton getInstance(){
        synchronized (LazySingleton.class){
            if (null == lazySingleton){
                lazySingleton = new LazySingleton();
            }
        }
        return lazySingleton;
    }
```

这种同步的方式我们解决了懒汉式在多线程下的问题，那我们也知道同步锁比较消耗资源，这里面有加锁和解锁的一个开销，而且我们锁的是整个类，这个锁的范围也是非常大的，对性能会有一定影响。

那我们有没有一种方式，在性能和安全性方面取得平衡？答案是有的，现在就继续演进我们的懒汉式。

##### DoubleCheck

双重检查这种方式兼顾了性能和安全性，并且还是懒加载的。

```java
public class LazyDoubleCheckSingleton {
    private LazyDoubleCheckSingleton() {
    }

    private static LazyDoubleCheckSingleton lazyDoubleCheckSingleton = null;

    /**
     * DoubleCheck关注的是双重检查
     * 首先方法不需要锁了，把锁放进了方法体
     */
    public static LazyDoubleCheckSingleton getInstance() {
        // 第一重判断
        if (null == lazyDoubleCheckSingleton) {
            // 这里注意：执行到这里，也就代表这个if是进来的，
            // 因为上一行代码没有锁，这个时候如果另外一个线程进来，判断为空依然会被synchronized阻塞
            // 我们想象一下如果没有第二重判断，依然会有问题，当然这里依然还有个小坑，一会儿讲
            synchronized (LazyDoubleCheckSingleton.class){
                // 第二重判断
                if (null == lazyDoubleCheckSingleton){
                    lazyDoubleCheckSingleton = new LazyDoubleCheckSingleton();
                }
            }
        }
        return lazyDoubleCheckSingleton;
    }
}
```

这样写解决了`synchronized`加在方法上的开销。

看上去我们这个实现非常完美，当多线程的时候，通过加锁保证了只有一个线程能创建对象。对象创建好后，以后再调用`getInstance()`方法的时候都不会再需要加锁，直接返回已创建好的对象。

但是我们这段程序依然有隐患，隐患出在第一次判空和实例化这行，现在说一下为什么：

首先在第一次判空的时候，虽然判断了这个对象是不是为空，这个时候是有可能它不为空的，虽然不为空，但是它很有可能还没有完成初始化，也就是说` lazyDoubleCheckSingleton = new LazyDoubleCheckSingleton()`还没有执行完成。

我们看下这一行代码：

`lazyDoubleCheckSingleton = new LazyDoubleCheckSingleton();`

这行代码看上去是一行,实际上这里经历了三个步骤:

​	1、分配内存给这个对象

​	2、初始化对象

​	3、设置`lazyDoubleCheckSingleton`指向刚刚分配的内存地址

隐患是在执行步骤2、3的时候可能会发生**指令重排序**，即他们的执行顺序可能会颠倒。

在java语言规范中，所有的线程在执行java程序时必须要遵守：**intra-thread semantics**这样一个规定，含义是保证重排序不会改变单线程内的程序执行结果。

 对于我们这个情况来说，单线程下2和3调换顺序不会影响到程序结果。

但是多线程下呢？

![单例模式](/public/images/2018/single-leton.jpg)

当对象指向内存空间之后，再对对象判空结果就会为false，这样其他线程拿到的是一个没有初始化完毕的对象，程序就会发成错误。

既然知道了问题所在我们就来解决这个问题，既然是重排序导致的问题，那我们就不让他重排序好了。

```java
private volatile static  LazyDoubleCheckSingleton lazyDoubleCheckSingleton = null;
```

我们只需要做这样一个小小的改动，加上`volatile`关键字。

至于这个关键字在这里所起的作用可以看这篇文章：[The "Double-Checked Locking is Broken" Declaration](http://www.cs.umd.edu/~pugh/java/memoryModel/DoubleCheckedLocking.html){:target="_blank"}

这里还有个知识点，就是内存可见性的问题，小伙伴们可以看看慕课网的这个免费课程，这位老师讲的很好的：[细说Java多线程之内存可见性](https://www.imooc.com/learn/352){:target="_blank"}

##### 静态内部类

通过静态内部类来实现单例的延迟加载。

Jvm在类的初始化阶段（Class被加载后，被线程使用前），jvm会去获取一个锁，这个锁可以同步多个线程对一个类的初始化操作，基于这个特性，我们可以用静态内部类来实现一个线程安全的并且延迟加载的解决方案。

```java
public class StaticInnerClassSingleton {

    private StaticInnerClassSingleton(){}

    private static class InnerClass{
        private static StaticInnerClassSingleton staticInnerClassSingleton 
            					= new StaticInnerClassSingleton();
    }

    public static StaticInnerClassSingleton getInstance(){
        return InnerClass.staticInnerClassSingleton;
    }
}

```

这种情况下，静态内部类只有被使用到的时候才会被加载，而且不管你有没有重排序都无所谓，不会被另一个线程看到，所以是线程安全的。



> 静态内部类和前面的DoubleCheck都是为了延迟加载，采用什么方案看情况而定。另一点关于面试方面的，只要面试问到设计模式，那么单例模式是百分之99会被问到，如果能这样迭代式的回答出来了，那么面试官给你的打分一定不低。

#### 饿汉式

这是单例模式实现最简单的一种方式，也就是在类加载的时候完成实例化。

```java
public class HungrySingleton {

    private HungrySingleton(){}

    private final static HungrySingleton hungrySingleton = new HungrySingleton();

    public static HungrySingleton getInstance(){
        return hungrySingleton;
    }
}
```

这样一个饿汉式就写完了，它的优点是写法简单，不用考虑线程问题。缺点就是没有延迟加载，如果整个系统运行期间都没有用到这个类，它就白白的占着资源。



---

单例模式的演进就告一段落了，接下来讲讲怎么破坏掉我们上面的单例模式，即让它不再单例！

#### 序列化和反序列化破坏单例

以上一步实现的`HungrySingleton`为例：

```java
public class HungrySingletonTest {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        HungrySingleton instance = HungrySingleton.getInstance();
        String fileName = "singleton_file";
        // 把对象写入到文件里
        ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(fileName));
        oos.writeObject(instance);
        // 将文件里的对象加载出来
        ObjectInputStream ois = new ObjectInputStream(new FileInputStream(new File(fileName)));
        HungrySingleton newInstance = (HungrySingleton) ois.readObject();

        // 会报一个NotSerializableException异常,因为我们的HungrySingleton对象没有实现Serializable接口
        // 这里是为了让小伙伴门加深一下印象. 实现接口重新运行结果如行尾注释所见

        System.out.println(instance); // HungrySingleton@506e1b77
        System.out.println(newInstance); // HungrySingleton@45ff54e6
        System.out.println(instance == newInstance); // false
    }
}
```

结果是`false`，但是这就违背了我们单例模式的初衷，我们希望序列化和反序列化后依然为同一个对象。

解这个问题呢也不难，重要的是要理解，为什么这么解，了解它的原理。

我们找到这个HungrySingleton类，在里面实现一个方法：

```java
public class HungrySingleton implements Serializable {

    private HungrySingleton(){}

    private final static HungrySingleton hungrySingleton = new HungrySingleton();

    public static HungrySingleton getInstance(){
        return hungrySingleton;
    }

    /**
     * 防止序列化破坏单例
     */
    private Object readResolve(){
        return hungrySingleton;
    }
}
```

改好之后我们重新执行以下上面的测试类，发现结果已然变成了`true`，小伙伴们一定有疑问，为什么实现这样一个方法就可以了呢，这个方法为什么叫这个名字呢？改成别的名字可以吗？。。。

>讲到这里说些题外话，我认为要成为一个好的程序员，学习任何东西不要只是会用就行，而是要深究其原理，知其所以然。然人的精力是有限的, 所以这个“深”的度也要把握好，用有限的精力创造出对自己而言最大的价值。

这个时候我们需要先看一下`ObjectInputStream#readObject`方法，小伙伴们也把源码点开，一起来看一下：

```java
    public final Object readObject()
        throws IOException, ClassNotFoundException
    {
        if (enableOverride) {
            return readObjectOverride();
        }

        // if nested read, passHandle contains handle of enclosing object
        int outerHandle = passHandle;
        try {
            // 注意这个obj，发现就是最后返回的对象
            Object obj = readObject0(false);
            handles.markDependency(outerHandle, passHandle);
            ClassNotFoundException ex = handles.lookupException(passHandle);
            if (ex != null) {
                throw ex;
            }
            if (depth == 0) {
                vlist.doCallbacks();
            }
            return obj;
        } finally {
            passHandle = outerHandle;
            if (closed && depth == 0) {
                clear();
            }
        }
    }
```

这是整个方法，小伙们看下我补充的中文注释，其它的地方不是我们需要关注的重点，直接跟进这个readObject0方法。

```java
    private Object readObject0(boolean unshared) throws IOException {
       ...
        try {
            switch (tc) {
                ...
                case TC_OBJECT:
                    // 跟进readOrdinaryObject方法
                    return checkResolve(readOrdinaryObject(unshared));
			   ...
                default:
                    ...
            }
        } finally {
            ...
        }
    }
```

代码太长我就不贴完整的了，大家直接找到这句代码，在源码中的1570行。

我门可以看到它在checkReslve之前先执行了readOrdinaryObject方法，继续。

```java
    private Object readOrdinaryObject(boolean unshared)
        throws IOException
    {
        if (bin.readByte() != TC_OBJECT) {
            throw new InternalError();
        }

        ObjectStreamClass desc = readClassDesc(false);
        desc.checkDeserialize();

        Class<?> cl = desc.forClass();
        if (cl == String.class || cl == Class.class
                || cl == ObjectStreamClass.class) {
            throw new InvalidClassException("invalid class descriptor");
        }

        Object obj;
        try {
            // 采用反射方式创建的对象,也就解释了为什么反序列化会是两个不同的对象
            obj = desc.isInstantiable() ? desc.newInstance() : null;
        } catch (Exception ex) {
            throw (IOException) new InvalidClassException(
                desc.forClass().getName(),
                "unable to create instance").initCause(ex);
        }

        passHandle = handles.assign(unshared ? unsharedMarker : obj);
        ClassNotFoundException resolveEx = desc.getResolveException();
        if (resolveEx != null) {
            handles.markException(passHandle, resolveEx);
        }

        if (desc.isExternalizable()) {
            readExternalData((Externalizable) obj, desc);
        } else {
            readSerialData(obj, desc);
        }

        handles.finish(passHandle);
        // 跟进hasReadResolveMethod方法
        if (obj != null &&
            handles.lookupException(passHandle) == null &&
            desc.hasReadResolveMethod())
        {
            // 调用readResolve方法重新赋值.
            Object rep = desc.invokeReadResolve(obj);
            if (unshared && rep.getClass().isArray()) {
                rep = cloneArray(rep);
            }
            if (rep != obj) {
                // Filter the replacement object
                if (rep != null) {
                    if (rep.getClass().isArray()) {
                        filterCheck(rep.getClass(), Array.getLength(rep));
                    } else {
                        filterCheck(rep.getClass(), -1);
                    }
                }
                handles.setObject(passHandle, obj = rep);
            }
        }

        return obj;
    }
```

小伙伴直接看我贴在源码中的注释，然后继续。

```java
    /**
     * Returns true if represented class is serializable or externalizable and
     * defines a conformant readResolve method.  Otherwise, returns false.
     */
    boolean hasReadResolveMethod() {
        requireInitialized();
        return (readResolveMethod != null);
    }
```

单从方法上我们看不出来什么意思好像，好在源码中都有详细的注释，大致意思是说如果是个`serializable`类型或者`externalizable`类型，并且有某个方法，就返回`true`。

这个变量`readResolveMethod`在源码中是一个`Method`类型，查找他的引用发现它是在源码中521行被赋值：

```java
 readResolveMethod = getInheritableMethod(cl, "readResolve", null, Object.class)
```

结合`readOrdinaryObject `方法看这样我们就找到了为什么定义一个叫这个名字的方法就会防止序列化出不同对象原因。

关于单例序列化和反序列化就讲到这。

#### 反射破坏单例

额。。。这个我就不多说了，无非就是通过反射修改构造器权限，我感觉你们应该都懂- -

#### Enum枚举单例

这个网上也有好多文章介绍了，且介绍的也很到位，小伙伴们直接百度一下就好了，建议反编译一下随便一个枚举类试试看，里面会有很多有趣的知识点。你会发现新大陆的~



文章就先到这，还有一些内容没写，以后再补吧。





