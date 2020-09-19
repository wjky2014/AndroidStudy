

 
##### 和您一起终身学习，这里是程序员Android

#1、哪些情况下的对象会被垃圾回收机制处理掉？

利用可达性分析算法，虚拟机会将一些对象定义为GC Roots，从GC Roots出发沿着引用链向下寻找，如果某个对象不能通过GC Roots寻找到，虚拟机就认为该对象可以被回收掉。

#####1.1 哪些对象可以被看做是GC Roots呢？
1）虚拟机栈（栈帧中的本地变量表）中引用的对象；
2）方法区中的类静态属性引用的对象，常量引用的对象；
3）本地方法栈中JNI(Native方法）引用的对象；

#####1.2 对象不可达，一定会被垃圾收集器回收么？
即使不可达，对象也不一定会被垃圾收集器回收
1）先判断对象是否有必要执行finalize()方法，对象必须重写finalize()方法且没有被运行过。
2）若有必要执行，会把对象放到一个队列中，JVM会开一个线程去回收它们，这是对象最后一次可以逃逸清理的机会。

#2、讲一下常见编码方式？
编码的意义：计算机中存储的最小单元是一个字节即8bit，所能表示的字符范围是255个，而人类要表示的符号太多，无法用一个字节来完全表示，固需要将符号编码，将各种语言翻译成计算机能懂的语言。

#####1）ASCII码：
总共128个，用一个字节的低7位表示，0〜31控制字符如换回车删除等；32~126是打印字符，可通过键盘输入并显示出来；

#####2）ISO-8859-1
用来扩展ASCII编码，256个字符，涵盖了大多数西欧语言字符。

#####3）GB2312
双字节编码，总编码范围是A1-A7,A1-A9是符号区，包含682个字符，B0-B7是汉字区，包含6763个汉字；

#####4）GBK
为了扩展GB2312,加入了更多的汉字，编码范围是8140~FEFE，有23940个码位，能表示21003个汉字。

#####5）UTF-16
 ISO试图想创建一个全新的超语言字典，世界上所有语言都可通过这本字典Unicode来相互翻译，而UTF-16定义了Unicode字符在计算机中存取方法，用两个字节来表示Unicode转化格式。不论什么字符都可用两字节表示，即16bit，固叫UTF-16。

#####6）UTF-8：
UTF-16统一采用两字节表示一个字符，但有些字符只用一个字节就可表示，浪费存储空间，而UTF-8采用一种变长技术，每个编码区域有不同的字码长度。  不同类型的字符可以由1~6个字节组成。                                                                                                                                                                                                                       

#3、utf-8编码中的中文占几个字节；int型几个字节？
utf-8是一种变长编码技术，utf-8编码中的中文占用的字节不确定，可能2个、3个、4个，int型占4个字节。

#4、静态代理和动态代理的区别，什么场景使用？
代理是一种常用的设计模式，
#####目的是：
为其他对象提供一个代理以控制对某个对象的访问，将两个类的关系解耦。代理类和委托类都要实现相同的接口，因为代理真正调用的是委托类的方法。

#####区别：
1）静态代理：
由程序员创建或是由特定工具生成，在代码编译时就确定了被代理的类是哪一个是静态代理。静态代理通常只代理一个类；

2）动态代理：
在代码运行期间，运用反射机制动态创建生成。动态代理代理的是一个接口下的多个实现类；

#####实现步骤：
a.实现InvocationHandler接口创建自己的调用处理器；
b.给Proxy类提供ClassLoader和代理接口类型数组创建动态代理类；
c.利用反射机制得到动态代理类的构造函数；
d.利用动态代理类的构造函数创建动态代理类对象；

#####使用场景：
Retrofit中直接调用接口的方法；Spring的AOP机制；

#5、Java的异常体系

Java中Throwable是所有异常和错误的超类，两个直接子类是Error（错误）和Exception（异常）：

#####1）Error是程序无法处理的错误，由JVM产生和抛出

如OOM、ThreadDeath等。这些异常发生时，JVM一般会选择终止程序。

#####2）Exception是程序本身可以处理的异常

又分为运行时异常(RuntimeException)(也叫Checked Eception)和非运行时异常(不检查异常Unchecked Exception)。

**运行时异常**
有NullPointerException\IndexOutOfBoundsException等，这些异常一般是由程序逻辑错误引起的，应尽可能避免。
**非运行时异常**
有IOException\SQLException\FileNotFoundException以及由用户自定义的Exception异常等。

#6、谈谈你对解析与分派的认识。
#####解析
指方法在运行前，即编译期间就可知的，有一个确定的版本，运行期间也不会改变。解析是静态的，在类加载的解析阶段就可将符号引用转变成直接引用。

#####分派
可分为静态分派和动态分派，重载属于静态分派，覆盖属于动态分派。
静态分派是指在重载时通过参数的静态类型而非实际类型作为判断依据，在编译阶段，编译器可根据参数的静态类型决定使用哪一个重载版本。
动态分派则需要根据实际类型来调用相应的方法。

#7、修改对象A的equals方法的签名，那么使用HashMap存放这个对象实例的时候，会调用哪个equals方法？

会调用对象的equals方法，如果对象的equals方法没有被重写，equals方法和==都是比较栈内局部变量表中指向堆内存地址值是否相等。

#8、Java中实现多态的机制是什么？

多态是指程序中定义的引用变量所指向的具体类型和通过该引用变量发出的方法调用在编译时不确定，在运行期间才确定，一个引用变量到底会指向哪个类的实例。这样就可以不用修改源程序，就可以让引用变量绑定到各种不同的类实现上。

Java实现多态有三个必要条件：继承、重定、向上转型，在多态中需要将子类的引用赋值给父类对象，只有这样该引用才能够具备调用父类方法和子类的方法。

#9、如何将一个Java对象序列化到文件里？

ObjectOutputStream.writeObject()负责将指定的流写入，ObjectInputStream.readObject()从指定流读取序列化数据。
```
//写入
try {
    ObjectOutputStream os = new ObjectOutputStream(new FileOutputStream("D:/student.txt"));
    os.writeObject(studentList);
    os.close();
} catch(FileNotFoundException e) {
    e.printStackTrace();
} catch(IOException e) {
    e.printStackTrace();
}
```
#10、说说你对Java反射的理解
在运行状态中，对任意一个类，都能知道这个类的所有属性和方法，对任意一个对象，都能调用它的任意一个方法和属性。这种能动态获取信息及动态调用对象方法的功能称为java语言的反射机制。

#####反射的作用：
开发过程中，经常会遇到某个类的某个成员变量、方法或属性是私有的，或只对系统应用开放，这里就可以利用java的反射机制通过反射来获取所需的私有成员或是方法。

1) 获取类的Class对象实例 `Class clz = Class.forName("com.zhenai.api.Apple");`

