---
title: Java Sneaky Throws Exception
date: 2019-05-18 11:24:02
tags:
- java
categories:
- 软件开发
---

![](http://ww1.sinaimg.cn/mw690/006cgBSrly1g35nd2ujltj30m80dcmye.jpg)

<!--more-->

### 相关概念
`Sneaky Throw` 允许抛出检查异常，但无需在方法签名上`throws`或`try catch`该检查异常
在JVM层面不管方法是否有throws都可以抛异常，不区分检查及非检异常，检查异常是编译器的约束
`Sneaky Throw` 利用Java8的类型推断机制，把检查异常转为非检查异常，绕过编译器

Java8语言规范中的定义

	A bound of the form “throws α” is purely informational:
	it directs resolution to optimize the instantiation of “α” so that,if possible, 
	it is not a checked exception type. (…)

	Otherwise, if the bound set contains “throws αi”, and the proper upper bounds of “αi” are, 
	at most, Exception, Throwable, and Object, then Ti = RuntimeException.


大意是 如`<T extends Throwable>` 若T不能推断出一个具体的检查异常类型，则T会被推断为`RuntimeException`

### 使用场景
`Sneaky Throw` 可以避免检查异常的`try catch`样板代码，可用于不能抛出异常的地方，如lambda表达式中或`Runnable`类似的接口

### 使用方法
1. 实现一个util方法，利用泛型类型推断，按java规范，E被推断为`RuntimeException`, 重新抛出异常
```java
public static <E extends Throwable> void sneakyThrow(Throwable e) throws E {
    throw (E) e;
}
```
2. 测试一下，把IOException转为非检查异常
```java

//无需throws try catch 编译通过
public static void throwsSneakyIOException() {
    sneakyThrow(new IOException("sneaky"));
}

```

### 注意事项
由于使用`Sneaky Throw` 后，就不能try catch具体检查异常进行异常处理了，需要谨慎使用
```java
//编译器报错 Exception IOException is never thrown
try {
    throwsSneakyIOException();
} catch (IOException ex) {
    ex.printStackTrace();
}
```

### lombok @SneakyThrows
可以使用lombok`@SneakyThrows`简化`Sneaky Throw`代码
#### example
```java
public class SneakyThrowsExample implements Runnable {
  @SneakyThrows(UnsupportedEncodingException.class)
  public String utf8ToString(byte[] bytes) {
    return new String(bytes, "UTF-8");
  }
  
  @SneakyThrows
  public void run() {
    //无线try catch
    throw new Throwable();
  }
}
```
#### 实现原理
以上example代码注解处理后
```java
public class SneakyThrowsExample implements Runnable {
  public String utf8ToString(byte[] bytes) {
    try {
      return new String(bytes, "UTF-8");
    } catch (UnsupportedEncodingException e) {
      throw Lombok.sneakyThrow(e);
    }
  }
  
  public void run() {
    try {
      throw new Throwable();
    } catch (Throwable t) {
      // 具体转换方法
      throw Lombok.sneakyThrow(t);
    }
  }
}
```
[Lombok.java](https://github.com/ComponentsTeam/Lombok/blob/master/src/core/lombok/Lombok.java)
```java
public class Lombok { 
    public static RuntimeException sneakyThrow(Throwable t) {
		if (t == null) throw new NullPointerException("t");
		Lombok.<RuntimeException>sneakyThrow0(t);
		return null;
	}
	
    //利用类型推断进行转换
	@SuppressWarnings("unchecked")
	private static <T extends Throwable> void sneakyThrow0(Throwable t) throws T {
		throw (T)t;
	}
}
```

### 转换lambda中的检查异常
1.实现一个unchecked方法，代理会抛检查异常的业务方法，try catch该业务方法，捕获到异常后，调用`sneakyThrow`异常转换方法，重新抛出转换后的非检查异常
2.在lambda中调用上述unchecked方法
具体实现见[Sneakily Throwing Exceptions in Lambda Expressions in Java](https://4comprehension.com/sneakily-throwing-exceptions-in-lambda-expressions-in-java/)

### 参考
[“Sneaky Throws” in Java](https://www.baeldung.com/java-sneaky-throws)
[Lombok @SneakyThrows](https://projectlombok.org/features/SneakyThrows)


