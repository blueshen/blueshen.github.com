www.shenyanchao.cn

---
layout: post
title: "Java新特性"
date: 2014-08-27 14:43
comments: true
categories: java
tags: [ java7, java8 ]
---
##Java7新特性

###Java7语法特性
根据JSR 334，Java7添加了数个语法方面的新特性：

####1. switch可以消化String

比如：

    public static void switchString(String s){
        switch (s){
        case "db": ...
        case "wls": ...
        case "ibm": ...
        case "soa": ...
        case "fa": ...
        default: ...
        }
    }
<!--more-->

####2. 新的整数字面表达方式 - "0b"前缀和"_"连数符，提升程序员的幸福感。
- a. 表示二进制字面值的前缀, 0b:
 比如以下三个变量的值相同：
 
        byte b1 = 0b00100001;     // New
        byte b2 = 0x21;        // Old
        byte b3 = 33;        // Old

- b. 用下划线连接整数提升其可读性，自身无含义，不可用在数字的起始和末尾：
 
    	long phone_nbr = 021_1111_2222;

####3. 简化了泛型对象创建的语法 - "菱形 new"，以下两个语句等价：

        ArrayList<String> al1 = new ArrayList<String>();    // Old
        ArrayList<String> al2 = new ArrayList<>();        // New

####4. 为所有的reflect操作异常找了个新爸爸 - ReflectOperationException，孩儿们是：

        ClassNotFoundException, 
        IllegalAccessException, 
        InstantiationException, 
        InvocationTargetException, 
        NoSuchFieldException, 
        NoSuchMethodException

####5. catch有了多重捕获功能，也玩起了包养的勾当，以下代码心领神会：

        try{
            // code
        }
        catch (SQLException | IOException ex) {
            // ...
        }

####6. 异常精确重抛 - 重抛时自动造型为子类，有点半主动制导武器的style：

        public void test() throws NoSuchMethodException, NoSuchFieldException{    // 子类
            try{
                // code
            }
            catch (RelectiveOperationException ex){    // 父类
                throws ex;
            }
        }

####7. 发明了try()结构 - Try with Resources，能够自动接住异常并关闭资源(所谓的资源需要利用新的java.lang.AutoCloseable接口)，注意以下代码中try后面跟的是"("不是"{"：
    try(BufferedReader br = new BufferedReader(new FileReader("/home/oracle/temp.txt"))){
        ... br.readLine() ...
    }
try-with-resources语句可以带catch，也可以向上例一样一个catch也没有。

###Java7 NIO 新方法
整体来说，对IO操作进行了优化，使用起来更加顺手，甚至可以替换apache common-io包。

####1.增加`java.nio.file.Paths`用于目录操作

		Path path = Paths.get("/home/shenyanchao", "Desktop");
        System.out.println(path.toAbsolutePath());
        System.out.println(path.getParent());
        System.out.println(path.getFileSystem().isOpen());
####2.增加`java.nio.file.Files`工具类来处理文件

        Files.copy(src,dest, StandardCopyOption.COPY_ATTRIBUTES,StandardCopyOption.REPLACE_EXISTING);

        Files.move(src,dest,StandardCopyOption.ATOMIC_MOVE);

        Files.createLink(src,dest);
        Files.createSymbolicLink(src,dest);
        Files.deleteIfExists(dest);

        Files.readAllLines(src);

        Files.createTempFile(src,"aa","bb");
####3.目录树遍历
使用`FileVisitor`来实现访问者模式。

    preVisitDirectory(T dir, BasicFileAttributes attrs);
    visitFile(T dir, BasicFileAttributes attrs);
    visitFileFailed(T dir, IOException exc);
    postVisitDirectory(T dir, IOException exc);
####4.使用WatchService来监控目录，变化请通知

        WatchService watchService = FileSystems.getDefault().newWatchService();
        Path path = Paths.get("/home/shenyanchao/Documents");
        WatchKey watchKey = path.register(watchService, StandardWatchEventKinds.ENTRY_CREATE,
                StandardWatchEventKinds.ENTRY_DELETE, StandardWatchEventKinds.ENTRY_MODIFY);
        while (true) {
            List<WatchEvent<?>> watchEvents = watchService.take().pollEvents();
            for (WatchEvent<?> watchEvent : watchEvents) {
                System.out.printf("[%s]文件发生了[%s]事件。%n", watchEvent.context(), watchEvent.kind());
            }
            boolean valid = watchKey.reset();
            if (!valid){
                break;
            }
        }
        
