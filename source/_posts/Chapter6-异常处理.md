---
title: Chapter6 异常处理
excerpt: 常见异常、异常处理机制try-catch-finally与throws，自定义异常类型、手动抛出异常
tags:
  - java
categories:
  - Java笔记
banner_img: /img/dog.png
index_img: /img/post/Java/java_logo.png
abbrlink: 6a6b852d
date: 2020-10-22 22:23:55
updated: 2020-10-22 22:23:55
subtitle:
---
## 6.1 异常概述与异常体系结构
### 1. 异常类型：
* Error：Java虚拟机无法解决的严重问题, 一般不编写针对性代码处理
  * 栈溢出 StackOverflowError
  * 堆溢出 OOM
* Exception:其它因编程错误或偶然的外在因素导致的一般性问题, 可以使
用针对性的代码进行处理
  * 空指针访问
  * 试图读取不存在的文件
  * 网络连接中断
  * 数组角标越界

### 2. 异常体系结构
分类：
* 编译时异常
* 运行时异常

### 3. 常见异常：

```java
java.lang.Throwable
 * 		|-----java.lang.Error:一般不编写针对性的代码进行处理。
 * 		|-----java.lang.Exception:可以进行异常的处理
 * 			|------编译时异常(checked)
 * 					|-----IOException
 * 						|-----FileNotFoundException
 * 					|-----ClassNotFoundException
 * 			|------运行时异常(unchecked,RuntimeException)
 * 					|-----NullPointerException
 * 					|-----ArrayIndexOutOfBoundsException
 * 					|-----ClassCastException
 * 					|-----NumberFormatException
 * 					|-----InputMismatchException
 * 					|-----ArithmeticException
```

## 6.2 异常处理机制
### 1. `try-catch-finally`
代码格式：
```java
try{
  //可能有问题的代码
}
catch(错误类型 变量1){
  // System.out.println(变量1.getMessage())
}
catch(错误类型 变量2){
  // 变量2.printStackTrace();
}
finally{
  // 一定会执行的代码
}
```
1. 用于处理编译时异常
2. `finally` 部分是可选的
3. `catch` 中的错误类型存在子父类关系时，子类在上
4. 常用异常处理方式：`getMessage()`、`printStackTrace()`
5. `try` 结构内定义的变量作用域仅在 `try` 结构内

### 2. `throws`
代码格式：
```java
throws ErrorType1, ErrorType2,···
```

示例：
```java
public class ExceptionTest {
	
	
	public static void main(String[] args){
		try{
			method();
			
		}catch(IOException e){
			e.printStackTrace();
		}	
	}
	
	public static void method() throws FileNotFoundException,IOException{
    // 方法体
    // ******
	}
	
}

```
1. 写在方法声明处(大括号前)，方法执行出现异常，则在异常处生成异常对象，满足throws后类型时则抛出
2. 只是向上层抛出，并未真正处理异常，通常配合`try-catch-finally`使用

### 3. 重写方法与异常抛出
1. 子类重写方法，抛出的异常不大于父类异常
2. 父类中方法未使用 `throws` 则子类也不能使用，此时只能使用`try-catch-finally`

## 6.3 手动抛出异常
1. 格式：
   ```java
   throw new 异常类型();
   ```
2. 抛出的异常必须是Throwable或其子类的实例

## 6.4 自定义异常类型
1. 继承于现有异常结构，`RuntimeException`、`Exception`
2. 提供全局常量：`seriaVersionUID`
3. 代码示例：
   ```java
   class MyException extends Exception {
      static final long serialVersionUID = 13465653435L;
      private int idnumber;

      public MyException(String message, int id) {
      super(message);
      this.idnumber = id;
      }

      public int getId() {
      return idnumber;
      }
   }
   ```


   注意例题EcmDef