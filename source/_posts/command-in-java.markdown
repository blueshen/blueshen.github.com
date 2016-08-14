---
layout: post
title: "设计模式：命令（Command） in java"
date: 2012-11-08 15:09
comments: true
categories: 设计模式
tags: [ Java, command ]
---
**定义：**命令模式是一种高内聚的模式。它将一个请求封装成一个对象，从而让使用不同请求来把客户端参数化，对请求排队或者记录请求日志，可以提供命令的撤销与恢复功能。   
听起来，好复杂！  

在程序员之间，流传着这样一句话：程序写到最后，就是if-else,for,while。  
真是枯燥啊！其实能体会到这种感觉，说明作为一个程序员，你已经有了一定的造诣了。那如何提高呢？   

试想，在代码中，你有很多if-else或者case语句。为什么有这样的语句呢？因为判断条件多啊，需要根据不同的条件来做不同的事情。2、3个条件还可以写，如果有20个条件呢，或者N多呢。那么我们的if-else那就判断N重条件，这简直是无法忍受的，写出的代码可维护性更不用说了。    
<!--more-->   
**命令模式**就是可以解决这种问题的方法之一。下面就来提高一下程序员的自我修养了。
   
命令模式中，主要有3个角色：  

* Receiver命令接收者
* Command命令
* Invoker命令调用者  

下面开始分别定义：   
**Receiver:**定义命令的N种接收者   

	public abstract class AbstractReceiver {
		public abstract void doSomething();
	}

	public class Receiver1 extends AbstractReceiver {
		@Override
		public void doSomething() {
			System.out.println("receiver1 do something");
		}
	}
	public class Receiver2 extends AbstractReceiver {
		@Override
		public void doSomething() {
			System.out.println("receiver2 do something");
		}
	}
	...
	public class ReceiverN
	...

**Command:**定义N种命令   

	public abstract class AbstractCommand {
		public abstract void execute();
	}
	
	public class Command1 extends AbstractCommand {
		private AbstractReceiver receiver;
		public Command1(AbstractReceiver receiver) {
			this.receiver = receiver;
		}
		@Override
		public void execute() {
			System.out.println("command1 命令发出");
			receiver.doSomething();
		}
	}
	public class Command2 extends AbstractCommand {
		private AbstractReceiver receiver;
		public Command2(AbstractReceiver receiver) {
			this.receiver = receiver;
		}
		@Override
		public void execute() {
			System.out.println("command2 命令发出");
			receiver.doSomething();
		}
	}
	...
    public class CommandN
	....

**Invoker:**定义调用者   

	public class Invoker {

		private List<AbstractCommand> commandList = new LinkedList<AbstractCommand>();

		public void addCommand(AbstractCommand command) {
			commandList.add(command);
		}

		public void addCommands(LinkedList<AbstractCommand> commands) {
			commandList.addAll(commands);
		}

		public void action() {
			for (AbstractCommand command : commandList) {
				command.execute();
			}
		}

	}
使用场景：  

	Invoker invoker = new Invoker();
	AbstractReceiver receiver1 = new Receiver1();
	AbstractReceiver receiver2 = new Receiver2();
	AbstractCommand command1 = new Command1(receiver2);
	AbstractCommand command2 = new Command2(receiver1);
	invoker.addCommand(command1);
	invoker.addCommand(command2);
	invoker.action();
运行结果：  

	command1 命令发出
	receiver2 do something
	command2 命令发出
	receiver1 do something
现在再回过来看命令模式的定义，就比较明白了吧。也就是说有N种请求条件，那么就定义N个类来封装请求，我们称之为命令（Command）。每个命令做什么操作呢，谁来执行这个命令呢，有命令自己来进行定义。这样就避免了if-else，而由N种命令来决定跳转关系。   

调用者（Invoker）呢，它维护了一个命令列表，并按照一定的顺序来发起命令调用。当然这个列表也有可能就只有一个命令了，就简化了一下。与if-else对比更直观的了。  

定义中还提到了命令撤销或恢复的功能，这种撤销与恢复也是命令的一种了，一般可以通过扩展一个命令出来，通过诸如日志等来恢复之前的操作。其实这个也是可以通过备忘录模式来实现的。
##命令模式 in JDK##

	java.lang.Runnable#run()
	javax.swing.Action#actionPeformed(ActionEvent e)
从Runnable来说，不同的实现者，通过调用run()来实现在不同线程执行不同的操作。
从Action来说，由于桌面UI有很多的操作事件，这些事件就是命令。通过actionPerformed函数，接受不同的命令参数来做出不同的表现。  