###Java7并发（JSR166Y）

####Fork Join框架，大任务分解为小任务
通过ForkJoinPool，ForkJoinTask来实现的。

    public class Fibonacci extends RecursiveTask<Integer> {

        final int n;

        Fibonacci(int n) {
            this.n = n;
        }

        @Override
        protected Integer compute() {
            if (n <= 1)
                return n;
            Fibonacci f1 = new Fibonacci(n - 1);
            f1.fork();
            Fibonacci f2 = new Fibonacci(n - 2);
            f2.fork();
            return f1.join() + f2.join();
        }

        public static void main(String[] args) {
            Fibonacci fibonacci = new Fibonacci(4);
            System.out.println(fibonacci.compute());
        }
    }
    
####TransferQueue，ConcurrentLinkedDeque等新类
TransferQueue是一个继承了 BlockingQueue的接口，并且增加若干新的方法。

####ThreadLocalRandom用于生成随机数

	ThreadLocalRandom.current().nextInt (...)
Random是线程安全的，但速度较慢。而这个是快速的，但是速度很快。适用于线程内部的使用。

###Java7 client
诸如更新了很多swing显示相关的api.   
更好的支持linux fonts

###Java7 VM新特性

####1.引入Garbage First回收算法
Garbage First简称G1，它的目标是要做到尽量减少GC所导致的应用暂停的时间，让应用达到准实时的效果，同时保持JVM堆空间的利用率。用于替代CMS

---

参考文档：<http://www.slideshare.net/boulderjug/55-things-in-java-7>

##Java8新特性
###1.接口默认方法[接口允许有实现啦]
Java 8允许我们给接口添加一个非抽象的方法实现，只需要使用 `default`关键字即可，这个特征又叫做扩展方法。

	interface Formula {
    	double calculate(int a);

    	default double sqrt(int a) {
        	return Math.sqrt(a);
    	}
	}

###2.Lambda表达式
据说借鉴了各种动态语言的新特性，比如scala，python
首先看看在老版本的Java中是如何排列字符串的：

    List<String> names = Arrays.asList("peter", "anna", "mike", "xenia");

    Collections.sort(names, new Comparator<String>() {
    @Override
    public int compare(String a, String b) {
        return b.compareTo(a);
    }
    });
Java 8 提供了更为简介的语法，lambda表达式：

    Collections.sort(names, (String a, String b) -> {
        return b.compareTo(a);
    });

可以更短：

	Collections.sort(names, (String a, String b) -> b.compareTo(a));
再短： 

    Collections.sort(names, (a, b) -> b.compareTo(a));
###3.@FunctionalInterface
这是新引入的一个注解，用于支持lambda。用`FunctionalInterface`标识的接口，必须只包含1个抽象方法。否则会编译报错的。因为如果有多个抽象方法，lambda就无法知道对应哪个方法了。

    @FunctionalInterface
    interface Converter<F, T> {
        T convert(F from);
    }

    Converter<String, Integer> converter = (from) -> Integer.valueOf(from);
    Integer converted = converter.convert("123");
    System.out.println(converted);    // 123

例子中，`(from) -> Integer.valueOf(from)`这个lambda表达式指出了convert方法的具体实现。

####(1)方法与构造函数的引用::
Java 8 允许你使用 :: 关键字来传递方法或者构造函数引用,下面的代码展示了如何引用一个静态方法

    Converter<String, Integer> converter = Integer::valueOf;
    Integer converted = converter.convert("123");
    System.out.println(converted);   // 123

当然也可以引用一个对象的方法：

    converter = something::startsWith;
    String converted = converter.convert("Java");
    System.out.println(converted);    // "J"
    
那么如何引用构造函数呢？

    class Person {
        String firstName;
        String lastName;

        Person() {}

        Person(String firstName, String lastName) {
            this.firstName = firstName;
            this.lastName = lastName;
        }
    }
