---
glayout: post
title: "TestNG深入理解"
date: 2013-06-05 20:10
comments: true
categories: testng
tags: [ testng ]
---
### TestNG annotaion:

- @DataProvider
- @ExpectedExceptions
- @Factory
- @Test
- @Parameters

---

```xml
<suite name="ParametersTest">
　　<test name="Regression1">
　　　　<classes>
　　　　　　<class name="com.example.ParameterSample" />
　　　　　　<class name="com.example.ParameterTest">
　　　　　　　　<mtehods>
　　　　　　　　　　<include name="database.*" />
　　　　　　　　　　<exclude name="inProgress" />
　　　　　　　　</methods>
　　　　　　</class>
　　　　</classes>
　　</test>
　　<test name="Parameters">
　　　　<packages>
　　　　　　<package name="test.parameters.Parameter*" />
　　　　</packages>
　　</test>
</suite>
```

一个suite(套件) 由一个或多个测试组成。
一个test(测试) 由一个或多个类组成
一个class(类) 由一个或多个方法组成。

@BeforeSuite/@AfterSuite 在某个测试套件开始之前/在某个套件所有测试方法执行之后
@BeforeTest/@AfterTest 在某个测试开始之前/在某个测试所有测试方法执行之后
@BeforeClass/@AfterClass 在某个测试类开始之前/在某个类的所有测试方法执行之后
@BeforeMethod/@AfterMethod 在某个测试方法之前/在某个测试方法执行之后
@BeforeGroup/@AfterGroup 在某个组的所有测试方法之前/在某个组的所有测试方法执行之后
<!--more-->

#### 1、分组

```java
@Test(groups = {"fast", "unit", "database"})
public void rowShouldBeInserted() {}

java org.testng.TestNG -groups fast com.example.MyTest
```
测试的一个目标就是确保代码按照预期的方式工作。这种测试要么在用户层面上进行，要么在编程层面上进行。这两种类型的测试分别是通过功能测试和单元测试来实现的。

针对失败而测试

Java提供了两种不同类型的异常：从java.lang.RuntimeException派生的运行时刻异常和从java.lang.Exception派生的被检查的异常。

抛出被检查的异常的经验法则：调用者可以为这个异常做什么吗？如果答案是肯定的，那么可能应该是用被检查的异常，否则，最好是选择运行时刻异常。


```java
@Test(expectedExceptions = {ReservationException.class, FlightCanceledException.class})
public void shouldThrowIfPlaneIsFull()
{
　　Plane plane = createPlane();
　　plane.bookAllSeats();
　　plane.bookPlane(createValidItinerary(), null);
}
```

属性expectedExceptions是一组类，包含了这个测试方法中预期会抛出的异常列表。如果没有抛出异常，或抛出的异常不再该属性的类表中，那么TestNG就会认为这个测试方法失败了。

单一职责：

```java
public class BookingTest
{
　　private Plane plane;

　　@BeforeMethod
　　public void init() { plane = createPlane(); }

　　@Test(expectedException = PlaneFullException.class)
　　public void shouldThrowIfPlaneIsFull()
　　{
　　　　plane.bookAllseats();
　　　　plane.bookPlane(createValidItinerary(), null);
　　}

　　@Test(expectedException = FlightCanceledException.class)
　　public void shouldThrowIfFlightIsCanceled()
　　{
　　　　cancelFlight(/* ... */);
　　　　plane.bookPlane(createValidItinerary(), null);
　　}
}
```


testng-failed.xml

当您执行包涵失败的测试套件时，TestNG会在输出目录(默认是test-output/)下自动生成一个名为testng-failded.xml的问他件。这个XML文件包含了原来的testng.xml中失败的方法所构成的子集。

```java
java org.testng.TestNG test.xml
java org.testng.TestNG test-output/testng-failed.xml
```


#### 2、工厂
TestNG让您可以选择自己将测试类实例化。这是通过@Factory annotation来实现的，他必须放在返回一个对象数组方法的顶部。所有这些对象都必须是包含TestNG annotation的类的实例。如果有@Factory annotation，那么这个循环会继续下去，知道TestNG拿到的都是没有@Factory annotation实例，或者@Factory方法都已被调用过的实例。

```java
public class ExpectedAttributes
{
　　private Image image;
　　private int width;
　　private height;
　　private String path;
　　
　　@Test
　　public void testWidth() {}

　　@Test
　　public void testHeight() {}

　　public PictureTest(String path, int width, int height, int depth) throws IOException
　　{
　　　　File f = new File(path);
　　　　this.path = path;
　　　　this.image = ImageIO.read(f);
　　　　this.width = width;
　　　　this.height = height;
　　}

　　private static String[] findImageFileNames() {}

　　@Factory
　　public static Object[] create() throws IOException
　　{
　　　　List result = new ArrayList();

　　　　String[] paths = findImageFileNames();

　　　　for (String path : paths) {
　　　　　　ExpectedAttributes ea = findAttributes(path);
　　　　　　result.add(new PictureTest(path, ea.width, ea.height, ea.depth));
　　　　}

　　　　return result.toArray();
　　}

　　public class ExpectedAttributes
　　{
　　　　public int width;
　　　　public int height;
　　　　public int depth;
　　}

　　private static ExpectedAttributes findExpectedAttributes(String path)
　　{
　　　　// ......
　　}
}
```

可以安全的在同一个类包含@Factory和@Test annotation，因为TestNG确保@Factory方法只被调用一次。


org.testng.ITest接口

```java
public interface ITest
{
	public String getTestName();
}
```
当TestNG遇到实现了这个接口的测试类时，他会在生成各种报告中包含getTestName()方法返回的信息。


```java
public class PictureTest implements ITest
{
　　public String getTestName()
　　{
　　　　return "[Picture: " + name + "]";
　　}
}
```


数据驱动测试

测试需要针对许多具有类似结构的数据来执行。
实际的测试逻辑是一样的，仅仅发生改变的是数据。
数据可以被一组不同的人修改。

参数测试方法
测试逻辑可以非常简单或不易改变，而提供給他的数据肯定会随着时间增长。


TestNG可以通过两种方式向测试方法传递参数：

- 利用testng.xml
- 利用DataProviders

1、利用testng.xml传递参数

```xml
<suite name="Parameters">
　　<parameter name="xml-file" value="accounts.xml" />
　　<parameter name="hostname" value="arkonis.example.com" />

　　<test name="ParameterTest">
　　　　<parameter name="hostname" value="terra.example.com" />
　　　　...
　　</test>
　　...
</suite>
```
在测试方法中指定参数

```java
@Test(parameters = {"xml-file"})
public void validateFile(String xmlFile)
{
　　// xmlFile has the value "accounts.xml"
}
```

如果犯下以下错误之一，TestNG将抛出一个异常：

- 在testng.xml中指定了一个参数，但不能转换为对应方法参数的类型。
- 声明了一个@Parameters annotation，但引用的参数名称在testng.xml中没有声明。



