---
layout: post
title: 代码证明内特朗箱子悖论！
tag: 代码实现
date: 2020-2-2
category: Technology blog
---
概率论真的很奇妙，其中内特朗箱子悖论让人一时比较难以理解，因此想利用代码创造大量例子证明

![img](https://img-blog.csdnimg.cn/2019050113345614.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDQzOTA4NQ==,size_16,color_FFFFFF,t_70)![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

图片来自知乎https://www.zhihu.com/question/26435542/answer/112876996?utm_source=wechat_session&utm_medium=social&utm_oi=855348107909664768

```java
import java.util.LinkedList;

public class boxQuestion {
	public static void main(String[] args) {
		float num1 = 0;
		float num2 = 0;
		test(num1,num2);
		
		}
	public static void test(float num1,float num2) {
		for(int i = 0; i < 10000;i ++) {
		LinkedList<Object> list =new LinkedList<Object>();
		boxFirst b1 = new boxFirst();
		boxSecond b2 = new boxSecond();
		boxThird b3 = new boxThird();
//		System.out.println(b1.box);
//		System.out.println(b2.box);
//		System.out.println(b3.box);
		list.add(b1.box);list.add(b2.box);list.add(b3.box);
//		System.out.println(list);
		LinkedList<Object> boxChoose = (LinkedList) list.get((int) (Math.random() * 3));
//		System.out.println(boxChoose);
		int temp = (int) (Math.random() * 2);
		String str1 = (String) boxChoose.get(temp);
//		System.out.println(str1);
		boxChoose.remove(temp);
		String str2 = (String) boxChoose.pop();
//		System.out.println(str2);
		String Regex = "red";
//		System.out.println(str1.matches("red+[1-3]"));
		if(str1.matches("red+[1-3]") == true){
			num1 ++;
			if(str2.matches("red+[1-3]")) {
			num2 ++;
			}	
		}
	}
		System.out.println("100000中第一个是红球的次数:" + num1);
		System.out.println("100000中在第一个是红球的概率下第二个也是红球的次数:" + num2);
		float answer = num2/num1;
		System.out.println("结果是：" + answer);
	}
		
	public static class boxFirst{
		LinkedList<String> box =new LinkedList<String>();
		public boxFirst() {
			box.add("red1");
			box.add("red2");
		}
	}
	public static class boxSecond{
		LinkedList<String> box =new LinkedList<String>();
		public boxSecond(){
			box.add("red3");
			box.add("blue3");
		}
	}
	public static class boxThird{
		LinkedList<String> box =new LinkedList<String>();
		public boxThird(){
			box.add("blue1");
			box.add("blue2");
		}
	}
	
}
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

运行结果

```java
100000中第一个是红球的次数:49759.0
100000中在第一个是红球的概率下第二个也是红球的次数:33169.0
结果是：0.66659296
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```java
100000中第一个是红球的次数:50210.0
100000中在第一个是红球的概率下第二个也是红球的次数:33525.0
结果是：0.6676957
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

```java
100000中第一个是红球的次数:49834.0
100000中在第一个是红球的概率下第二个也是红球的次数:33488.0
结果是：0.671991
```

![点击并拖拽以移动](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)