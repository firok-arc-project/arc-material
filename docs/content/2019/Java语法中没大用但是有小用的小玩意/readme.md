---
title: Java 语法中没大用但是有小用的小玩意
id: bfe86253-fb7e-4a73-9682-ecb332a9648f
createTimestamp: 2019-12-28T10:19:00+08:00
updateTimestamp: 2019-12-28T10:19:00+08:00
sortTimestamp: 2019-12-28T10:19:00+08:00
tags:
  - doc
  - tech
---

## 泛型参数上的 `&`符号 (多重界限泛型)

```java
public interface IValue // 示例接口
{
    int getValue();
}
public static class A // 示例基类
{
    int getA() { return 123; }
}
public static class B extends A implements IValue // 示例子类
{
    @Override
    public int getValue() { return 456; }

    int getB() { return 789; }
}
public static <T extends A & IValue> void test(T obj) // 示例泛型方法
{
    System.out.println("A: " + obj.getA());
    System.out.println("Value: " + obj.getValue());
}

public static void main(String...args)
{
    test(new B());
}
```

使用 `<T extends A & IValue>` 这样的写法会限制形参类型,
省掉一些 `instanceof` 的功夫

## 标签

Java 没有 `goto` 语句, 但是提供了 **语句标签** (label) 辅助流程控制

### 退出语句块

语句标签可以用来标记/退出一个语句块, 有点像在方法内部做 `return`

```java
public static void main(String...args)
{
    BLOCK: // 语句标签
    {
        System.out.println(1);
        System.out.println(2);
        if(true) break BLOCK;
        System.out.println(3);
    }
}
```

### 循环控制

标签还可以用来控制多层嵌套循环的执行

```java
public static void main(String...args)
{
    FOR_X: for(int x=0;x<10;x++)
    {
        FOR_Y: for(int y=0;y<10;y++)
        {
            FOR_Z: for(int z=0;z<10;z++)
            {
                if(x==5) break FOR_Y;
                else if(x==6) continue FOR_X;
            }
        }
    }
}
```