2.利用@DataProvider传递参数

如果需要向测试方法传递的参数不是基本的Java类型，或者如果需要的值智能在运行时刻创建，那么我们应该考虑使用@DataProvider annotation。

数据提供者是用@Dataprovider标注的方法。这个annotation只有一个字符串属性：他的名称，如果没有提供名称，数据提供者的名称就默认采用方法的名称。

数据提供者同时实现两个目的：

向测试方法传递任意数目的参数
根据需要，允许利用不同的参数集合对他的测试方法进行多次调用。

```java
@Test(dataProvider = "range-provider")
public void testIsBetWeen(int n, int lower, int upper, boolean expected)
{
　　println("Received " + n + " " + lower + "-" + upper + " expected: " + expected);
　　assert.assertEquals(expected, isBetween(n, lower, upper));
}

@DataProvider(name = "range-provider")
public Object[][] rangeData()
{
　　int lower = 5;
　　int upper = 10;

　　return new Object[][] {
　　　　{ lower-1, lower, upper, false},
　　　　{ lower, lower, upper, true},
　　　　{ lower+1, lower, upper, true},
　　　　{ upper, lower, upper, true},
　　　　{ upper+1, lower, upper, false},
　　};
}
```

由于数据提供者是测试类中的一个方法，他可以属于一个超类，然后被一些测试方法复用。我们也可以有几个数据提供者，只要他们定义在测试类或者他的一个子类上。当我们像在合适的地方记录数据源，并在几个测试方法中复用他时，这种方法是很方边的。

针对数据提供者的参数
数据提供者本身可以接受两个类型的参数：Method和ITestContext


```java
@DataProvider
public void craete() { ... }

@DataProvider
public void create(Method method) { ... }

@DataProvider
public void create(ITestContext context) { ... }

@DataProvider
public void create(Method method, ITestContext context) { ... }
```

Method参数
如果数据提供者的第一个参数是java.lang.reflect.Method，TestNG传递这个将调用的测试方法。如果您希望数据提供者根据不同的测试方法返回不同的数据，那么这种做法就非常有用。

```java
@DataProvider
public Object[][] provideNumbers(Method method)
{
　　String methodName = method.getName();

　　if (methodName.equals("tow")) {
　　　　return new Object[][] { new Object[] {2} };
　　}
　　if (methodName.equals("three")) {
　　　　return new Object[][] { new Object[] {3} };
　　}
}

@Test(dataProvider = "provideNumbers")
public void two(int param)
{
　　System.out.println("Two received: " + param);
}

@Test(dataProvider = "provideNumbers")
public void three(int param)
{
　　System.out.println("Three received: " + param);
}
```

使用同一个数据提供者的地方：

数据提供者代码相当复杂，应该保存在一个地方，这样维护起来更方便。
我们要传入数据的那些测试方法具有许多参数，其中只有少数参数是不一样的。
我们引入了某个方法的特殊情况。

ITestContext参数
如果一个数据提供者在方法签名中声名了一个ITestContext类型的参数，TestNG就会将当前的测试上下文设置给它，这使得数据提供者能够知道当前测试执行的运行时刻参数。

```java
@DataProvider
public Object[][] randomIntegers(ITestContext context)
{
　　String[] groups = context.getIncludeGroups();

　　int size = 2;
　　for (String group : groups) {
　　　　if (group.equals("functional-test")) {
　　　　　　size = 10;
　　　　　　break;
　　　　}
　　}

　　Object[][] result = new Object[size][];
　　Random r = new Random();
　　for (int i = 0; i < size; i++) {
　　　　result[i] = new Object[] { new Integer(r.nextInt()) };
　　}

　　return result;
}

@Test(dataProvider = "randomIntegers", groups = {"unit-test", "functional-test"})
public void random(Integer n)
{
　　// ......
}
```

ITestContext对象中的数据是运行时刻的信息，不是静态的信息：这个测试方法即属于unit-test组，也属于functional-test组，但在运行时刻，我们决定只执行functional-test组，这个值由ITestContext#getIncludeGroups方法返回。

延迟数据提供者

为了实现这种方法，TestNG允许我们从数据提供者返回一个Iterator，而不是一个二维对象数组。

这种方法与数组不同之处在于，当TestNG需要从数据提供者取得下一组参数时，他会调用Iterator的next方法，这样就有机会在最后一刻实例化相应的对象，即刚好在需要这些参数的测试方法被调用之前。

```java
@DataProvider(name = "generate-accounts-lazy")
public Iterator generateAccountsLazy
{
　　return new AccountIterator();
}

@Test(dataProvider = "generate-accounts-lazy")
public void testAccount(Account a)
{
　　System.out.println("Testing account " + a);
}

class AccountIterator implements Iterator
{
　　private static final int MAX = 4;
　　private int index = 0;

　　public boolean hasNext()
　　{
　　　　return index < MAX;
　　}

　　public Object next()
　　{
　　　　return new Object[] { new Account(index++); }
　　}

　　public void remove()
　　{
　　　　throw new UnsupportedOperationException();
　　}
}
```

如果传递的参数是简单类型的常数，利用testng.xml的方法是很好的。档我们需要更多灵活性，并知道参数的数目和值将随时间增加时，我们可能应该选择@DataProvider。


提供数据
数据的位置可能是：硬编码在Java源码中、再文本文件中、在属性文件中、在Excel表格中、在数据库中、在网络中...。

数据提供者还是工厂
数据提供者向测试方法传递参数，而工厂像构造方法传递参数。

不如不能确定使用哪种方法，那么就看看测试方法所使用的参数。是否有几个测试方法需要接收同样的参数？如果是这样，您可能最好是将这些参数保存在一个字段中，然后在几个方法中复用这个字段，这就意味着最好是选择工厂。反之，如果所有的测试方法都需要传入不同的参数，那么数据提供者可能是最好的选择。



异步测试
异步代码通常出现在下列领域：

- 基于消息的框架，其中发送者和接收者是解耦合的。(JMS)
- 由java.util.concurrent提供的异步机制(FutureTask)
- 由SWT或Swing这样的工具集开发的图形用户界面，其中代码与主要的图形部分运行在不同的线程中。


测试异步代码比测试同步代码的问题更多：

- 无法确定异步调用何时质性。
- 无法确定异步调用是否会完成。


当调用异步时有三种可能的结果：

- 调用完成并成功。
- 调用完成并失败。
- 调用没有完成。
基本上，异步编程遵循着一种非常简单的模式：在发出一个请求时指定一个对象或一个函数，当收到响应时系统会调用回调。

测试异步代码也同样遵循下面的模式：

发出异步调用，他会立即返回。如果可能，制定一个回调对象。



如果有回调方法：

等待结果，在接到结果是设置布尔变量，反应结果是否是您的预期。

在测试方法中，监视那个布尔变量，等到他被设置或过了一段时间。

如果没有回调方法：

