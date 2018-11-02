---
layout:     post
title:      "单例模式及getInstance()的用法"
subtitle:   "使用getInstance()方法的原因及作用"
date:       2018-11-02

author:     "kikyoar"
header-img: "img/post-bg-Java-version.jpg"
tags:
    - Java
---  

单例设计模式：对问题行之有效的解决方法，其实它是一种思想

**解决的问题**：

- 可以保证一个类在内存中的对象唯一性  
- 必须对于多个程序使用同一配置信息对象时，就保证该对象的唯一性  

**如何保证对象的唯一性？**  

- 不允许其他程序用new创建其他对象
- 在该类中创建一个本类实例
- 对外提供一个方法让其他程序可以获取该对象
 
**步骤**：

- 私有化该类的构造函数
- 通过new在本类中创建一个本类对象
- 定义一个公有的方法，将创建的对象返回  

**`单例模式`一般用于比较大，复杂的对象，只初始化一次，应该还有一个private的构造函数，使得不能用new来实例化对象，只能调用getInstance方法来得到对象，而getInstance保证了每次调用都返回相同的对象**   


**单例模式的的分类**  

- 饿汉式：  `类已加载，对象就已经存在了`

		  
		class Single
		{
		    private static Single s = new Single();
		    private Single(){}
		    public static Single getInstance(){
		        return s;
		    }
		
		}  
		
- 懒汉式：`类的延迟加载形式`   

	**类加载进来，没有对象，只有调用了getInstance方法时，才会创建对象**  
	
		class Single
		{
		    private static Single s2 = null;
		    private Single(){}
		    public static Single getInstance()
		    {
		        if (s2==null)
		        {
		            s2 = new Single();
		        }
		        return s2;
		
		    }
		}    


**举个栗子：**  

	public class SingleDemo {
	    public static void main(String[] args) {
	
	        Single ss =Single.getInstance();
	        //Single ss = Single.s;    //不可控
	        //Test t1 = new Test();
	        //Test t2 = new Test();
	        Test t1 = Test.getInstance();
	        Test t2 = Test.getInstance();
	        t1.setNum(20);
	        t2.setNum(10);
	        System.out.println(t1.getNum());
	        System.out.println(t2.getNum());
	    }
	}
	
	class Test{
	    private static Test t = new Test();
	    private Test(){}
	    public static Test getInstance(){
	        return t;
	    }
	    private int num;
	    public void setNum(int num){
	        this.num = num;
	    }
	    public int getNum(){
	        return num;
	    }
	}

**运行结果：**  

	10
	10  
	

详细解释一下：对象的实例化方法，也是比较多的，最常用的方法是直接使用new，而这是最普通的，如果要考虑到其它的需要，如单实例模式，层次间调用等等   

**getInstance方法。这是一个设计方式的代表，而不仅仅指代一个方法名**    

- new的使用:  
	如Object _object = new Object()，这时候，就必须要知道有第二个Object的存在，而第二个Object也常常是在当前的应用程序域中的，可以被直接调用的
 
-  GetInstance的使用:  
	在主开始时调用，返回一个实例化对象，此对象是static的，在内存中保留着它的引用，即内存中有一块区域专门用来存放静态方法和变量，可以直接使用，调用多次返回同一个对象  

**两者区别对照**:  

  * 大部分类(非抽象类/接口/屏蔽了constructor的类)都可以用new，new就是通过生产一个新的实例对象，或者在栈上声明一个对象，每部分的调用，用的都是一个新的对象   
  * getInstance是少部分类才有的一个方法，各自的实现也不同  
  * getInstance在单例模式(保证一个类仅有一个实例，并提供一个访问它的访问点)的类中常见，用来生成唯一的实例，getInstance往往是static的  

	  * 对象使用之前通过getInstance得到而不需要自己定义，用完之后不需要delete  
	  * new 一定要生成一个新对象，分配内存；getInstance() 则不一定要再次创建，它可以把一个已存在的引用给你使用，这在效能上优于new  
	  * new创建后只能当次使用，而getInstance()可以跨栈区域使用，或者远程跨区域使用。所以getInstance()通常是创建static静态实例方法的  

**总结**：

getInstance这个方法在单例模式用的甚多，为了避免对内存造成浪费，直到需要实例化该类的时候才将其实例化，所以用getInstance来获取该对象，至于其他时候，也就是为了简便而已，为了不让程序在实例化对象的时候，不用每次都用new，索性提供一个instance方法，不必一执行这个类就初始化，这样做到不浪费系统资源！`单例模式可以防止数据的冲突，节省内存空间`

