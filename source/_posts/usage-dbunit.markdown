---
layout: post
title: "DbUnit使用入门"
date: 2013-06-27 15:57
comments: true
categories: 单元测试
tags: [ dbunit, h2database, ut，sql-maven-plugin, 数据库 ]
---
[DbUnit](http://www.dbunit.org/)是一个意在对使用数据库驱动项目进行测试的JUnit扩展。它使得你的数据库在各个Test之间处于一个已知的状态。这很好的解决了以下问题：当一个测试破坏了数据库时，导致其后面的所有测试失败或给出了错误的结果。

### DbUnit原理
DbUnit通过维护真实数据库与数据集(DataSet)之间的关系来发现与暴露测试过程中的问题。此处DataSet可以自建，可以由数据库导出，并以多种方式体现，xml文件、XLS文件和数据库查询数据等，一般多用XML文件。在测试过程中，DataSet被称为期望结果(expected result),真实数据库被称真实结果(actual result),你所要做的就是通过DbUnit完成期望结果与真实结果之间的操作与比较，从而发现问题和校验结果。
DbUnit包括三个核心部分:

- IDatabaseConnection ：描述DbUnit数据库连接接口；
- IDataSet：数据集操作接口；
- DatabaseOperation：描述测试用例测试方法执行前与执行后所做操作的抽象类；

<!--more-->
值得关注的是DatabaseOperation的各种实现，比较常用的有 REFRESH、DELETE_ALL和CLEAN_INSERT等。这些操作关系到数据集与数据库数据的同步、数据准备，不小心就会对数据库原有数据造成影响，所以务必做好备份。

DatabaseOperation有以下的可选项：

- NONE：不执行任何操作，是getTearDownOperation的默认返回值。
- UPDATE：将数据集中的内容更新到数据库中。它假设数据库中已经有对应的记录，否则将失败。
- INSERT：将数据集中的内容插入到数据库中。它假设数据库中没有对应的记录，否则将失败。
- REFRESH：将数据集中的内容刷新到数据库中。如果数据库有对应的记录，则更新，没有则插入。
- DELETE：删除数据库中与数据集对应的记录。
- DELETE_ALL：删除表中所有的记录，如果没有对应的表，则不受影响。
- TRUNCATE_TABLE：与DELETE_ALL类似，更轻量级，不能rollback。
- CLEAN_INSERT：是一个组合操作，是DELETE_ALL和INSERT的组合。是getSetUpOeration的默认返回值。

### 开始使用DbUnit
#### DataSet数据集准备
DataSet可以手工编写，当然也可以从已有数据库导出。以使用广泛的FlatXMlDataSet来说，可以[手工编写](http://www.dbunit.org/components.html#FlatXmlDataSet)。另外也可以从数据库读取，DbUnit提供了相关的API：

        QueryDataSet dataSet = new QueryDataSet(getConnection());
        dataSet.addTable("user", "select * from user ");
        FlatXmlDataSet.write(dataSet, new FileOutputStream("data.xml"));
#### 继承DBTestCase来实现测试用例
最简单的使用DbUnit的方式就是继承DBTestCase。当然有一些方法需要重写，比如getDataSet()用来读取DataSet并返回。DBTestCase依赖于IDatabaseTester来连接数据库。默认使用的是PropertiesBasedJdbcDatabaseTester，它会从系统变量内获取DriverManager使用的相关变量。如果要使用其它的方式，可以重写getDatabaseTester()。当然也可以直接继承DBTestCase的其它子类。比如：

- JdbcBasedDBTestCase
- DataSourceBasedDBTestCase
- JndiBasedDBTestCase

下面依默认的情况，那么如何设置系统变量呢？在Constructor里就好了。

```java
public class SampleTest extends DBTestCase
{
    public SampleTest(String name)
    {
        super( name );
        System.setProperty(PropertiesBasedJdbcDatabaseTester.DBUNIT_DRIVER_CLASS, "org.h2.Driver");
        System.setProperty(PropertiesBasedJdbcDatabaseTester.DBUNIT_CONNECTION_URL, "jdbc:h2:~/dbunitdemo");
        System.setProperty(PropertiesBasedJdbcDatabaseTester.DBUNIT_USERNAME, "sa");
        System.setProperty(PropertiesBasedJdbcDatabaseTester.DBUNIT_PASSWORD, "");

    protected IDataSet getDataSet() throws Exception
    {
        return new FlatXmlDataSetBuilder().build(new FileInputStream("dataset.xml"));
    }
}
```
那么如果我要使用其它的DatabaseTester怎么办？比如JdbcDatabaseTester。那直接重写getDatabaseTester(),返回JdbcDatabaseTester就好了。其它同理。当然那个Constructor就可以去除哪些属性设置了。

```java
protected IDatabaseTester getDatabaseTester() throws ClassNotFoundException {
    return new JdbcDatabaseTester("org.h2.Driver", "jdbc:h2:~/dbunitdemo", "sa", "");
}
```

#### 定制测试执行前后的操作
默认情况下，在Test执行前会做一个CLEAN_INSERT操作，然后结束后做一个NONE操作。那么，如何定制这个操作呢？我们可以重写getSetUpOperation()和 getTearDownOperation()方法就可以了。

```java
protected DatabaseOperation getSetUpOperation() throws Exception
{
    return DatabaseOperation.REFRESH;
}

protected DatabaseOperation getTearDownOperation() throws Exception
{
    return DatabaseOperation.NONE;
}
```
#### DatabaseConfig设置
有时候，需要对DatabaseConnection做一些特殊的配置，那么这个时候，我们可以重写setUpDatabaseConfig(DatabaseConfig config)。

```java
protected void setUpDatabaseConfig(DatabaseConfig config) {
        config.setProperty(DatabaseConfig.PROPERTY_BATCH_SIZE, new Integer(97));
        config.setFeature(DatabaseConfig.FEATURE_BATCHED_STATEMENTS, true);
}
```
比如，我在使用H2Database时，老是报一个这样的warn:

    WARN org.dbunit.dataset.AbstractTableMetaData - Potential problem found: The configured data type factory 'class org.dbunit.dataset.datatype.DefaultDataTypeFactory' might cause problems with the current database 'H2' (e.g. some datatypes may not be supported properly). In rare cases you might see this message because the list of supported database products is incomplete (list=[derby]). If so please request a java-class update via the forums.If you are using your own IDataTypeFactory extending DefaultDataTypeFactory, ensure that you override getValidDbProducts() to specify the supported database products.
那么，如何让这个WARN消失呢？加上这个配置：

```java
config.setProperty(DatabaseConfig.PROPERTY_DATATYPE_FACTORY, new H2DataTypeFactory());
```
#### Database数据验证
DbUnit提供了校验2个Table或者datasets是否包含相同数据的方法。

```java
public class Assertion
{
    public static void assertEquals(ITable expected, ITable actual)
    public static void assertEquals(IDataSet expected, IDataSet actual)
}
```
下面的例子，展示了如何对比一个数据库Table和Flat Xml table的数据是否一致。


```java
    // Fetch database data after executing your code
    IDataSet databaseDataSet = getConnection().createDataSet();
    ITable actualTable = databaseDataSet.getTable("TABLE_NAME");

    // Load expected data from an XML dataset
    IDataSet expectedDataSet = new FlatXmlDataSetBuilder().build(new File("expectedDataSet.xml"));
    ITable expectedTable = expectedDataSet.getTable("TABLE_NAME");

    // Assert actual database table match expected table
    Assertion.assertEquals(expectedTable, actualTable);
```
### DbUnit的运行步骤
由于DBTestCase最终都是继承自JUnit的TestCase的，很明显，在一个测试方法执行前都会调用setUp(),执行后调用tearDown()。在DatabaseTestCase中对这2个函数进行了重写，如下所示：

```java
protected void setUp() throws Exception
{
    logger.debug("setUp() - start");

    super.setUp();
    final IDatabaseTester databaseTester = getDatabaseTester();
    assertNotNull( "DatabaseTester is not set", databaseTester );
    databaseTester.setSetUpOperation( getSetUpOperation() );
    databaseTester.setDataSet( getDataSet() );
    databaseTester.setOperationListener(getOperationListener());
    databaseTester.onSetup();
}

protected void tearDown() throws Exception
{
    logger.debug("tearDown() - start");

    try {
        final IDatabaseTester databaseTester = getDatabaseTester();
        assertNotNull( "DatabaseTester is not set", databaseTester );
        databaseTester.setTearDownOperation( getTearDownOperation() );
        databaseTester.setDataSet( getDataSet() );
        databaseTester.setOperationListener(getOperationListener());
        databaseTester.onTearDown();
    } finally {
        tester = null;
        super.tearDown();
    }
}
```
这2个重写的方法，也验证了前面所说，DBTestCase是依赖于databaseTester的。前面所做的一些重写方法，在这里得到了使用，从而改变了测试执行的动作。databaseTester.onSetup()，databaseTester.onTearDown()分别按照配置完成了测试执行前后的操作。

总结：

- 1.移除数据库中的所有记录（CLEAN_INSERT中的DELETE_ALL)。
- 2.将数据集中的数据加载到数据库中（CLEAN_INSERT中的INSERT)。
- 3.运行测试。
- 4.测试运行完毕后，不执行任何操作。