在测试方法中，定期检查预期的值。如果过了一段时间还没有检查到预期值，就失败并退出。

不指定回调方法

```java
private volatile boolean success = false;

@BeforeClass
public void sendMessage()
{
　　// send the message;
　　// Successful completion should eventually set success to true;
}

@Test(timeOut = 10000)
public void waitForAnswer()
{
　　while (!success) {
　　　　Thead.sleep(1000);
　　}
}
```

在这个测试中，消息是作为测试初始化的一部分，利用@BeforeClass发出的，这保证了这段代码在测试方法调用之前执行并且只执行一次。在初始化后TestNG将调用waitForAswer测试方法，他将进行不完全忙等。

有回调方法：

```java
@Test(groups = “send”)
public void sendMessage()
{
　　// send the message
}

@Test(timeOut = 10000, dependsOnGroups = {“send”})
public void waitForAnswer()
{
　　while (!success) {
　　　　Thread.sleep(1000);
　　}
}
```

现在sendMessage()是一个@Test方法，他将包含在最终的报告中，如果发送消息失败，TestNG将跳过waitForAnswer测试方法，并把他表示为SKIP。

```java
@Test(timeOut = 10000, invocationCount=100, successPercentage = 98)
public void waitForAnswer ……
```
TestNG调用该方法100次，如果98%的调用成功，就认为总体测试通过。

测试多线程代码
并发测试

```java
private Singleton singleton;

@Test(invocationCount = 100, threadPoolSize = 10)
public void testSingleton()
{
　　Thread.yield();
　　Singleton p = Singleton.getInstance();
}

public static Singleton getInstance()
{
　　if (instance == null) {
　　　　Thread.yield();
　　　　Assert.assertNull(instance);
　　　　instance = new Singleton();
　　}
　　return instance;
}
```

@invocationCount相当简单，在不考虑并发时也可以使用：他决定了TestNG调用一个测试方法的次数。

@threadPoolSize要求TestNG分配一定数量的线程，并使用这些线程来调用这个测试方法，当一个测试完成之后，执行他的线程将归还给线程池，然后可以用于下一次调用。

并发执行
​    <suite name=”TestNG JDK 1.5” verbose=“1” parallel=“methods” thread-count = “2”>......</suite>
thread-count属性指定了线程数目，TestNG将使用这些线程来执行这个测试套件中的所有测试方法，parallel属性告诉TestNG您在执行这些测试时希望采用的并行模式。

parallel=”methods” 在这种模式下，每个测试方法将在他自己的一个线程中执行。

parallel=”test” 在这种模式下，在某个<test>标签内的所有测试方法将在他们自己的一个线程中执行。

在tests模式中，TestNG保证每个<test>将在他自己的线程中执行。如果希望测试不安全的代码，这一点是非常重要的。在method模式中，所有限制都被取消，无法预测哪些方法将在同一个线程中执行，哪些方法将在不同的测试中执行。

为可模拟性设计
为了能够成功地使用模拟模拟对象或桩，重要得失要确保代码的设计能让使用模拟对象或桩变得简单而直接。
这种设计最重要的方面就是正确的确定组件之间的交互，从而确定组件的交互接口。

如果我们有2个组件A和B，A需要用到B，那么应该通过B的接口来完成，而不是通过B的具体实现。

Singleton查找

```java
public void doWork1()
{
　　C c = C.getInstance();
　　c.doSomething();
}
```
对于某个对象智能由一个实例，这在项目生命周期的后期产生阻碍效果。

JNDI定位服务

```java
public void doWork2()
{
　　C c = (C) new InitialContext().lockup("C");
　　c.Something();
}
```
不能够向A提供一个受控制的B的实例。只有一个全局实例，A只能取得这个实例。


依赖注入

```java
private C c;

public void setC(C c)
{
　　this.c = c;
}
```

从外部通知A应该使用哪个B的实例。这让我们能够根据实际情况灵活地决定向A提供B的哪个实例。

EasyMock

```java
import static org.easymock.EasyMock.*;

public class EasyMockUserManagerTest
{
　　@Test
　　public void createUser()
　　{
　　　　UserManager manager = new UserManagerImpl();
　　　　UserDao dao = createMock(UserDao.class);
　　　　Mailer mailer = createMock(Mailer.class);

　　　　manager.setDao(dao);
　　　　manager.setMailer(mailer);

　　　　expect(dao.saveUser("tester")).andReturn(true);
　　　　expect(mailer.sendMail(eq("tester"), (String) notNull(), (String) notNull())).addReturn(true);

　　　　replay(dao, mailer);

　　　　manager.createUser("tester");
　　　　verify(mailer, dao);
　　}
}
```

1创建模拟对象
这是通过createMock方法完成的，传入希望模拟的类作为参数。

2纪录预期行为
只要在模拟对象上调用我们预期会被调用的方法，就能纪录预期的行为。当用到某些具体的参数时，只要将这些参数传入就可以了。

3调用主要被测对象
在主要的被测对象上调用一个方法或一组方法，预期这次调用将倒置被测对象调用模拟对象的那些预期的方法。

4验证预期行为
最后调用verify，检查所有的模拟对象。

JMock
jMock是一个模拟库，她让我们通过编成的方式来之行约束条件。


选择正确的策略

缺少接口
有时候，我们面对的是庞大臃肿的遗留系统，没有向期望的那样有很好的设计。
大多数模拟库现在都允许替换类，而不仅是接口。这些库会在运行时刻生成一个新类，通过字节码操作来实现指定的契约。

复杂的类
如果我们得到了一些类，他们拥有20多个方法，与许多其他组件交互，而且随着是间的推移变得越来越复杂。
这种情况下，使用动态的模拟对象库效果会比较好，因为他们能够定义单个方法的行为，而不是考虑所有的方法。

契约纪录
使用模拟对象让我们记录更多的契约信息，而不止是方法签名。我们可以随时验证器乐的。

测试目标
根据经验法则，如果希望测试组件之间交互，模拟对象可能优于桩对象。模拟库能够以一种准确的方式来指定交互。而桩该作为被测试组件使用的那些次要的组件。在这种情况下，测试的目标是测试被测试组件本身，而不是他与其他组件之间的交互。

模拟易犯的错误
依赖模拟对象会导至许多问题，所以重要的是要知道使用模拟对象不利的一面：

- 模拟外部API行
- 虚假的安全感
- 维护开销
- 继承与复杂性

依赖的测试
层叠失败：一个测试的失败导致一组测试的失败。

依赖的代码
只要测时方法依赖于其他测试方法，就很难以隔离的方式执行这些测试方法。
彼此依赖的测试方法通常会出现这样的情况，因为他们共享了一些状态，而在测试之间共享状态是不好的。


利用TestNG进行依赖的测试
TestNG通过@Test annotation的两个属性(dependsOnGroups和dependsOnMethods)赖支持依赖的测试。


