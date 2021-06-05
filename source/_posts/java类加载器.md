---
title: java类加载器
date: 2021-03-03 15:38:06
---

### 类加载的过程
java源文件（.java文件）经过编译器编译之后转换成java字节码文件（.class文件），类加载器会通过调用`loadClass`读取java字节码，调用`defineClass`转换成java.lang.Class类的一个实例（对象），存储在元空间（jdk8）,前者称为初始加载器（initiating loader）,后者称为一个类的定义加载器（defining loader）。我们可以通过调用该类的构造方法创建该类的实例（对象），其中java字节码可能是动态生成的，也可能是通过网络下载的。
### 类加载器
* **bootstrap class loader：**它用来加载 Java 的核心库，是用原生代码来实现的，并不继承自 java.lang.ClassLoader ，`Launcher.getBootstrapClassPath()`可以获取加载的类路径。
* **extensions class loader：**它负责加载JRE的扩展目录，lib/ext或者由java.ext.dirs系统属性指定的目录中的JAR包的类。由Java语言实现，父类加载器为null。
* **system class loader：**它根据 Java 应用的类路径（CLASSPATH）来加载 Java 类。一般来说，Java 应用的类都是由它来完成加载的。可以通过 ClassLoader.getSystemClassLoader() 来获取它。

### 类加载器的代理模式（双亲委派）
**方式：**类加载器在尝试自己去查找某个类的字节代码并定义它时，会先代理给其父类加载器，由父类加载器先去尝试加载这个类，依次类推。
**JVM虚拟机是如何判断两个类是相同的：** `类的全限定名是否相同` ,`加载此类的类加载器是否一样`,也就是说同一个类被不同的类加载器加载，也会被认为是不同的类。
**双亲委派的作用：**这种机制保证了java核心库的类型安全。例，java.lang.Object类，如果加载此类的时候是由自己的类加载器来完成的话，就会存在多个版本的java.lang.Object类，互相不兼容，双亲委派规避了这种错误。
**不能解决的问题：**很多服务提供者接口（Service Provider Interface，SPI）例，JDBC、JCE、JNDI、JAXP 和 JBI 等，是属于java核心类库的一部分，由bootstrap class loader加载，其实现类一般由system class loader加载，这样接口类和实现类不会被认为是相同的。线程上下文类加载器正好解决了这个问题。

### 线程上下文类加载器
线程上下文类加载器（context class loader）是从 JDK 1.2 开始引入的。类 java.lang.Thread 中的方法 getContextClassLoader() 和 setContextClassLoader(ClassLoader cl) 用来获取和设置线程的上下文类加载器。如果没有通过 setContextClassLoader(ClassLoader cl) 方法进行设置的话，线程将继承其父线程的上下文类加载器。Java 应用运行的初始线程的上下文类加载器是系统类加载器。在线程中运行的代码可以通过此类加载器来加载类和资源。

### 自定义类加载器
``` java
public class FileSystemClassLoader extends ClassLoader {

    private String rootDir;

    public FileSystemClassLoader(String rootDir) {
        this.rootDir = rootDir;
    }

    protected Class<?> findClass(String name) throws ClassNotFoundException {
        byte[] classData = getClassData(name);
        if (classData == null) {
            throw new ClassNotFoundException();
        }
        else {
            return defineClass(name, classData, 0, classData.length);
        }
    }

    private byte[] getClassData(String className) {
        String path = classNameToPath(className);
        try {
            InputStream ins = new FileInputStream(path);
            ByteArrayOutputStream baos = new ByteArrayOutputStream();
            int bufferSize = 4096;
            byte[] buffer = new byte[bufferSize];
            int bytesNumRead = 0;
            while ((bytesNumRead = ins.read(buffer)) != -1) {
                baos.write(buffer, 0, bytesNumRead);
            }
            return baos.toByteArray();
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    private String classNameToPath(String className) {
        return rootDir + File.separatorChar
                + className.replace('.', File.separatorChar) + ".class";
    }
 }
```
类 FileSystemClassLoader 继承自类 java.lang.ClassLoader 。在 java.lang.ClassLoader 类介绍 中列出的 java.lang.ClassLoader 类的常用方法中，一般来说，自己开发的类加载器只需要覆写 findClass(String name) 方法即可。 java.lang.ClassLoader 类的方法 loadClass() 封装了前面提到的代理模式的实现。该方法会首先调用 findLoadedClass() 方法来检查该类是否已经被加载过；如果没有加载过的话，会调用父类加载器的 loadClass() 方法来尝试加载该类；如果父类加载器无法加载该类的话，就调用 findClass() 方法来查找该类。因此，为了保证类加载器都正确实现代理模式，在开发自己的类加载器时，最好不要覆写 loadClass() 方法，而是覆写 findClass() 方法。

