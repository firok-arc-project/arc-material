---
title: 高版本 Java 中 `switch` 表达式对于 `null` 值的处理
id: 496be652-e12f-444e-be17-b353b4fa3493
createTimestamp: 2025-06-30T15:10:00+08:00
updateTimestamp: 2025-06-30T15:10:00+08:00
sortTimestamp: 2025-06-30T15:10:00+08:00
tags:
  - doc
  - tech
---

`switch` 表达式语法在 Java 14 被引入, 随后 `switch` 表达式对 `null` 值的支持在 Java 17 被引入.

在老版本 Java 中对于这样的代码:

```java
enum TestEnum
{
    A, B, C, D;
}
TestEnum any = ...;

if(any != null)
{
    switch (any)
    {
        case A:
        {
            System.out.println("case A");
            break;
        }
        case B:
        {
            System.out.println("case B");
            break;
        }
        default:
        {
            System.out.println("case C or D");
            break;
        }
    }
}
else // 原始 switch 语句不支持传入 null, 需要专门做处理
{
    System.out.println("case null");
}

```

可以被简化为:

```java
switch (any)
{
    case A -> System.out.println("case A");
    case B -> System.out.println("case B");
    case null -> System.out.println("case null");
    default -> System.out.println("case C or D");
}
```

需要注意的是, 如果已知传入值可能为 `null`, 那么 `case null` 分支就 **不可省略**. 因为即使是写了 `default` 分支, 在没有写 `case null` 分支的时候, 向这个 `switch` 语句传入 `null` 值还是会引发 `NullPointerException`.