这是一个pojo类。下面创建一个用于创建person对象的FunctionalInterface：

    interface PersonFactory<P extends Person> {
        P create(String firstName, String lastName);
    }
这里我们使用构造函数引用来将他们关联起来，而不是实现一个完整的工厂：

    PersonFactory<Person> personFactory = Person::new;
    Person person = personFactory.create("Peter", "Parker");
我们只需要使用 Person::new 来获取Person类构造函数的引用，Java编译器会自动根据PersonFactory.create方法的签名来选择合适的构造函数。

####(2)内嵌的Functional Interfaces
上面提到的Comparator接口，为什么可以使用lambda表达式，正是因为在java 8中，被定义为了FunctionalInterface。这些已经存在的接口是通过添加@FunctionalInterface注解来支持的。

另外，java 8 api还提供了丰富的接口。这些接口貌似都是从Google Guava包里得到的提示，命名甚至都是一样的。

- Predicates   

Predicate 接口只有一个参数，返回boolean类型。该接口包含多种默认方法来将Predicate组合成其他复杂的逻辑（比如：与，或，非）：

        Predicate<String> predicate = (s) -> s.length() > 0;

        predicate.test("foo");              // true
        predicate.negate().test("foo");     // false

        Predicate<Boolean> nonNull = Objects::nonNull;
        Predicate<Boolean> isNull = Objects::isNull;

        Predicate<String> isEmpty = String::isEmpty;
        Predicate<String> isNotEmpty = isEmpty.negate();
        
- Function

Function 接口有一个参数并且返回一个结果，并附带了一些可以和其他函数组合的默认方法（compose, andThen）：

        Function<String, Integer> toInteger = Integer::valueOf;
        Function<String, String> backToString = toInteger.andThen(String::valueOf);

        backToString.apply("123");     // "123"
        
- Supplier

Supplier 接口返回一个给定类型的泛型，和Function接口不同的是该接口不接受任何参数.

        Supplier<Person> personSupplier = Person::new;
        personSupplier.get();   // new Person
        
- Consumer

Consumer在一个输入参数上做一些操作。

        Consumer<Person> greeter = (p) -> System.out.println("Hello, " + p.firstName);
        greeter.accept(new Person("Luke", "Skywalker"));

- Comparator

Comparator是早就存在的，不过java 8提供了一些默认方法。

        Comparator<Person> comparator = (p1, p2) -> p1.firstName.compareTo(p2.firstName);

        Person p1 = new Person("John", "Doe");
        Person p2 = new Person("Alice", "Wonderland");

        comparator.compare(p1, p2);             // > 0
        comparator.reversed().compare(p1, p2);  // < 0
        
- Optional

Optional其实并不是一个FunctionInterface，而是一个用来避免`NullPointerException`的工具。

        Optional<String> optional = Optional.of("bam");

        optional.isPresent();           // true
        optional.get();                 // "bam"
        optional.orElse("fallback");    // "bam"

        optional.ifPresent((s) -> System.out.println(s.charAt(0)));     // "b"
        
- Stream

`java.util.stream.Stream`代表了能在其上做一系列操作的一串元素。在java 8中，Collections被扩展了。我们可以通过`Collections.stream()`或者`Collections.parallelStream()`来创建Stream。
首先，新建一个数组。

        List<String> stringCollection = new ArrayList<>();
        stringCollection.add("ddd2");
        stringCollection.add("aaa2");
        stringCollection.add("bbb1");
        stringCollection.add("aaa1");
        stringCollection.add("bbb3");
        stringCollection.add("ccc");
        stringCollection.add("bbb2");
        stringCollection.add("ddd1");
具体使用如下：

        stringCollection
            .stream()
            .sorted()
            .filter((s) -> s.startsWith("a"))
            .forEach(System.out::println);

        // "aaa1", "aaa2"
map()函数可以把每个值转换为另外的值。

        stringCollection
            .stream()
            .map(String::toUpperCase)
            .sorted((a, b) -> b.compareTo(a))
            .forEach(System.out::println);

        // "DDD2", "DDD1", "CCC", "BBB3", "BBB2", "AAA2", "AAA1"        
