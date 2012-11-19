---
layout: post
title: "设计模式：迭代器（Iterator） in java"
date: 2012-11-08 16:50
comments: true
categories: 设计模式
tags: [ Java, Iterator ]
---
**定义：**迭代器模式提供了一种方法来访问一个容器对象种的各个元素，而又不暴露这个对象的内部细节。   
>在Java中已经默认提供了Iterator支持，各种容器类都进行了实现，而事实上，迭代器模式就是为了解决如何遍历这些容器里的元素而诞生的。   

**迭代器模式**主要有以下的角色：

* Iterator抽象迭代器：   
负责定义访问与遍历元素的接口。基本有3个固定的方法`hasNext()`,`next()`,`remove()`;
* Concrete Iterator具体迭代器：    
迭代器的实现类，实现接口，完成元素遍历。
* Aggregate抽象容器：   
定义创建具体迭代器的接口。在Java种一般是`iterator()`方法。
* Concrete Aggregate具体容器：  
实现创建迭代器接口，返回迭代器实例对象。   

<!--more-->
为了方便，下面就直接以ArrayList为例，来看看迭代器是如何实现的吧。  
在使用时，通常这样对ArrayList进行遍历：

		List<String> list = new ArrayList<String>();
		list.add("first");
		list.add("second");
		Iterator<String> itr = list.iterator();
		while (itr.hasNext()) {
			String element = itr.next();
			System.out.println(element);
		}
从这里，我们可以看出来，这就是完全的迭代器模式。  
首先来看，Iterator接口（java.util.Iterator）：  

	public interface Iterator<E> {
  
   		boolean hasNext();
  
    	E next();
  
    	void remove();
	}
然后是Iterator的具体实现类(java.util.AbstractList<E>$Itr)：  
	private class Itr implements Iterator<E> {
	
		int cursor = 0;

		int lastRet = -1;
	
		int expectedModCount = modCount;

		public boolean hasNext() {
            return cursor != size();
		}

		public E next() {
            checkForComodification();
	    	try {
			E next = get(cursor);
			lastRet = cursor++;
			return next;
	    	} catch (IndexOutOfBoundsException e) {
				checkForComodification();
				throw new NoSuchElementException();
	    	}
		}

		public void remove() {
	    	if (lastRet == -1)
			throw new IllegalStateException();
            	checkForComodification();

	    	try {
				AbstractList.this.remove(lastRet);
			if (lastRet < cursor)
		    	cursor--;
			lastRet = -1;
			expectedModCount = modCount;
	    	} catch (IndexOutOfBoundsException e) {
				throw new ConcurrentModificationException();
	    	}
		}

		final void checkForComodification() {
	    	if (modCount != expectedModCount)
				throw new ConcurrentModificationException();
		}
    }
这个就是具体的Iterator了，当然还有另外一个ListItr extends Itr，这个是专门针对List进行操作的。   
那么Aggregate接口对应的是哪个呢？就是`java.util.AbstractCollection<E>` ，它里面定义了这样的一个接口：   

	...
  	/**
     * Returns an iterator over the elements contained in this collection.
     *
     * @return an iterator over the elements contained in this collection
     */
    public abstract Iterator<E> iterator();
	...
那么具体实现是在哪儿？`java.util.AbstractList<E>`做了实现。

	    public Iterator<E> iterator() {
			return new Itr();
    	}
返回了一个具体的迭代器实例。而ArrayList,LinkedList都是继承AbstractList的，因此自动具有了返回迭代器实例的功能。  

因此，Java种的容器类只负责元素的的维护就好了。访问就教给迭代器吧，Java做的如此完善，以至于我们都不必再自己写Iterator了。    