### 使用什么数据库？
由于单元测试，与生产环境不要求是一样的数据库。因此，原则上是可以随意选取的。但是考虑到敏捷性，尽量选取轻量级的，以及可移植的。
这里推荐一个数据库[H2Database](http://www.h2database.com/html/main.html)，它是一个内存数据库，极为轻量。它与其它几种数据库的对比如下：

|                           | H2   | Derby | HSQLDB | MySQL | PostgreSQL |
| ------------------------- | ---- | ----- | ------ | ----- | ---------- |
| Pure Java                 | YES  | YES   | YES    | NO    | NO         |
| Memory Mode               | YES  | YES   | YES    | NO    | NO         |
| Encrypted Database        | YES  | YES   | YES    | NO    | NO         |
| ODBC Driver               | YES  | NO    | NO     | YES   | YES        |
| Fulltext Search           | YES  | NO    | NO     | YES   | YES        |
| Multi Version Concurrency | YES  | NO    | YES    | YES   | YES        |
| Footprint (jar/dll size)  | ~1MB | ~2 MB | ~1 MB  | ~4 MB | ~6 MB      |

之所以选择内存数据库，是因为在诸如持续集成时，不同的机器可能配置不一样，想运行还要搭建数据库，这个比较麻烦啊。
有了数据库就牵涉到如何初始化数据库的问题。如果你使用MAVEN触发Test，这里推荐一个[sql-maven-plugin](http://mojo.codehaus.org/sql-maven-plugin/)。它可以方便的执行数据库SQL脚本来创建数据库。

```xml
            <plugin>
                <groupId>org.codehaus.mojo</groupId>
                <artifactId>sql-maven-plugin</artifactId>
                <version>1.5</version>
                <dependencies>
                    <dependency>
                        <groupId>com.h2database</groupId>
                        <artifactId>h2</artifactId>
                        <version>1.3.172</version>
                    </dependency>
                </dependencies>
                <configuration>
                    <driver>org.h2.Driver</driver>
                    <url>jdbc:h2:~/dbunitdemo</url>
                    <username>sa</username>
                    <password></password>
                    <srcFiles>
                        <srcFile>${project.basedir}/src/main/sql/dbunitdemo.sql</srcFile>
                    </srcFiles>
                </configuration>
                <executions>
                    <execution>
                        <id>create-db</id>
                        <phase>process-test-resources</phase>
                        <goals>
                            <goal>execute</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
```
这里，创建了一个H2Database。直接执行`mvn sql:execute`用来初始化数据库。当然，这里把执行配置到了process-test-resources阶段，直接执行`mvn test`就OK了。