2) 根据Class对象实例获取Constructor对象  `Constructor appConstructor = clz.getConstructor();
`
3) 使用Constructor对象的`newInstance`方法获取反射类对象 `Object appleObj = appConstructor.newInstance();
`
4) 获取方法的Method对象  `Method setPriceMethod = clz.getMethod("setPrice", int.class);
`
5) 利用invoke方法调用方法`  setPriceMethod.invoke(appleObj, 14);`

6) 通过getFields()可以获取Class类的属性，但无法获取私有属性，而getDeclaredFields()可以获取到包括私有属性在内的所有属性。带有Declared修饰的方法可以反射到私有的方法，没有Declared修饰的只能用来反射公有的方法，其他如Annotation\Field\Constructor也是如此。

#11、说说你对Java注解的理解
注解是通过@interface关键字来进行定义的，形式和接口差不多，只是前面多了一个@
```
public @interface TestAnnotation {

}
```
使用时@TestAnnotation来引用，要使注解能正常工作，还需要使用元注解，它是可以注解到注解上的注解。元标签有@Retention @Documented @Target @Inherited @Repeatable五种

@Retention说明注解的存活时间，取值有RetentionPolicy.SOURCE 注解只在源码阶段保留，在编译器进行编译时被丢弃；RetentionPolicy.CLASS 注解只保留到编译进行的时候，并不会被加载到JVM中。RetentionPolicy.RUNTIME可以留到程序运行的时候，它会被加载进入到JVM中，所以在程序运行时可以获取到它们。

@Documented 注解中的元素包含到javadoc中去

@Target  限定注解的应用场景，ElementType.FIELD给属性进行注解；ElementType.LOCAL_VARIABLE可以给局部变量进行注解；ElementType.METHOD可以给方法进行注解；ElementType.PACKAGE可以给一个包进行注解 ElementType.TYPE可以给一个类型进行注解，如类、接口、枚举