```java
@Test
public void launchServer() {}

@Test(dependsOnMethods = "launchServer")
public void deploy() {}

@Test(dependsOnMethods = "deploy")
public void test1() {}

@Test(dependsOnMethods = "deploy")
public void test2() {}
```

dependsOnMethods的问题:
通过字符串来执行方法名称，如果将来对他进行重构，代码就有可能失效。方法名称违反了"不要重复自己"的原则，方法名称即在Java方法中用到，也在字符串中使用，另外，等我们不断添加新的测试方法时，这个测试用例伸缩性也不好。

```java
@Test(groups = "init")
public void launchServer() {}

@Test(dependsOnGroups = "init", groups = "deploy-apps")
public void deploy() {}

@Test(dependsOnGroups = "init", groups = "deploy-apps")
public void deployAuthenticationServer() {}

@Test(dependsOnGroups = "deploy-apps")
public void test1() {}

@Test(dependsOnGroups = "deploy-apps")
public void test2() {}
```

利用组来指定依赖关系可以解决我们遇到的所有问题：

- 不在遇到重构问题，可以任意秀该方法的名称。
- 不再违反DRY原则。
- 当新方法须要加入到依赖关系中时，只要将他放到适当的组中，并确保他依赖于正确的组。

依赖的测试和线程
当打算并行执行测试时，要记住，线程池中的一个或多个线程将用于依次执行每个方法。所以，如果打算在不同的线程中执行一些测试，过渡的使用依赖的测试将影像执行性能。

配置方法的失败
依赖测试方法和配置方法之间唯一的不同就是，测试方法隐式的依赖于配置方法。

虽然dependsOnMethods可以处理简单的测试或之由一个测试方法依赖于另一个测试方法的情况，但是在大多数情况下，您都应该使用dependsOnGroups，这种方式的伸缩性好，面对将来的重构也更为健壮。

既然我们提供了准去的依赖信息，那么TestNG就能够按照于骑的顺序来执行测试。

测试隔离并没有因此而受到影响。

如果出现层叠式的错误，依赖测试可以加快测试执行速读。


继承和annotation范围

```java
public class CreditCardTest
{
　　@Test(groups = "web.credit-card")
　　public void test1() {}

　　@Test(groups = "web.credit-card")
　　public void test2() {}
}
```

他违反了"不要重复自己"的原则
他为将来添加测试方法的开发者带来了负担。

```java
@Target({METHOD, TYPE, CONSTRUCTOR})
public @interface Test{}
@Test(groups = "web.credit-card")
public class CreditCardTest
{
　　public void test1() {}
　　public void test2() {}
}
```

annotation继承


```java
@Test(groups = "web.credit-card")
class BaseWebTest {}

public class WebTest extends BaseWebTest
{
　　public test1() {}
　　public test2() {}
}
```


所有扩展自BaseWebTest的类都会看到，他们所有的工有方法都自动成为web.credit-card组的成员。
WebTest变成了一个普通的传统Java对象(POJO),不带任何annotation。

集成易犯的错误
由于TestNG的测试方法必须是公有的，在基类中声明的方法会自动在子类中可见，所以他们永远也不需要作为测试类显式的列出(不要将测试基类列在testng.xml文件中)

测试分组
分组解决了上面提到的局限性，实际上，他们进一步提升了TesgNG的一个设计目标：在静态模型(测试代码)和动态模型(执行哪些测试)之间实现清晰的分离。

语法
@Test annotation和配置annotation(@BeforeClass, @AfterClass, @BeforeMethod...)都可以属于分组

```java
@Test(groups = {"group1"})
@Test(groups = {"group1", "group2"})
@Test(groups = "group1")

@Test(groups = "group2")
public class B
{
　　@Test
　　public test1() {}

　　@Test(groups = "group3")
　　public test2() {}
}
```

test1属于group2组，test2同时属于group2组和group3组


分组与运行时刻

```xml
<suite name="Simple suite">
　　<test name="GroupTest">
　　　　<groups>
　　　　　　<run>
　　　　　　　　<include name="group1" />
　　　　　　</run>
　　　　</groups>
　　　　<classes>
　　　　　　<class name="com.example.A" />
　　　　</classes>
　　</test>
</suite>
```
这个testng.xml告诉TestNG执行com.example.A类中所有属于group1组的测试方法。

```xml
<include name="database" />
<exclude name="gui" />
```

如果某个方法即属于包含的组，又属于排除的组，那么排除的组优先。
如果既没有include，也没有exclude，那么TestNG将忽略组，执行所有的测试方法。

另一个功能就是可以在testng.xml中利用正则表达式来指定组。


```xml
<groups>
　　<define name="all-web">
　　　　<include name="jsp" />
　　　　<include name="servlet" />
　　</define>
　　<run>
　　　　<include name="all-web">
　　</run>
</groups>
```

在设计组的层次关系时，能够在testng.xml中定义新组带来灵活性：
可以在代码中使用粒度非常小的分组，然后在运行时刻将这些小分组合并成大分组。

执行分组
利用命令行执行

```java
java org.testng.TestNG -groups jsp -groups servlet -excludegroups broken com.example.MytestClass
```

利用ant

```xml
<testng groups="jsp, servlet" excludegroups="broken">
　　<classfileset>
　　　　<include name="com/example/MyTestClass.class" />
　　</classfileset>
</testng>
```

利用Maven

```xml
<dependencies>
　　<dependency>
　　　　<groupId>org.testng</groupId>
　　　　<artifactId>testng</artifactId>
　　　　<version>5.10</version>
　　　　<classifier>jdk15</classifier>
　　</dependency>
</dependencies>

<build>
　　<plugins>
　　　　<plugin>
　　　　　　<groupId>org.apache.maven.plugins</groupId>
　　　　　　<artifactId>maven-surefire-plugin</artifactId>
　　　　　　<version>2.5</version>

　　　　　　<configuration>
　　　　　　　　<suiteXmlFiles>
　　　　　　　　　　<suiteXmlFile>testng.xml</suiteXmlFile>
　　　　　　　　<suiteXmlFiles>
　　　　　　</configuration>
　　　　</plugin>
　　</plugins>
</build>
```


利用Java API

```java
TestNG tng = new TestNG();
tng.setGroups("jsp, servlet");
tng.setExcludeGroups("broken")
```

排除失败的测试
创建一个特书的组如broken

```java
@Test(groups = { "web", "broken"})
```
然后在运行时刻排除这个组。

```xml
<exclude name="broken" />
```


组分类

测试类型:单元测试、继承测试
测试规模:小规模、大规模
功能描述:数据库、界面
测试速度:慢测试、快测试
过程描述:冒烟测试、发布测试

让开发者能够指定方法的分组，主要的好处在于开发者因此能够很容易找出他们需要执行哪些测试。(如刚刚修改了数据库代码，可能只需要执行fast和database组测试)

组命名

