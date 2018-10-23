---
layout: post
title: "Mockito简介"
date: 2013-06-21 21:29
comments: true
categories: ut
tags: [ mockito, ut, powermock, stub, mock]
---

Mockito 是目前 java 单测中使用比较流行的 mock 工具。其他还有 EasyMock，JMock，MockCreator，Mockrunner，MockMaker 及 PowerMock。

项目地址:<https://code.google.com/p/mockito/>

## powermock 简介
EasyMock 以及 Mockito 都因为可以极大地简化单元测试的书写过程而被许多人应用在自己的工作中，但是这两种 Mock 工具都不可以实现对静态函数、构造函数、私有函数、Final 函数以及系统函数的模拟，但是这些方法往往是我们在大型系统中需要的功能。PowerMock 是在 EasyMock 以及 Mockito 基础上的扩展，通过定制类加载器等技术，PowerMock 实现了之前提到的所有模拟功能，使其成为大型系统上单元测试中的必备工具。缺点是缺少文档。

项目地址:<https://code.google.com/p/powermock/>
<!--more-->
## Mock 和Stub
### Mock
所谓的mock，即模拟，模仿的意思。Mock 技术的主要作用是使用mock工具模拟一些在应用中不容易构造或者比较复杂的对象，从而把测试目标与测试边界以外的对象隔离开。

### Stub
Stub,桩。单元测试过程中，对于在应用中不容易构造或者比较复杂的对象，用一个虚拟的对象来代替它。从类的实现方式上看，stub有一个显式的类实现，按照stub类的复用层次可以实现为普通类(被多个测试案例复用)，内部类(被同一个测试案例的多个测试方法复用)乃至内部匿名类(只用于当前测试方法)。stub的方法也会有具体的实现，哪怕简单到只有一个简单的return语句。

### Stub 与 Mock 的区别
Stub 是在单元测试过程中去代替某些对象来提供所需的测试数据，适用于基于状态的（state-based）测试，关注的是输入和输出。而Mock适用于基于交互的（interaction-based）测试，关注的是交互过程，不只是模拟状态，还能够模拟模块或对象的行为逻辑并能验证其正确性，Mock不需要类的显示实现，直接用工具模拟。

## Mockito 的使用

### Maven
通过Maven管理的，需要在项目的Pom.xml中增加如下的依赖：

	<dependencies>
	<dependency>
	<groupId>org.mockito</groupId>
	<artifactId>mockito-all</artifactId>
	<version>1.9.0</version>
	<scope>test</scope>
	</dependency>
	</dependencies>

在程序中可以`import org.mockito.Mockito`，然后调用它的static方法。

### Maven 程序
#### 1.模拟对象
创建 Mock 对象的语法为 mock(class or interface)。
#### 2.设置对象调用的预期返回值
通过 when(mock.someMethod()).thenReturn(value) 来设定 Mock 对象某个方法调用时的返回值。或者使用 when(mock.someMethod()).thenThrow(new RuntimeException) 的方式来设定当调用某个方法时抛出的异常。
#### 3.验证被测试类方法
Mock 对象一旦建立便会自动记录自己的交互行为，所以我们可以有选择的对它的 交互行为进行验证。在 Mockito 中验证 Mock 对象交互行为的方法是 verify(mock).someMethod(…)。最后 Assert() 验证返回值是否和预期一样。

### Demo

## Mock 对象的创建
	mock(Class<T> classToMock);
	mock(Class<T> classToMock, String name)
    mock(Class<T> classToMock, Answer defaultAnswer)
	mock(Class<T> classToMock, MockSettings mockSettings)
	mock(Class<T> classToMock, ReturnValues returnValues)

可以对类和接口进行mock对象的创建，创建时可以为mock对象命名。对mock对象命名的好处是调试的时候容易辨认mock对象。

## Mock对象的期望行为和返回值设定
假设我们创建了LinkedList类的mock对象：

	LinkedList mockedList = mock(LinkedList.class);

### 对方法进行设定返回值和异常
#### 对包含返回值的方法的设定

	when(mockedList.get(0)).thenReturn("first");
	when(mockedList.get(1)).thenThrow(new RuntimeException());