另外提供有各种match方法。

        boolean anyStartsWithA =
            stringCollection
                .stream()
                .anyMatch((s) -> s.startsWith("a"));

        System.out.println(anyStartsWithA);      // true

        boolean allStartsWithA =
            stringCollection
                .stream()
                .allMatch((s) -> s.startsWith("a"));

        System.out.println(allStartsWithA);      // false

        boolean noneStartsWithZ =
            stringCollection
                .stream()
                .noneMatch((s) -> s.startsWith("z"));

        System.out.println(noneStartsWithZ);      // true
count()方法返回的是一个值。

        long startsWithB =
            stringCollection
                .stream()
                .filter((s) -> s.startsWith("b"))
                .count();

        System.out.println(startsWithB);    // 3        
reduce则进行了归一化处理：

        Optional<String> reduced =
            stringCollection
                .stream()
                .sorted()
                .reduce((s1, s2) -> s1 + "#" + s2);

        reduced.ifPresent(System.out::println);
        // "aaa1#aaa2#bbb1#bbb2#bbb3#ccc#ddd1#ddd2"
        
- ParallelStream 

它是一个并行的。速度要比stream快的多。

- Map

Map并不支持Stream，不过Java 8 提供了各种有用的新方法：

        Map<Integer, String> map = new HashMap<>();

        for (int i = 0; i < 10; i++) {
            map.putIfAbsent(i, "val" + i);
        }

        map.forEach((id, val) -> System.out.println(val));

        map.computeIfPresent(3, (num, val) -> val + num);
        map.get(3);             // val33

        map.computeIfPresent(9, (num, val) -> null);
        map.containsKey(9);     // false

        map.computeIfAbsent(23, num -> "val" + num);
        map.containsKey(23);    // true

        map.computeIfAbsent(3, num -> "bam");
        map.get(3);             // val33

        map.remove(3, "val3");
        map.get(3);             // val33

        map.remove(3, "val33");
        map.get(3);             // null

        map.getOrDefault(42, "not found");  // not found

        map.merge(9, "val9", (value, newValue) -> value.concat(newValue));
        map.get(9);             // val9

        map.merge(9, "concat", (value, newValue) -> value.concat(newValue));
        map.get(9);             // val9concat
        
###4.时间API
Java8更新了Date API,这个新的api与joda-time类似。
####(1)Clock
Clock类提供了访问当前日期和时间的方法，Clock是时区敏感的，可以用来取代 System.currentTimeMillis() 来获取当前的微秒数。某一个特定的时间点也可以使用Instant类来表示，Instant类也可以用来创建老的java.util.Date对象。

        Clock clock = Clock.systemDefaultZone();
        long millis = clock.millis();

        Instant instant = clock.instant();
        Date legacyDate = Date.from(instant);   // legacy java.util.Date
        
####(2)TimeZones
在新API中时区使用ZoneId来表示。时区可以很方便的使用静态方法of来获取到。 时区定义了到UTS时间的时间差，在Instant时间点对象到本地日期对象之间转换的时候是极其重要的

        System.out.println(ZoneId.getAvailableZoneIds());
        // prints all available timezone ids

        ZoneId zone1 = ZoneId.of("Europe/Berlin");
        ZoneId zone2 = ZoneId.of("Brazil/East");
        System.out.println(zone1.getRules());
        System.out.println(zone2.getRules());

        // ZoneRules[currentStandardOffset=+01:00]
        // ZoneRules[currentStandardOffset=-03:00]
####(3)LocalTime
LocalTime 定义了一个没有时区信息的时间，例如 晚上10点，或者 17:30:15。下面的例子使用前面代码创建的时区创建了两个本地时间。之后比较时间并以小时和分钟为单位计算两个时间的时间差：

        LocalTime now1 = LocalTime.now(zone1);
        LocalTime now2 = LocalTime.now(zone2);

        System.out.println(now1.isBefore(now2));  // false

        long hoursBetween = ChronoUnit.HOURS.between(now1, now2);
        long minutesBetween = ChronoUnit.MINUTES.between(now1, now2);

        System.out.println(hoursBetween);       // -3
        System.out.println(minutesBetween);     // -239
