---
layout: post
title: "Spring中配置quartz集群"
date: 2014-06-24 15:10
comments: true
categories: java
tags: [ quartz, spring, 集群 ]
---
### 为什么使用quartz集群？

在服务部署一个节点的时候，quartz任务是可以正常运行的。但是如果你业务上需要部署2个或者以上的集群时，就需要处理集群之间的定时任务执行问题了。而quartz集群就是为了解决这个问题的。前提是集群的时间同步，以及共用同一个数据库。
quartz集群在spring中的配置
#### 1.导入数据库表

以mysql为例，下载quartz发行版，在/docs/dbTables下找到tables_mysql_innodb.sql。导入数据结构到数据库内。 使用tables_mysql.sql的话，由于没有指定使用innodB引擎，在一些默认使用MYISAM的数据库实例内可能会报错。

注意事项：

修改SQL： TYPE=InnoDB –> ENGINE=InnoDB
<!--more-->

#### 2.项目中加入配置文件quartz.properties

    #============================================================================
    # Configure Main Scheduler Properties
    #============================================================================
    org.quartz.scheduler.instanceName = ClusteredScheduler
    org.quartz.scheduler.instanceId = AUTO
    org.quartz.scheduler.skipUpdateCheck = true
    
    #============================================================================
    # Configure ThreadPool
    #============================================================================
    org.quartz.threadPool.class = org.quartz.simpl.SimpleThreadPool
    org.quartz.threadPool.threadCount = 5
    org.quartz.threadPool.threadPriority = 5
    
    #============================================================================
    # Configure JobStore
    #============================================================================
    org.quartz.jobStore.class = org.quartz.impl.jdbcjobstore.JobStoreTX
    org.quartz.jobStore.driverDelegateClass=org.quartz.impl.jdbcjobstore.StdJDBCDelegate
    org.quartz.jobStore.misfireThreshold = 60000
    org.quartz.jobStore.useProperties = false
    org.quartz.jobStore.tablePrefix = QRTZ_
    
    org.quartz.jobStore.isClustered = true
    org.quartz.jobStore.clusterCheckinInterval = 15000

#### 3.增加applicationContext-quartz.xml

    <?xml version="1.0" encoding="UTF-8"?>
    <beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
           xmlns:util="http://www.springframework.org/schema/util"
           xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
                            http://www.springframework.org/schema/util http://www.springframework.org/schema/util/spring-util-3.0.xsd"
           default-lazy-init="false">
    
        <description>Quartz的定时集群任务配置</description>
    
        <bean id="quartzDataSource" class="org.springframework.jdbc.datasource.SimpleDriverDataSource">
            <property name="driverClass" value="${db.driver}" />
            <property name="url" value="${db.url}" />
            <property name="username" value="${db.user}" />
            <property name="password" value="${db.pass}" />
        </bean>
    
        <!-- Quartz集群Schduler -->
        <bean id="clusterQuartzScheduler" class="org.springframework.scheduling.quartz.SchedulerFactoryBean">
            <!-- Triggers集成 -->
            <property name="triggers">
                <list>
                    <ref bean="testTrigger" />
                </list>
            </property>
            <!--  quartz配置文件路径-->
            <property name="configLocation" value="classpath:quartz/quartz.properties" />
            <!-- 启动时延期3秒开始任务 -->
            <property name="startupDelay" value="3" />
            <!-- 保存Job数据到数据库所需的数据源 -->
            <property name="dataSource" ref="quartzDataSource" />
            <!-- Job接受applicationContext的成员变量名 -->
            <property name="applicationContextSchedulerContextKey" value="applicationContext" />
            <property name="overwriteExistingJobs" value="true" />
            <property name="jobFactory">
                <bean class="com.shenyanchao.quartz.AutoWiringSpringBeanJobFactory"/>
            </property>
         </bean>


        <bean id="testTrigger" class="org.springframework.scheduling.quartz.CronTriggerBean">
            <property name="jobDetail" ref="testJobDetail" />
            <property name="cronExpression" value="* 0/10 * * * ?" />
        </bean>
    
        <!-- Timer JobDetail, 基于JobDetailBean实例化Job Class,可持久化到数据库实现集群 -->
        <bean id="testJobDetail" class="org.springframework.scheduling.quartz.JobDetailBean">
            <property name="jobClass" value="cn.shenyanchao.quartz.TestTask" />
        </bean>
    
        <!-- Timer Job的可配置属性,在job中通过applicationContext动态获取 -->
        <util:map id="timerJobConfig">
            <entry key="nodeName" value="default" />
        </util:map>
    </beans>

其中尤其注意，设置overwriteExistingJobs为true，这个选项可以在修改cronExpression之后，能够更新到数据库，否则无法生效。

另外，配置JobFactory使得QuartzJob可以@Autowired注入spring托管的实例。内容如下：

    public final class AutoWiringSpringBeanJobFactory extends SpringBeanJobFactory implements ApplicationContextAware {
    
            private transient AutowireCapableBeanFactory beanFactory;
    
            public void setApplicationContext(final ApplicationContext context) {
                beanFactory = context.getAutowireCapableBeanFactory();
            }
    
            @Override
            protected Object createJobInstance(final TriggerFiredBundle bundle) throws Exception {
                final Object job = super.createJobInstance(bundle);
                beanFactory.autowireBean(job);
                return job;
            }
        }

#### 4. 如何写JOB？

    @Component
    public class TestTask extends QuartzJobBean {


        @Autowired
        private UserService userService;
    
        @Override
        protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
            System.out.println(userService.findByName("shenyanchao").getEmail());
        }
    }

由于使用MethodInvokingFactoryBean总是报seriziable错误，因此本例使用的是JobDetailBean。那这也意味着要继承QuartzJobBean。同时由于配置了JobFactory，使得可以直接注入UserService等实例。

#### 5.quartz在mysql5.6下报错

    You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near 'OPTION SQL_SELECT_LIMIT=5' at line 1

这个错误是由于mysql connector的版本太低导致的，可以通过升级版本来解决。 参见<http://stackoverflow.com/questions/13023548/mysql-server-version-for-the-right-syntax-to-use-near-option-sql-select-limit-1>