@Inherited 若一个超类被@Inherited注解过的注解进行注解，它的子类没有被任何注解应用的话，该子类就可继承超类的注解；

#####注解的作用：

1）提供信息给编译器：编译器可利用注解来探测错误和警告信息
2）编译阶段：软件工具可以利用注解信息来生成代码、html文档或做其它相应处理；
3）运行阶段：程序运行时可利用注解提取代码

注解是通过反射获取的，可以通过Class对象的isAnnotationPresent()方法判断它是否应用了某个注解，再通过getAnnotation()方法获取Annotation对象

#12、说一下泛型原理，并举例说明

泛型就是将类型变成参数传入，使得可以使用的类型多样化，从而实现解耦。Java泛型是在Java1.5以后出现的，为保持对以前版本的兼容，使用了擦除的方法实现泛型。擦除是指在一定程度无视类型参数T，直接从T所在的类开始向上T的父类去擦除，如调用泛型方法，传入类型参数T进入方法内部，若没在声明时做类似public T methodName(T extends Father t){}，Java就进行了向上类型的擦除，直接把参数t当做Object类来处理，而不是传进去的T。即在有泛型的任何类和方法内部，它都无法知道自己的泛型参数，擦除和转型都是在边界上发生，即传进去的参在进入类或方法时被擦除掉，但传出来的时候又被转成了我们设置的T。在泛型类或方法内，任何涉及到具体类型（即擦除后的类型的子类）操作都不能进行，如new T()，或者T.play()（play为某子类的方法而不是擦除后的类的方法）

#13、Java中String的了解

1）String类是final型，固String类不能被继承，它的成员方法也都默认为final方法。String对象一旦创建就固定不变了，对String对象的任何改变都不影响到原对象，相关的任何改变操作都会生成新的String对象。
2）String类是通过char数组来保存字符串的，String对equals方法进行了重定，比较的是值相等。
```
String a = "test"; String b = "test"; String c = new String("test");
```
a、b和字面上的test都是指向JVM字符串常量池中的"test"对象，他们指向同一个对象。而new关键字一定会产生一个对象test，该对象存储在堆中。所以new String("test")产生了两个对象，保存在栈中的c和保存在堆中的test。而在java中根本就不存在两个完全一模一样的字符串对象，故在堆中的test应该是引用字符串常量池中的test。
```
String str1 = "abc"; //栈中开辟一块空间存放引用str1，str1指向池中String常量"abc"
String str2 = "def"; //栈中开辟一块空间存放引用str2，str2指向池中String常量"def"
String str3 = str1 + str2;//栈中开辟一块空间存放引用str3
//str1+str2通过StringBuilder的最后一步toString()方法返回一个新的String对象"abcdef"
//会在堆中开辟一块空间存放此对象，引用str3指向堆中的(str1+str2)所返回的新String对象。
System.out.println(str3 == "abcdef");//返回false
```
因为str3指向堆中的"abcdef"对象，而"abcdef"是字符池中的对象，所以结果为false。JVM对String str="abc"对象放在常量池是在编译时做的，而String str3=str1+str2是在运行时才知道的，new对象也是在运行时才做的。

#14、String为什么要设计成不可变的？
#####1）字符串常量池需要String不可变
因为String设计成不可变，当创建一个String对象时，若此字符串值已经存在于常量池中，则不会创建一个新的对象，而是引用已经存在的对象。如果字符串变量允许必变，会导致各种逻辑错误，如改变一个对象会影响到另一个独立对象。

#####2）String对象可以缓存hashCode。

字符串的不可变性保证了hash码的唯一性，因此可以缓存String的hashCode，这样不用每次去重新计算哈希码。在进行字符串比较时，可以直接比较hashCode，提高了比较性能；

#####3）安全性
String被许多java类用来当作参数，如url地址，文件path路径，反射机制所需的Strign参数等，若String可变，将会引起各种安全隐患。

 

**友情推荐：**
[Android 干货分享 ](https://mp.weixin.qq.com/s/zOTO6z7bvHGhN0lhTMvR8w)

至此，本篇已结束。转载网络的文章，小编觉得很优秀，欢迎点击阅读原文，支持原创作者，如有侵权，恳请联系小编删除，欢迎您的建议与指正。同时期待您的关注，感谢您的阅读，谢谢！


![](https://upload-images.jianshu.io/upload_images/5851256-9398f7356f9c0525.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