LocalTime 提供了多种工厂方法来简化对象的创建，包括解析时间字符串

        LocalTime late = LocalTime.of(23, 59, 59);
        System.out.println(late);       // 23:59:59

        DateTimeFormatter germanFormatter =
            DateTimeFormatter
                .ofLocalizedTime(FormatStyle.SHORT)
                .withLocale(Locale.GERMAN);

        LocalTime leetTime = LocalTime.parse("13:37", germanFormatter);
        System.out.println(leetTime);   // 13:37
        
####(4)LocalDate
LocalDate 表示了一个确切的日期，比如 2014-03-11。该对象值是不可变的，用起来和LocalTime基本一致。下面的例子展示了如何给Date对象加减天/月/年。另外要注意的是这些对象是不可变的，操作返回的总是一个新实例。

        LocalDate today = LocalDate.now();
        LocalDate tomorrow = today.plus(1, ChronoUnit.DAYS);
        LocalDate yesterday = tomorrow.minusDays(2);

        LocalDate independenceDay = LocalDate.of(2014, Month.JULY, 4);
        DayOfWeek dayOfWeek = independenceDay.getDayOfWeek();
        System.out.println(dayOfWeek);    // FRIDAY

        DateTimeFormatter germanFormatter =
            DateTimeFormatter
                .ofLocalizedDate(FormatStyle.MEDIUM)
                .withLocale(Locale.GERMAN);

        LocalDate xmas = LocalDate.parse("24.12.2014", germanFormatter);
        System.out.println(xmas);   // 2014-12-24
####(5)LocalDateTime
        LocalDateTime 同时表示了时间和日期，相当于前两节内容合并到一个对象上了。LocalDateTime和LocalTime还有LocalDate一样，都是不可变的。LocalDateTime提供了一些能访问具体字段的方法。

        LocalDateTime sylvester = LocalDateTime.of(2014, Month.DECEMBER, 31, 23, 59, 59);

        DayOfWeek dayOfWeek = sylvester.getDayOfWeek();
        System.out.println(dayOfWeek);      // WEDNESDAY

        Month month = sylvester.getMonth();
        System.out.println(month);          // DECEMBER

        long minuteOfDay = sylvester.getLong(ChronoField.MINUTE_OF_DAY);
        System.out.println(minuteOfDay);    // 1439

        Instant instant = sylvester
                .atZone(ZoneId.systemDefault())
                .toInstant();

        Date legacyDate = Date.from(instant);
        System.out.println(legacyDate);     // Wed Dec 31 23:59:59 CET 2014

        DateTimeFormatter formatter =
            DateTimeFormatter
                .ofPattern("MMM dd, yyyy - HH:mm");

        LocalDateTime parsed = LocalDateTime.parse("Nov 03, 2014 - 07:13", formatter);
        String string = formatter.format(parsed);
        System.out.println(string);     // Nov 03, 2014 - 07:13
        
###5.支持重复注解了，注解也可以用在任何地方了
java8之前，类，属性，方法才有注解，现在几乎任何地方都可以了。

    new @Interned MyObject();
    myString = (@NonNull String) str;
    
    void monitorTemperature() throws @Critical TemperatureException { ... }
之前要实现重复注解需要这样：

    public @interface Authority {
         String role();
    }
     
    public @interface Authorities {
        Authority[] value();
    }
     
    public class RepeatAnnotationUseOldVersion {
         
        @Authorities({@Authority(role="Admin"),@Authority(role="Manager")})
        public void doSomeThing(){
        }
    }

而现在：

    @Repeatable(Authorities.class)
    public @interface Authority {
         String role();
    }
     
    public @interface Authorities {
        Authority[] value();
    }
     
    public class RepeatAnnotationUseNewVersion {
        @Authority(role="Admin")
        @Authority(role="Manager")
        public void doSomeThing(){ }
    }
        
###6.Nashorn JavaScript 引擎
简单的说，它是 Rhino 的接替者.升级啦。

###7.StampedLock
它是java8在java.util.concurrent.locks新增的一个API。

ReentrantReadWriteLock 在沒有任何读写锁时，才可以取得写入锁，这可用于实现了悲观读取（Pessimistic Reading），即如果执行中进行读取时，经常可能有另一执行要写入的需求，为了保持同步，ReentrantReadWriteLock 的读取锁定就可派上用场。