```java
@Test(groups = {"os.linux.debian"})
@Test(groups = {"database.table.ACCOUNTS"})
@Test(groups = {"database.ejb3.connection"})
```

TestNG能够利用正则表达式来之定要执行的组，如果与这项功能配合使用，这种命名方式就很有用了。

```xml
<groups>
　　<run>
　　　　<include name="database.*" />
　　</run>
</groups>
```

代码覆盖率
类的覆盖率：类覆盖描熟了项目中多少类已被测试套件访问。　
方法覆盖率：方法覆盖率是被访问的方法的百分比。
语句覆盖率：语句覆盖率追踪单条源代码语句的调用。
语句块覆盖率：语句快覆盖率将语句块作为基本的覆盖律单元。
分支覆盖率：分支覆盖率也被称为判断覆盖率。指标计算哪些代码分支被执行。

覆盖律工具
Clover、EMMA和Cobertura

成功使用覆盖率的建议
覆盖率报告的信息何音的解读不同
覆盖率很难
百分比没有意义
为覆盖率而设计是错误得
有一点好过没有
覆盖律工具不会测试不存在的代码
覆盖率的历史讲述了自己的故事

企业级测试
单元测试：单元测试对系统中的一个单元进行独立的测试。

功能测试：功能测试关注一项功能。通常涉及不同组件之间的交互。

继承测试：继承测试是一种端到端的测试，他会执行整个应用栈，包括所有的外部依赖关系或系统。

一个具体的例子
系统中有一个相当典型的组件，他接收一条JMS消息，其中包含一段有效的XML文本。这段XML文本相当长，描述了一笔财务交易。这个组件的工作是读出这条消息，解析XML，根据消息的内容条填充一些数据库的表，然后调用一个存储过程来处理这些表。

测试内容

我们将创建一个成功测试。希望确保，如果收到一条有效的XML消息，我们会正确地处理他，并更新正确的数据表，然后存储过程的调用也成功。
我们将摹拟不同的场景。希望能够为测试替工不同的XML文本，这样就能够很容易地不断添加测试粒子数据。
我们提供明确的失败测试。失败的行为将被记录和测试，这样当组件内部出现失败时，他的状态就可以与测，并且很容易记录下来。

非测试内容

我们不测试JMS provider的功能。假定他是一个完全兼容的实现，已经正确的进行了配置，将成功地提交我们期望的消息。
我们不执行捕捉所有错误得测试。失败测试应该针对明确的、可重现的失败场景。
我们不测试API。例如，JDBC去冬的行为不是测试的主题。确保所有的测试都贯注业务功能，要避免对Java语言的语义进行测试。

测试成功场景
对于JMS API的接口
利用模拟对象(或桩)对象，创建TextMessage实现，用一个简单的POJO来表现，带有消息内容和其他属性的设置方法。
重构该组件，解除业务功能与JMS API的耦合。

```java
public void onMessage(Message message)
{
    TextMEssage tm = (TextMessage) message;
    processDocument(tm.getText());
}

public void processDocument(String xml)
{
    // code previously in onMessage that updates DB
}

@Test
public void componentUpdateDatabase() throws Exception {}
```


构件测试数据

```java
@Test(dataProvider = "")
public void componentUpdateDatabase() throws Exception  {}

@DataProvider(name = "component-data-files")
public Iterator<Object[]> loadXML() throws Exception {}
```

我们的测试现在接受一个参数，不再需要自己考虑测试数据的来源，也不需要考虑如何加载测试数据。他要做的只是指定数据提供者。加载XML的实际工作现在代理给了一段独立的加载程序。

```java
@DataProvider(name = "component-data-files")
public Iterator<Object[]> loadXML() throws Exception
{
    File[] f = new File("filepath").listFiles();
    final Iterator<File> files = Arrays.asList(f).iterator();

    return new Iterator<Object[]>() {
        public boolean hasNext()
        {
            return files.hasNext();
        }

        public Object[] next()
        {
            return new Object[] { IOUtils.readFile(files.next()) };
        }

        public void remove()
        {
            throw new UnsupportedOperationException();
        }
    };
}
```

当然，可以从数据提供者返回一个Object[]的数组，但是，这种方法意味着我们必需将所有的文件的数据都一次性加载到内存中，因为数组必须事先填充。


测试准备问题

幂等的测试是指，这个测试执行一次和执行多次的结果是一样的。如果某个东西是幂等的，那么说明他在多次调用时状态不会改变。

不仅需要测试是幂等的，而且测试的次序应该无关紧要。所以除了需要是是幂等的之外，测试不应该在状态或数据方面影像其他测试。

对于一些写操作，成功执行之后很容易会对再次执行产生影响，下面方法有助于我们对付这个问题：

嵌入式数据
​    有一些基于Java的数据库引擎，在设计时专门考虑了嵌入式支持。这些数据库可以在测试过程中临时创建并进行初始化。他们开销很小，通常性能不错。
​    不足之处在于，它们与应用程序实际执行的环境差别非常大。通常在数据库特征上存在巨大的诧异。

在测试准备时初始化数据
​    测试数据库总加载一定数量的已知测试数据。其中包含希望操作的所有数据，以及组件所依赖的所有外部引用。
​    不足之处在于，很难维护一个健壮的数据集，使他对测试有足够的意义。

事务回滚
​    另一种方法就是利用Java API来防止数据写入到持久数据存储中。总的方法是开是一个事务，执行所有的写操作，验证一切正常，然后让事务回滚。
​    不足之处在于，如果没有复杂的嵌套事务准备，就不能测试参与事务的代码或开始自己的事务的代码。


选择正确的策略


```java
private WrappedConnection wrappedConnection;

@BeforeMethod
public void connection() throws SQLException
{
    connection = DatabaseHelper.getConnection();
    connection.setAutoCommit(false);
    wrappedConnection = new WrappedConnection(connection);
    wrappedConnection.setSuppressCommit(true);
}

@AfterMethod
public void rollback() throws SQLException
{
    wrappedConnection.rollback();
}

public class WrappedConnection implements Connection
{
    private Connection connection;
    private boolean suppressClose;
    private boolean suppressCommit;

    public WrappedConnection(Connection connection)
    {
        this.connection = connection;
    }

    public void commit() throws SQLException
    {
        if (!suppressCommit)
            connection.commit();
    }

    // ......
}
```


错误处理



```java
@Test(dataProvider = "component-data-files")
public void componentupdateDatabase(String xml) throws Exception
{
    Component component = new Component();
    try {
        component.processDocument(xml);
    }
    catch (InvalidTradeException e) {
        return;
    }
    // rest of test code
}
```

这种方法在于，他们没能让我们区分失败是否是预期的。相反，我们应该能够区分预期的成功和预期的失败。这个测时目前在两种情况下会通过：要么遇到好的数据时会通过，要么遇到坏数据时会通过。在每种情况下，我们都不能确定会发生些什么。