Mockito支持迭代风格的返回值设定

	when(mockedList.get(anyInt()).thenReturn("first").thenThrow(new RuntimeException());
	when(mockedList.get(anyInt()).thenReturn("first","second");

Stubbing的另一种风格

	doReturn("Hello").when(mockedList).get(0);
	doReturn("Hello").doReturn("world").when(mockedList).get(anyInt());

抛出异常

	doThrow(new RuntimeException()).when(mockedList).get(0);

#### 对void方法进行方法预期设定

	doNothing().when(mockedClass).SomeVoidMethod();
 	doThrow(new RuntimeException()).when(mockedClass).SomeVoidMethod();
迭代风格

	doNothing().doThrow(new RuntimeException()).when(mockedClass).SomeVoidMethod();

## 参数匹配器
在Stubbing和Verify的时候，有时需要更加灵活的参数需求。参数匹配器(Argument Matcher)能够满足需求。

	//stubbing using anyInt() argument matcher
	when(mockedList.get(anyInt())).thenReturn("element");

	//following prints "element"
	System.out.println(mockedList.get(999));

	//you can also verify using argument matcher
	verify(mockedList).get(anyInt());

需要注意的是，如果使用了参数匹配器，所有的参数都需要由匹配器提供。如下eq("third argument")，直接修改为“third argument”会报错。

	verify(mockedClass).someMethod(anyObject(), anyString(), eq("third argument"));

## Mock对象行为的验证
Mock 对象行为的验证，关注其交互行为，如mock对象调用的参数，调用次数，调用顺序等。

### 调用次数验证
	public static <T> T verify(T mock).someMethod()
	public static <T> T verify(T mock, VerificationMode mode).someMethod()

	Parameters:
		mock - to be verified
		mode - times(M), atLeastOnce() , atLeast(N) , atMost(X) , never()
	Returns:
		mock object itself

### 调用顺序验证
	public static InOrder inOrder(java.lang.Object... mocks)

创建mock对象

	// Multiple mocks that must be used in a particular order
	List firstMock = mock(List.class);
	List secondMock = mock(List.class);
调用mock对象的方法

	//using mocks
	firstMock.add("was called first");
	secondMock.add("was called second");
创建InOrder对象

	//create inOrder object passing any mocks that need to be verified in order
	InOrder inOrder = inOrder(firstMock, secondMock);

验证方法调用

	//following will make sure that firstMock was called before secondMock
	inOrder.verify(firstMock).add("was called first");
	inOrder.verify(secondMock).add("was called second");



## `RETURN_SMART_NULLS和RETURN_DEEP_STUBS`
`RETURN_SMART_NULLS` 是实现了Answer 接口的对象，它是创建mock对象时的一个可选参数， `mock(class,answer)`。在创建mock对象时，使用该参数，调用没有stubbed的方法会返回 SmartNull 。如返回类型为String的，会返回空"", int 会返回 0,List 会返回 null。

mock对象使用RETURN_DEEP_STUBS 参数，会自动mock该对象中包含的对象。

## 注解
Mockito支持对变量进行注解，如将mock对象设为测试类的属性，然后通过注解的方式@Mock来定义它，可以减少重复代码，增强可维护性。Mockito支持的注解有@Mock，@Spy，@Captor，@InjectMocks

### Annotation 的初始化
初始化方法为调用MockitoAnnotations.initMocks(testClass)，可以放到@Before中。

	public class ArticleManagerTest {

	    @Mock private ArticleCalculator calculator;
	    @Mock private ArticleDatabase database;
	    @Mock private UserProvider userProvider;

	 	@Before public void setup() {
			MockitoAnnotations.initMocks(testClass);
	    }
	}

使用Mockito提供的Junit Runner可以省略上述步骤。

	@RunWith(MockitoJUnitRunner.class)
	public class ExampleTest {
	    @Mock private List list;

	    @Test public void shouldDoSomething() {
	        list.add(100);
	    }
	}

## powermock 的使用
### Maven配置
	<dependency>
		<groupId>org.powermock</groupId>
		<artifactId>powermock-module-junit4</artifactId>
		<version>1.4.10</version>
		<scope>test</scope>
	</dependency>
	<dependency>
		<groupId>org.powermock</groupId>
		<artifactId>powermock-api-mockito</artifactId>
		<version>1.4.10</version>
		<scope>test</scope>
	</dependency>
## PowerMock 在单元测试中的应用
### 模拟 Static 方法
在任何需要用到 PowerMock 的类开始之前，首先我们要做如下声明：

	@RunWith(PowerMockRunner.class)

然后，还需要用注释的形式将需要测试的静态方法提供给 PowerMock：

	@PrepareForTest( { YourClassWithEgStaticMethod.class })

然后就可以开始写测试代码：

	1，首先，需要有一个含有 static 方法的代码 , 如
	public class IdGenerator {
	    ...
	    public static long generateNewId() {
	        ...
	    }
	    ...
	 }
	2，然后，在被测代码中，引用了以上方法
    public class ClassUnderTest {
    	...
    	public void methodToTest() {
        ..
        final long id = IdGenerator.generateNewId();
        ..
     	}
    	...
 	}

	3，为了达到单元测试的目的，需要让静态方法 generateNewId()返回各种值
	来达到对被测试方法 methodToTest()的覆盖测试，实现方式如下：

	 @RunWith(PowerMockRunner.class)
	 @PrepareForTest(IdGenerator.class)
	 public class MyTestClass {
	    @Test
	    public void demoStaticMethodMocking() throws Exception {
	        PowerMockito.mockStatic(IdGenerator.class);
	        when(IdGenerator.generateNewId()).thenReturn(2L);

	        new ClassUnderTest().methodToTest();

	        verifyStatic();
	        IdGenerator.generateNewId();
	    }
	 }

### 模拟构造函数
有时候，能模拟构造函数，从而使被测代码中 new 操作返回的对象可以被随意定制，会很大程度的提高单元测试的效率，考虑如下：

	public class DirectoryStructure {
	    public boolean create(String directoryPath) {
	        File directory = new File(directoryPath);

	        if (directory.exists()) {
	            throw new IllegalArgumentException(
	            "\"" + directoryPath + "\" already exists.");
	        }

	        return directory.mkdirs();
	    }
	 }

为了充分测试 create()函数，我们需要被 new 出来的 File 对象返回文件存在和不存在两种结果。在 PowerMock 出现之前，实现这个单元测试的方式通常都会需要在实际的文件系统中去创建对应的路径以及文件。然而，在 PowerMock 的帮助下，本函数的测试可以和实际的文件系统彻底独立开来：使用 PowerMock 来模拟 File 类的构造函数，使其返回指定的模拟 File 对象而不是实际的 File 对象，然后只需要通过修改指定的模拟 File 对象的实现，即可实现对被测试代码的覆盖测试，参考如下：

	 @RunWith(PowerMockRunner.class)
	 @PrepareForTest(DirectoryStructure.class)
	 public class DirectoryStructureTest {
	    @Test
	    public void createDirectoryStructureWhenPathDoesntExist()
		throws Exception {
	        final String directoryPath = "mocked path";

	        File directoryMock = mock(File.class);

	        //File的初始化函数的mock
	        whenNew(File.class).withArguments(directoryPath)
				.thenReturn(directoryMock);

	        // Standard expectations
	        when(directoryMock.exists()).thenReturn(false);
	        when(directoryMock.mkdirs()).thenReturn(true);

	        assertTrue(new NewFileExample()
				.createDirectoryStructure(directoryPath));

	        // Optionally verify that a new File was "created".
	        verifyNew(File.class).withArguments(directoryPath);
	    }
	 }
使用 whenNew().withArguments().thenReturn() 语句即可实现对具体类的构造函数的模拟操作。然后对于之前创建的模拟对象 directoryMock使用 When().thenReturn() 语句，即可实现需要的所有功能，从而实现对被测对象的覆盖测试。在本测试中，因为实际的模拟操作是在类 DirectoryStructureTest 中实现，所以需要指定的 @PrepareForTest 对象是 DirectoryStructureTest.class。

### 模拟私有以及 Final 方法
为了实现对类的私有方法或者是 Final 方法的模拟操作，需要 PowerMock 提供的另外一项技术：局部模拟。

在之前的介绍的模拟操作中，我们总是去模拟一整个类或者对象，然后使用 When().thenReturn()语句去指定其中值得关心的部分函数的返回值，从而达到搭建各种测试环境的目标。对于没有使用 When().thenReturn()方法指定的函数，系统会返回各种类型的默认值。

局部模拟则提供了另外一种方式，在使用局部模拟时，被创建出来的模拟对象依然是原系统对象，虽然可以使用方法 When().thenReturn()来指定某些具体方法的返回值，但是没有被用此函数修改过的函数依然按照系统原始类的方式来执行。

这种局部模拟的方式的强大之处在于，除开一般方法可以使用之外，Final 方法和私有方法一样可以使用。
参考如下所示的被测代码：

	 public final class PrivatePartialMockingExample {
	    public String methodToTest() {
	        return methodToMock("input");
	    }

	    private String methodToMock(String input) {
	        return "REAL VALUE = " + input;
	    }
	 }
为了保持单元测试的纯洁性，在测试方法 methodToTest()时，我们不希望受到私有函数 methodToMock()实现的干扰，为了达到这个目的，我们使用刚提到的局部模拟方法来实现 , 实现方式如下：

	 @RunWith(PowerMockRunner.class)
	 @PrepareForTest(PrivatePartialMockingExample.class)
	 public class PrivatePartialMockingExampleTest {
	    @Test
	    public void demoPrivateMethodMocking() throws Exception {
	        final String expected = "TEST VALUE";
	        final String nameOfMethodToMock = "methodToMock";
	        final String input = "input";

	        PrivatePartialMockingExample underTest = spy(new PrivatePartialMockingExample());

	        /*
	         * Setup the expectation to the private method using the method name
	         */
	        when(underTest, nameOfMethodToMock, input).thenReturn(expected);

	        assertEquals(expected, underTest.methodToTest());

	        // Optionally verify that the private method was actually called
	        verifyPrivate(underTest).invoke(nameOfMethodToMock, input);
	    }
	 }
可以发现，为了实现局部模拟操作，用来创建模拟对象的函数从 mock() 变成了 spy()，操作对象也从类本身变成了一个具体的对象。同时，When() 函数也使用了不同的版本：在模拟私有方法或者是 Final 方法时，When() 函数需要依次指定模拟对象、被指定的函数名字以及针对该函数的输入参数列表。

参考文献:<http://www.ibm.com/developerworks/cn/java/j-lo-powermock/>

---
Thanks to：lizejun
