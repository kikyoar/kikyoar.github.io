---
layout:     post
title:      "Java数组的三种打印方式"
subtitle:   "Java数组声明的几种方式"
date:       2018-08-06
author:     "kikyoar"
header-img: "img/post-bg-Java-version.jpg"
tags:
    - Java
---  


## Java数组声明几种方式

- 第一种(声明并初始化)：

		数据类型[] 数组名={值,值,...};

		eg：int[] a = {1,2,3,4,5,6,7,8};
    
- 第二种(声明后赋值)：
          
		数据类型[] 数组名 = new 数据类型[数组长度];
		数组名[下标1]=值;数组名[下标2]=值;.....

		eg：String[] a =new String[4];
		a[0]="hello";
		a[1]="world";
 		....
    
- 第三种()：
		
		数据类型[] 数组名=new 数据类型[]{值,值,...};
		
		
## Java数组的三种打印方式  

**1、数组的输出的三种方式**  

**一维数组：**

`定义一个数组   int[] array = {1,2,3,4,5};`

(1)传统的for循环方式
	
	for(int i=0;i<array.length;i++)
	{
	      System.out.println(a[i]);
	}


(2)for each循环

	for(int a:array)
	    System.out.println(a)  
	    
(3)利用Array类中的toString方法  

调用Array.toString(a)，返回一个包含数组元素的字符串，这些元素被放置在括号内，并用逗号分开  


	int[] array = {1,2,3,4,5};
	System.out.println(Arrays.toString(array));  
	
输出：[1, 2, 3, 4, 5]  
说明：System.out.println(array);这样是不行的，这样打印是的是数组的首地址


**二维数组：**  

对于二维数组也对应这三种方法，定义一个二维数组：  

	 int[][] magicSquare =
	    	 {
	    		 {16,3,2,13},
	    		 {5,10,11,8},
	    		 {9,6,7,3}
	    	 };  
	    	 
**Java实际没有多维数组，只有一维数组，多维数组被解读为"数组的数组"，例如二维数组magicSquare是包含{magicSquare[0]，magicSquare[1]，magicSquare[2]}三个元素的一维数组，magicSqure[0]是包含{16,3,2,13}四个元素的一维数组，同理magicSquare[1]，magicSquare[2]也一样**  

(1)传统的for循环方式  

	for(int i=0;i<magicSquare.length;i++)
	     {
	    	 for(int j=0;j<magicSquare[i].length;j++)
	    	 {
	    		 System.out.print(magicSquare[i][j]+" ");
	    	 }
	    	 System.out.println();	//换行
	     }

(2)for each循环  

	for(int[] a:magicSquare)
	     {
	    	 for(int b:a)
	    	 {
	    		 System.out.print(b+" ");
	    	 }
	    	 System.out.println();//换行
	     }  
	     
(3)利用Array类中的toString方法  

	for(int i=0;i<magicSquare.length;i++)
	    System.out.println(Arrays.toString(magicSquare[i]));


	    	 