### 类加载器与 Web 容器
以 Apache Tomcat 来说，每个 Web 应用都有一个对应的类加载器实例。该类加载器也使用代理模式，所不同的是它是首先尝试去加载某个类，如果找不到再代理给父类加载器。这与一般类加载器的顺序是相反的。这是 Java Servlet 规范中的推荐做法，其目的是使得 Web 应用自己的类的优先级高于 Web 容器提供的类。这种代理模式的一个例外是：Java 核心库的类是不在查找范围之内的。这也是为了保证 Java 核心库的类型安全。

### 类加载器与 OSGi
OSGi™ 是 Java 上的动态模块系统。它为开发人员提供了面向服务和基于组件的运行环境，并提供标准的方式用来管理软件的生命周期。OSGi 已经被实现和部署在很多产品上，在开源社区也得到了广泛的支持。Eclipse 就是基于 OSGi 技术来构建的。

OSGi 中的每个模块（bundle）都包含 Java 包和类。模块可以声明它所依赖的需要导入（import）的其它模块的 Java 包和类（通过 Import-Package ），也可以声明导出（export）自己的包和类，供其它模块使用（通过 Export-Package ）。也就是说需要能够隐藏和共享一个模块中的某些 Java 包和类。这是通过 OSGi 特有的类加载器机制来实现的。OSGi 中的每个模块都有对应的一个类加载器。它负责加载模块自己包含的 Java 包和类。当它需要加载 Java 核心库的类时（以 java 开头的包和类），它会代理给父类加载器（通常是启动类加载器）来完成。当它需要加载所导入的 Java 类时，它会代理给导出此 Java 类的模块来完成加载。模块也可以显式的声明某些 Java 包和类，必须由父类加载器来加载。只需要设置系统属性 org.osgi.framework.bootdelegation 的值即可。

假设有两个模块 bundleA 和 bundleB，它们都有自己对应的类加载器 classLoaderA 和 classLoaderB。在 bundleA 中包含类 com.bundleA.Sample ，并且该类被声明为导出的，也就是说可以被其它模块所使用的。bundleB 声明了导入 bundleA 提供的类 com.bundleA.Sample ，并包含一个类 com.bundleB.NewSample 继承自 com.bundleA.Sample 。在 bundleB 启动的时候，其类加载器 classLoaderB 需要加载类 com.bundleB.NewSample ，进而需要加载类 com.bundleA.Sample 。由于 bundleB 声明了类 com.bundleA.Sample 是导入的，classLoaderB 把加载类 com.bundleA.Sample 的工作代理给导出该类的 bundleA 的类加载器 classLoaderA。classLoaderA 在其模块内部查找类 com.bundleA.Sample 并定义它，所得到的类 com.bundleA.Sample 实例就可以被所有声明导入了此类的模块使用。对于以 java 开头的类，都是由父类加载器来加载的。如果声明了系统属性 org.osgi.framework.bootdelegation=com.example.core.* ，那么对于包 com.example.core 中的类，都是由父类加载器来完成的。

* 如果一个类库只有一个模块使用，把该类库的 jar 包放在模块中，在 Bundle-ClassPath 中指明即可。
* 如果一个类库被多个模块共用，可以为这个类库单独的创建一个模块，把其它模块需要用到的 Java 包声明为导出的。其它模块声明导入这些类。
* 如果类库提供了 SPI 接口，并且利用线程上下文类加载器来加载 SPI 实现的 Java 类，有可能会找不到 Java 类。如果出现了 NoClassDefFoundError 异常，首先检查当前线程的上下文类加载器是否正确。通过 Thread.currentThread().getContextClassLoader() 就可以得到该类加载器。该类加载器应该是该模块对应的类加载器。如果不是的话，可以首先通过 class.getClassLoader() 来得到模块对应的类加载器，再通过 Thread.currentThread().setContextClassLoader() 来设置当前线程的上下文类加载器。