然而，如果读取执行情况很多，写入很少的情况下，使用 ReentrantReadWriteLock 可能会使写入线程遭遇饥饿（Starvation）问题，也就是写入线程吃吃无法竞争到锁定而一直处于等待状态。

StampedLock控制锁有三种模式（写，读，乐观读），一个StampedLock状态是由版本和模式两个部分组成，锁获取方法返回一个数字作为票据stamp，它用相应的锁状态表示并控制访问，数字0表示没有写锁被授权访问。在读锁上分为悲观锁和乐观锁。

所谓的乐观读模式，也就是若读的操作很多，写的操作很少的情况下，你可以乐观地认为，写入与读取同时发生几率很少，因此不悲观地使用完全的读取锁定，程序可以查看读取资料之后，是否遭到写入执行的变更，再采取后续的措施（重新读取变更信息，或者抛出异常） ，这一个小小改进，可大幅度提高程序的吞吐量！！

它是java8在java.util.concurrent.locks新增的一个API。

ReentrantReadWriteLock 在沒有任何读写锁时，才可以取得写入锁，这可用于实现了悲观读取（Pessimistic Reading），即如果执行中进行读取时，经常可能有另一执行要写入的需求，为了保持同步，ReentrantReadWriteLock 的读取锁定就可派上用场。

然而，如果读取执行情况很多，写入很少的情况下，使用 ReentrantReadWriteLock 可能会使写入线程遭遇饥饿（Starvation）问题，也就是写入线程吃吃无法竞争到锁定而一直处于等待状态。

StampedLock控制锁有三种模式（写，读，乐观读），一个StampedLock状态是由版本和模式两个部分组成，锁获取方法返回一个数字作为票据stamp，它用相应的锁状态表示并控制访问，数字0表示没有写锁被授权访问。在读锁上分为悲观锁和乐观锁。

所谓的乐观读模式，也就是若读的操作很多，写的操作很少的情况下，你可以乐观地认为，写入与读取同时发生几率很少，因此不悲观地使用完全的读取锁定，程序可以查看读取资料之后，是否遭到写入执行的变更，再采取后续的措施（重新读取变更信息，或者抛出异常） ，这一个小小改进，可大幅度提高程序的吞吐量！！

    class Point {
       private double x, y;
       private final StampedLock sl = new StampedLock();
       void move(double deltaX, double deltaY) { // an exclusively locked method
         long stamp = sl.writeLock();
         try {
           x += deltaX;
           y += deltaY;
         } finally {
           sl.unlockWrite(stamp);
         }
       }
      //下面看看乐观读锁案例
       double distanceFromOrigin() { // A read-only method
         long stamp = sl.tryOptimisticRead(); //获得一个乐观读锁
         double currentX = x, currentY = y; //将两个字段读入本地局部变量
         if (!sl.validate(stamp)) { //检查发出乐观读锁后同时是否有其他写锁发生？
            stamp = sl.readLock(); //如果没有，我们再次获得一个读悲观锁
            try {
              currentX = x; // 将两个字段读入本地局部变量
              currentY = y; // 将两个字段读入本地局部变量
            } finally {
               sl.unlockRead(stamp);
            }
         }
         return Math.sqrt(currentX * currentX + currentY * currentY);
       }
    //下面是悲观读锁案例
       void moveIfAtOrigin(double newX, double newY) { // upgrade
         // Could instead start with optimistic, not read mode
         long stamp = sl.readLock();
         try {
           while (x == 0.0 && y == 0.0) { //循环，检查当前状态是否符合
             long ws = sl.tryConvertToWriteLock(stamp); //将读锁转为写锁
             if (ws != 0L) { //这是确认转为写锁是否成功
               stamp = ws; //如果成功 替换票据
               x = newX; //进行状态改变
               y = newY; //进行状态改变
               break;
             }
             else { //如果不能成功转换为写锁
               sl.unlockRead(stamp); //我们显式释放读锁
               stamp = sl.writeLock(); //显式直接进行写锁 然后再通过循环再试
             }
           }
         } finally {
           sl.unlock(stamp); //释放读锁或写锁
         }
       }
     }
     
- - -

参考文档：<http://winterbe.com/posts/2014/03/16/java-8-tutorial/>