一个测试不应该在两种或两种以上的情况下都通过。如果测试验证了不同的失败情况，这没问题，但如果测试在好数据和坏数据的情况下都通过，那就会导致一些微妙的错误，这类错误难以被发现。(因此我们定义了另一个目录和数据提供者来处理失败的情况)

```java
@Test(dataProvider = "component-invalid-data-files", expectedException = InvalidTradeException.class)
public void componentInvalidInput(String xml) throws Exception
{
    Component component = new Component();
    component.processDocument(xml);
    // rest of test code
}
```
逐渐出现的单元测试
单元测试不一定是在其它测试之前编写的，他们可以是功能测试驱动的。特别是对于大型项目或原有的代码来说，一开始就编写有用的单元测试可能很困难，因为在不了解全局的情况下，单元测试可能太琐碎或不太重要。相反，单元测试可以从有意义的继承测试中推导出来，因为调试开发功能测试和集成测试的过程揭示他们所需的单元测试。

对于例子来说我们需要将XML验证与数据库处理放到各自独立的方法中。这样就能对他们进行测试

```java
public void processDocument(String xml) throws InvalidDocumentException
{
    Document doc = XMLHelper.parseDocument(xml);
    validateDocument(doc);
    // ......
}

public void validateDocument(Document doc) throws InvalidDocumentException
{
    // perform constraint checks that can't be captured by XML
}
```

这测重构的结果是我们得到了一个简单的单元测试。

不论测试编写的次序如何，功能测试和单元测试都是互不的。功能测试是更为水平化的测试，涉及许多不同的组件，执行代码的很多部分。相反，单元测试是更为垂直化的测试，他关注范围狭窄的主题，比功能测试要彻底得多。

竞争消费者模式
消费者的执行是并发的，所以我们必须在测试中进行某种程度的模拟，生产环境中的真实情况。在我们这样作了之后，也希望验证结果。不论哪个消费者先开始，也不论哪个消费者先结束，都没有关系。我们希望确定对于给定数量的消费者，我们将得道一组已知的结果，可以进行验证。

```java
private final List<Object[]> data = Collections.synchronizedList(new ArrayList<Object[]>());

@BeforeClass
public void populateData()
{
    data.add(new Object[] {"value1", "prop1"});
    data.add(new Object[] {"value2", "prop2"});
    data.add(new Object[] {"value3", "prop3"});
}

@Test(threadPoolSize = 3, invocationCount = 3, dataProvider = "concurrent-processor")
public void runConcurrentProcessors(String value, String someProp)
{
    MessageProcessor processor = new MessageProcessor();
    processor.process(value, someProp);
}

@Test(dependsOnMethods = "runConcurrentProcessors")
public void verifyConcurrentProcessors()
{
    // load data from db
    // verify that we have 3 results
    // verify that each of the 3 result matches our 3 input
}

@DataProvider(name = "concurrent-processor")
public Object[][] getProcessorData()
{
    return new Object[][] {data.remove(data.size() - 1)};
}
```

我们的测试被分成两个，一个负责执行消费者，另一个负责验证结果。原因是runConcurrentProcessors会被调用多次，而我们只需要在所有方法调用完成之后，对结果验证一次。为了表示这种次序，我们利用了dependsOnMethods这个annotation属性。

当TestNG发现一个数据提供者时，他将针对数据提供者返回的每一条数据调用一次测试。类似的，当我们指定调用次数时，TestNG会按照指定的次数调用测试。因此，如果我们返回数据提供者中准备好的3条数据，那么每个线程都会执行3次测试。

因此解决方案是使用一个栈结构，每次调用数据提供者时，返回一条数据，并将这条数据从列表中清除。数据提供者将被调用3次，每次都将为数据返回不一样的数据。


原则：将数据的考虑和功能的考虑分开来是很关键的。
在这个例子中，消费者需要的数据应该和实际的测试没有依赖关系。这种方法意味着，随着我们对数据的需求不断变化，变得更为复杂，测试本身却不需要被修改。


一个具体的例子
我们希望测试一个登录servlet。这个servlet接受一个请求，检查用户名和口令，如果他们有效，就在会话中加入一个标记。表明用户已登录。

这个例子展示了重构在测试方面起到的重要辅助作用，说明了即使对于看上去很麻烦、需要一很复杂地方是进行交互的API，页可以通过抽象去掉依赖关系。这种抽象意味着 在测试过程中，我们可以利用更简单的对象，这些对象更容易构造，因此也更容易测试。

增加可测试性的一个副作用就是改进了设计。为提高可测试性而进行重构，可以帮助我们以一种实际的、代码中的方式来确定职责和考虑，而这种效果通过画设计图是很难达到的。

Java EE测试
容器内测试与容器外测试的对比

容器内测试
优点：
​    完全符合运行时环境

缺点：
​    启动消耗大
​    难以部署新的测试
​    难以自动化
​    夸平台测试的复杂性增加

容器外测试
优点：
​    提供了相对较快的启动
​    可以完全控制环境
​    可以自动化
​    容易测试
缺点：
​    没有符合运行时环境
​    测试所用的实现可能与运行时的实现来自不同的提供商。


容器内测试
测试步骤：

创建一个测试环境的实例。
确定测试。
在测试框架中注册测试。
注册一个监听者来接收搜测试结果。


创建测试环境

```java
TestNG tester = new TestNG();
tester.setDeafultSuiteName("container-tests");
```

确定测试
假定所有的测试类都在WEB-INF/classes目录下，我们可以递归地读入这个目录，找到其中所有的类文件。

```java
public static Class[] scan(ServletContext context)
{
    String root = context.getReadPath();
    ClassScanner scanner = new ClassScanner(new File(root));
    scanner.setClassLoader(ClassScanner.class.getClassLoader());
    final File testDir = new File(root, "com/foo/tests");
    scanner.setFilter(new FileFilter() {
        public boolean accept(File pathname) {
            return pathname.getPath().startsWith(testDir.getPath());
        }
    });
    Class[] classes = scanner.getClasses();
    return classes;
}
```

context是一个ServletContext实例，他是通过调用servlet或JSP页面得到的。

注册测试
注册测试类的动作告诉了TestNG他要查看的一组类，TestNG将在这组类中查找需要执行哪些测试。他将检查每个指定的类，确定他是否包涵测试方法或配置方法。当所有类都检查过后，TestNG内部会生成一个依赖关系图，以决定照到的这些测试的执行次序。

```java
tester.setTestClasses(classes);
```

注册结果监听者

TestNG自代了3个默认的报告类：

SuiteHTMLRepoter 默认报告类，他在一个目录下输出交叉引用HTML文件，让您能看到某个具体测试的结果。

FailedReporter: 这个报高生成一个TestNG执行配置，该配置包含了所有前一次运行时失败的测试。他也是默认运行的。

EmailableReporter: 这个报告类生成一个报告文件，可以很容易地通过电子邮件发送，显示测试的结果。


默认情况下，EmailableReporter在磁盘上生成一个文件。


