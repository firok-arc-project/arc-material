---
title: 实例化注解对象
id: 69dc97f3-9749-4535-afbc-5b485b5f27b3
createTimestamp: 2023-10-24T16:35:00+08:00
updateTimestamp: 2023-10-24T16:35:00+08:00
sortTimestamp: 2023-10-24T16:35:00+08:00
tags:
  - doc
  - tech
---

对的, 用代码实例化注解对象.

```java
public class AnnotationInstanceTests
{
    @Retention(RetentionPolicy.RUNTIME)
    @Target(ElementType.METHOD)
    public @interface TestAnno
    {
        String v() default "default-value";
    }

    public static TestAnno createAnnoInstance(String value)
    {
        return new TestAnno() {

            @Override
            public String v()
            {
                return value;
            }

            @Override
            public Class<TestAnno> annotationType()
            {
                return TestAnno.class;
            }
        };
    }

    @Test
    @TestAnno
    public void testInstance() throws Exception
    {
        var ta1 = createAnnoInstance("value1");
        var ta2 = createAnnoInstance("value2");
        System.out.println(ta1.v());
        System.out.println(ta2.v());

        var ta = AnnotationInstanceTests.class
                .getMethod("testInstance")
                .getAnnotation(TestAnno.class);
        System.out.println(ta.v());

        var taRandomNumber = new TestAnno() {
            private final Random rand = new Random();
            private int times = 0;
            @Override
            public String v()
            {
                times++;
                return times + " = " + rand.nextInt();
            }

            @Override
            public Class<? extends Annotation> annotationType()
            {
                return TestAnno.class;
            }
        };
        System.out.println(taRandomNumber.v());
        System.out.println(taRandomNumber.v());
        System.out.println(taRandomNumber.v());
        System.out.println(taRandomNumber.v());
    }
}
```

然后你会得到:

```
value1
value2
default-value
1 = -655248413
2 = -1664205039
3 = 734830359
4 = 36076286
```

哇, 真是神奇, 真是有趣 🤣👉💻

> 参见 [How to create an instance of an annotation](https://stackoverflow.com/a/16303007/9907751)
>