```java
public class SinglePageReporter extends EmailableReporter
{
    private Writer writer;

    public SinglePageReporter()
    {
        this.writer = writer;
    }

    protected PrintWriter createWriter(String out)
    {
        return new PrintWriter(writer);
    }
}
```


调用TestNG的JSP页面


```jsp
<%@ page import="org.testng.*, java.io.*" %>
<%
    TestNG tester = new TestNG();
    tester.setDefaultSuiteName("container-tests");

    String root = application.getRealPath("/WEB-INF/classes");
    ClassScanner scanner = new ClassScanner(new File(root));
    scanner.setLoader(getClass().getClassLoader());
    scanner.setFilter(new FileFilter() {
        public boolean accept(File pathname) {
            return pathname.getPath().indexOf(Test) > -1;
        }
    });

    Class[] classes = scanner.getClasses();
    tester.setTestClasses(classes);

    IReporter reporter = new SinglePageReporter(out);
    tester.addListener(reporter);

    tester.run();
%>
```

Java命名和目录接(JNDI)
JNDI是一个在全局目录中查找资源的API。可以把他看成是一个很大的树型结构，我们在其中按照名称查找某个节点。

```java
new InitialContext().lockup("someObject");
```
上面创建一个InitialContext对象，如果在执行容器内执行，会利用供应商提供的API实现，来查找容器内部命名目录结构。创建了上下文之后，我们在其中查找对象，列出他的内容，或遍历这棵树。所有这些都是通过JNDI API来完成的。InitialContext的构造方法由一个重载的版本，接受一个Hashtable对象作为参数，其中包含不同的环境变量值，决定了上下文应该如何创建。

```java
Hashtable env = new Hashtable();
env.put(Context.INITIAL_CONTEXT_FACTORY, "");
env.put(Context.PROVIDER_URL, "smqp://localhost:4001");
Object topic = new InitialContext(env).lookup("myTopic");
```

避免JNDI
组件依赖关系要么通过服务定位(通常是JNDI)来实现，要么通过注入来实现。如果您可以选择，就采用注入的方式，因为这样测试的开销最小，并且这种方式带来了更大的灵活性。

Java消息服务(JMS)

```java
private Session session;
private Destination destination;

@BeforeClass(groups = "jms")
public void setupJMS() throws NamingException, JMSException
{
    Hashtable env = new Hashtable();
    // populate environmet for out specific provider
    InitialContext context = new InitialContext(env);

    ConnectionFactory factory = (ConnectionFactory) context.lookup("QueueConnectionFactory");
    Connection connection = factory.createConnection();
    session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
    destination = (Detination) context.lookup("TestQueue@router1");
}

@Test(groups = "jms")
public void sendMessage() throws JMSException
{
    TextMessage msg = session.createTextMessage();
    msg.setText("hello!");
    msg.setStringProperty("test", "test1");
    session.createProducer(destination).send(msg);
}

@Test(groups = "jms", dependsOnMethods = "sendMessage", timeOut = 1000)
public void receiveMessage() throws JMSException
{
    MessageConsumer consumer = session.createConsumer(destination, "test", "test1");
    TextMessage msg = (TextMessage) consumer.receive();
    assert "hello!".equals(msg.getText());
}
```



在测试中使用ActiveMQ

```java
@BeforeClass(groups = "jms")
public void setupActiveMQ() throws Exception
{
    BrokerService broker = new BrokerService();
    broker.setPersistent(false);
    broker.setUseJmx(false);
    broker.start();

    URI uri = broker.gettVmCnnectionURI();
    ConnectionFactory factory = new ActiveMQConnectionFactory(uri);
    Connection connection = factory.createConnection();
    connection.start();

    session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
    destination = session.createQueue("TestQueue@router1");
}
```



处理状态
在JMS的例子中，当我们拥有多个测试时，会引发一个有趣的问题。因为测试是由一对方法组成的，所以让我们假定同一个类中还有另一对发送/接收测试。

一种方法是将发送和接受者放在一个测试中，并在每个测试方法之前初始化消息代理。请注意两点都要做到，因为让消息代里在一对测试方法之前初始化是比较麻烦的。

另一种方法是使用消息分捡器。JMS消息分捡器让我们能够过滤接收到的消息，这样可以只接收与分捡器匹配的消息。

Spring
Spring的测试包功能

TestNG通过自己的一组针对Spring的类来解决这些问题，这些类作为扩展提供。org.testng.spring.test包中包含了所有Spring在其测试包中提供的类，这些类已修改过，可以用在基于TestNG的测试中。

AbstractSpringContextTests
这是所有Spring测试的基类。他的主要职责是提供上下文的管理。这个类包含一个静态map，其中包含所有注册的Spring上下文。

AbstractSingleSpringContextTests
这个类扩展了AbstractSpringContextTests，提供了装入一个ApplicationContext对象的钩子。他的子类应该实现getConfigLocation(String[] paths)方法。这个方法返回一个字符串数组，指出了Spring配置文件的位置，通常是从classpath加载的。


```java
import org.testng.annotation.Test;
import org.testng.spring.test.AbstractSingleSpringContextTests;

public class SpringContextTests extends AbstractSingleSpringContextTests
{
    protected String[] getConfigLocations()
    {
        return new String[] {"/spring.xml"};
    }

    @Test
    public void beanDefined()
    {
        assert applicationContext.getBean("myBean") != null;
    }
}
```


Spring的配置方法被声明在名为Spring-init的TestNG分组中。我们不必依赖于单个的onSetUp或ontearDown方法，可以根据需要声明任意多个@BeforeMethod/@AfterMethod配置方法，只要指定他们依赖于spring-init，就可以确保他们在Spring执行完准备工作之后得到调用。


AbstractDependencyInjectionSpringContextTests
这个类提供的最有趣的功能就是向测试注入依赖。测试依赖关系可以表现为设值方法或成员字断。测试也可以指定Spring应该对他们的树性执行哪种类型的自动编织。

```java
public class SpringInjectionTests extends AbstractDependncyInjectionSpringContextTests
{
    private PersonManager manager;

    protected String[] getConfigLocation()
    {
        return new String[] {"/spring.xml"};
    }

    public void setManager(PersonManager manager)
    {
        this.manager = manager;
    }

    @Test
    public void verifyManager()
    {
        assert manager != null;
    }
}
```

这个类有一个事务管理器属性。因为他派生自支持注入的类，配置文件中必需指定一个PlatforTransactionManager，以便注入。

```java
public class StrpingTranscationTests extends AbstractTransactionalSpringContextTests
{
    private PersonManager manager;

    public void setManager(PersonManager manager)
    {
        this.manager = manager
    }

    protected String[] getConfigLocation()
    {
        return new String[] {"/spring.xml"};
    }

    @Test
    public void savePerson()
    {
        Person p = new Person();
        manager.savePerson(p);

        assert p.getId() != null;
        // setComplete();
    }
}
```


我们没有在测试中指定任何事务行为，超类自动处理了事务方面的问题，让每个测试在他自己的事务中执行，这个十五将在该测试完成之后回滚。

增加的调用setComplete通知超类在测试执行之后应该提交这个事务，而不是回滚。调用这个方法由一个有趣的副作用：这个类中所有后续测试都会提交事务，而不是依赖于默认行为。

答案在于JUnit和TestNG之间的一个微妙区别。Spring测试假定采用了JUnit的语义。每个测试类对每个测试方法都会重新实例化。因此，所有的测试都假定每个测试开始时，测试时里的状态都会复原，但TestNG不是这样的。

AbstractTransactionalDataSouceSpringContextTests
这个类添加了一些JDBC相关的便利方法。

AbstractAnnotationAwareTransactionalTests
这个类支持超类提供的所有功能之外，这个类允许我们在测试方法上指定Spring特有的事务annotation，而不是通过编程的方式来指定事务行为。

Guice
第2章中的例子，对于每个接口，我们都有两个实现。一个是实际的产品实现，他会与一些外部的依赖关系进行交互，如UserDAO对象会与数据库交互，Mailer对象会与SMTP邮件服务器交互。我们还有桩对象实现。


```java
@Test
public void verifyCreateUser()
{
    UserManager manager = new UserManagerImpl();
    MailerStub mailer = new MailerStub();

    manager.setMailer(mailer);
    manager.setDao(new UserDAOStub());

    manager.createUser("tester");
    assert mailer.getMails().size() == 1;
}
```


Guice注入测试


```java
@Inject private UserManager manager;
@Inject private MailerStub mailer;

@Test
public void verifyCreateUser()
{
    manager.createUser("tester");
    assert mailer.getMails().size() == 1;
}
```



Spring注入测试


```java
private UserManager manager;
private MailerStub mailer;

public void verifyCreateUser()
{
    manager.createUser("tester");
    assert mailer.getMails().size() == 1;
}

public void setManager(UserManager manager)
{
    this.manager = manager;
}

public void setMailer(MailerStub mailer)
{
    this.mailer = mailer;
}
```


对象工厂

杂谈
关注和提供异常
一层遇到了一个没有预料到的错误，不知道如何处理。所以这一层就快乐地向上一层抛出一个异常，希望这个可怜的异常最终会遇到知道怎么作的人。


吞掉抛出异常

```java
try {
    callBackend();
}
catch (SQLException ex) {
    throw new BackendException("Error in backed");
}
```
这种方法问题在于，实际上我们丢试了真实错误的有意义的信息。当我们最后有机会处理这个异常的时候，我们得到的信息仅仅是某一层中出了问题，但我们不知道是在这一曾本身出了问题，还是更低的层出了问题。


记日志并抛出


```java
try {
    callBackend();
}
catch (SQLException ex) {
    log.error("Error calling backend", ex);
    throw ex;
}
```

问题在于调用栈经常发生的情况：信息隐藏。假定一个应用程序有三层或四层，每一层都在日志中记录他处里的异常，查遍8页的调用栈信息并不是最有效地方式。


嵌套抛出

```java
try {
    callBackend();
}
catch (SQLException ex) {
    throw new BackendException("Error in backend", ex);
}
```
当这个调用栈显示的时候，没有丝毫暗示表明背后的原因是什么。您必须编写一个帮助方法，从外面的包装中取出背后实际的异常。


我们建议两种解决方案

避免需检察异常。运行时异常很合适这样的情况。
包装异常。假如您非常肯定在调用栈打印时，背后的原因会显示出来，那么包装的异常也很好。


有状态测试
有两种截然不同的状态分类：不可修改的状态和可修改的状态

不可修改的状态
访问共享的不可修改的状态的测试方法，相互之间是独立的。
因为这些方法都不能修改他们读到的状态，所以调用顺序可以是任意的，因此他们没有违反测试访法应该彼此肚里的原则。

可修改的状态


```java
public class MyTest extends TestCase
{
    private int count = 0;

    public void test1()
    {
        count++;
        assertEquals(1, count);
    }

    public void test2()
    {
        count++;
        assertEquals(2, count);
    }
}
```

JUnit会让这个测试通过，但TestNG不会。
只有当您知道测试方法被调用的顺序时，共享可修改的状态才有意义。

安全的共享数
安全          不可修改的状态

安全　　　可修改的状态与完全指定的依赖关系

不安全　　可修改的状态与不确定的依赖关系


测试驱动开发的缺点
他注重微观设计超过宏观测试

他在实践中难以应用

TDD注重微观设计超过宏观设计

测试驱动开发可以得到更健壮的软件，但他也可能导致不必要的反复和过度重构的趋势，这可能对饮的软件、设计和最后期限产生负面影响。

TDD难以应用

这种方式得到的代码本身并不会优先于传统方法测试的代码。

如何创建测试并不重要，重要的是确实要创建测试

测试私有方法
如果他有可能出问题，您就应该测试他。

测试私有方法的比较好的方法是提升方法的可见性，例如，让他变成保护可见，或者包可见。后者更好一些，因为您可以把测试和被测类放在同一个包中，然后您就可以访问他的所有字段。如果这些方法都不可取，我们测试私有方法的方式就是利用反射。

```java

   public class LoginController
    {
        private String userName;

        private void init()
        {
            this.username = getUserNameFromCookie();
        }
    }
    
    @Test
    public void verifyInit()
    {
        LoginController lc = new LoginController();
    
        Field f = lc.getClass().getField("username");
        Object valueBefore = f.get(lc);
        Assert.assertNull(valueBefore);
    
        Method m = lc.getClass().getDeclaredMethod("init", null);
        m.setAccessible(true);
        m.invock(lc);
    
        Object valueAfter = f.get(lc);
        Assert.assertNotNull(vlaueAfter);
    }

```
我们利用字符串来描述Java元素。如果您对这些元素进行重命名，这种危险的实践肯定会失效。

我们利用了该类危险的私有信息。我们不仅假定存在一些私有方法和属性，而且也假定这个方法将以某种方式修改一个字段。


测试与封装
如果让代码更可测试，不要担心破坏封装。可测试星应该胜过封装。

让一个私有方法(或字段)成为包可见的、保护可见的或公有的。

去掉方法的final限定符。这让测试勒能够扩展这些类，或重写这些方法，模拟或稍微改变他们的实现，从而让系统的其它部分更可测试。


记日志的最佳实践
在出错时，输出错误或警告是合理的。但是对於警告的情况，重要的是确定这是不是该做的事情。我们添加的每一条无用的日志信息都会干扰有用的信息，所已精心选择是有意义的。

对于调试需求，记日志是有用的。但是，只要有一个开关能打开或关闭，绝大多数的记日志需求都能够满足了，不需要复杂的解决方案。

---

参考文档：

<http://www.cnblogs.com/rilley/archive/2012/11/09/2762818.html>