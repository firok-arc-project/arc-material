---
title: 享元二三事
id: 3e674769-d522-49c8-9967-b61bc781b28b
createTimestamp: 2021-05-09T22:56:00+08:00
updateTimestamp: 2021-05-09T22:56:00+08:00
sortTimestamp: 2021-05-09T22:56:00+08:00
tags:
  - doc
  - tech
---

## 前言

个人对于享元模式的理解是: 将 **数据** 和 **行为** 相较一般的面向对象设计更加分离.

## 一般面向对象设计

一般的面向对象设计的基础是 **类(class)** ,
类中包含的 **字段(field)** 和 **方法(method)** 就代表了 **数据** 和 **行为** .
调用 **实例(instance)** 的方法时,
会由于子类实现不同、数据不同、参数不同进而展现出不同的行为.

下面是一个一般面向对象设计的实例

```java
interface ISpeak { void speak(); }
class Human implements ISpeak {
    public int age;
    @Override
    public void speak() {
        System.out.println("I'm "+age+"years old");
    }
}
class Dog implements ISpeak {
    public int barkTimes;
    @Override
    public void speak() {
        for(int i=0; i<barkTimes; i++)
            System.out.println("woof!");
    }
}

List<ISpeak> listSpeakers = new ArrayList<>();
Human human1 = new Human(); human1.age = 10;
Human human2 = new Human(); human2.age = 20;
Human human3 = new Human(); human3.age = 30;
Dog dog = new Dog(); dog.barkTimes = 3;

listSpeakers.add(human1);
listSpeakers.add(human2);
listSpeakers.add(human3);
listSpeakers.add(dog);
for(ISpeak speaker : listSpeakers)
    speaker.speak();
```

不难理解, 上面的代码定义了 `ISpeak` 接口,
并且在类 `Human` 和 `Dog` 中提供了具体实现.
当遍历调用 `listSpeakers` 中的对象方法时,
将会由于实现不同、数据不同而输出不同的内容.

## 有什么不好的吗

一般面向对象设计对于大部分情况能都较好的实现所需业务,但是当对象数量特别大 / 对象本身比较重型时,  就会出现一些问题.

> "重"指占用的内存总量非常多, 或结构非常复杂

比如, 在上面的示例代码中,如果某模块需要调用某个 `Human` 实例的 `speak` 方法,会有下面两种情况:

1. 对象不在内存
   需要将这个 `Human` 对象实例重新构造出来,才能继续向下执行业务逻辑.
2. 对象在内存
   找到这个 `Human` 实例,
   执行业务逻辑.

第一种情况下, 只要对象数量到达一定程度 / 对象数据本身重到一定程度,
就会导致系统在序列化 / 反序列化过程浪费大量的性能.

第二种情况看起来非常简单直观, 但是不难想象,
使用这种方法也会在对象数量过多 / 对象重型时造成内存的大量占用.

## 那么应该怎么办

应该发现, 上述不论哪种情况,
都是当 **数据** 的量级到达一定程度导致的问题.
那么此时, 使用享元模式, 将数据和行为切分开,
就可以对数据本身进行优化了.

## 听起来很棒, 具体怎么做呢

下面是一个享元模式的实例

```java
interface ISpeak
{
    void speak(Map<String,Object> params);
}
class Human implements ISpeak
{
    static final Human instance = new Human();

    @Override
    public void speak(Map<String,Object> data)
    {
        try
        {
            int age = Integer.parseInt( String.valueOf(data.get("age")) );
            System.out.println("I'm "+age+" years old");
        }
        catch (Exception e)
        {
            System.out.println("I can't tell how old I am");
        }
    }
}
class Dog implements ISpeak
{
    static final Dog instance = new Dog();

    @Override
    public void speak(Map<String,Object> data)
    {
        try
        {
            int barkTimes = Integer.parseInt( String.valueOf(data.get("barkTimes")) );
            for(int i=0; i<barkTimes; i++)
                System.out.println("woof!");
        }
        catch (Exception e)
        {
            System.out.println("...");
        }
    }
}

Consumer<Map<String,Object>> funHowToSpeak = (data) -> {
    String typeSpeaker = String.valueOf(data.get("type"));
    switch (typeSpeaker)
    {
        case "human": Human.instance.speak(data); break;
        case "dog": Dog.instance.speak(data); break;
        default: throw new RuntimeException("Can't find a kind of speaker: "+typeSpeaker);
    }
};

List<Map<String,Object>> listSpeakDatas = new ArrayList<>();
Map<String,Object> speaker1 = new HashMap<>();
speaker1.put("type","dog"); speaker1.put("barkTimes",3);
Map<String,Object> speaker2 = new HashMap<>();
speaker2.put("type","human"); speaker2.put("age",20);
Map<String,Object> speaker3 = new HashMap<>();
speaker3.put("type","cat"); speaker3.put("meowTimes",3);

listSpeakDatas.add(speaker1);
listSpeakDatas.add(speaker2);
listSpeakDatas.add(speaker3);

for(Map<String,Object> data : listSpeakDatas)
    funHowToSpeak.accept(data);

```

代码相较一般面向对象设计长了很多, 但是并不困难, 拆成若干部分来看.

### 行为原型

首先, 原本的代表行为原型的接口发生了变化.
现在需要接收一个 **数据对象** .

```java
interface ISpeak
{
    void speak(Map<String,Object> params);
}
```

具体会由于实际相关的

> 为了方便, 这里的数据对象是使用了一个 `Map<String,Object>` 表示
> 实际上数据对象可以使用任何形式
> 如使用POJO, 或拆分成多个参数等

### 行为对象和寻找行为对象的方式

紧接着是两个改写过的 **行为对象**

```java
class Human
{
    static final Human instance = new Human();
    ...
}
class Dog
{
    static final Dog instance = new Dog();
    ...
}
```

此时, **每个** 行为对象的实例代表的是 **一类** 实体.
对于 **每种** 实体, 也只需要 **一个** 行为对象实例. (所以这里构造了一个单例)

后面的 `funHowToSpeak` 方法包含了两部分,
一是先通过某种方式, 确定某次行为执行时的主体类型, 找到对应的行为对象实例;
二是交付数据并执行.

## 变式

示例中, 用于确定主体类型的 `type` 是跟具体的数据一并放到 `Map<String,Object>` 中的, 这不是强制的.
只需要能够通过某种方式确定一个主体, 并且通过某种方式交付数据即可.

```java
int[] arraySpeakerTypes = new int[] { 1, 2, 3, };

Function<Integer,ISpeak> funFindSpeaker = (type) -> {
    switch(type)
    {
        case 1: return Dog.instance;
        case 2: return Human.instance;
        default: return null;
    }
};

Function<Integer,Map<String,Object>> funFindSpeakData = (type) -> {
    File file = null;
    String paramName = null;
    switch(type) {
        case 1:
            file = new File("./dog.txt");
            paramName = "barkTimes";
            break;
        case 2:
            file = new File("./human.txt");
            paramName = "age";
            break;
        default: break;
    }

    Map<String,Object> ret = new HashMap<>();
    if(file != null)
    {
        String paramValue = null;
        try(Scanner in = new Scanner(file))
        {
            paramValue = in.readLine();
        }
        catch(Exception ignored) { }

        ret.put(paramName, paramValue);
    }
    return ret;
};

for(int speakerType : arraySpeakerTypes)
{
    ISpeaker speaker = funFindSpeaker.apply(speakerType);
    Map<String,Object> speakData = funFindSpeakData.apply(speakerType);
    speaker.speak( speakData );
}
```

上面的代码就展示了一种即时构造行为数据的方式.

## 进一步的优化

上面的例子中对享元模式的是通过一个写死的 **函数(function)** 实现的.
实际上更合适的方式是先实现一套 **注册表机制** 用于发现行为对象,比如使用一个 `Map<Type,Instance>` 来记录类型到主体的映射等.

> 当享元模式的对象之间存在一定父子实现关系时
> 使用享元模式还可以享受到来自于面向对象的优势

由于行为被拆分出来,
相关的数据也 **不一定需要** 一直放在内存中, 可以随取随用;
且不同行为所需的不同数据也不一定需要同时序列化 / 反序列化,  (比如, 执行说话行为时不需要吃饭行为的数据)
可以仅当执行某行为时才构造 **执行此行为所需的数据** .

当然, 所有的数据也可以按照某种方式进行 **缓存** 以提升运行速度.

进一步, 原本"分散"到各个对象中的数据现在被集中起来, 就可以按照某种方式 **压缩** .
代表行为主体类型的值可以压缩, 行为所需的数据也可以进行压缩.

## 比较

| 对比             | 一般面向对象模式                                    | 享元模式               |
| ---------------- | --------------------------------------------------- | ---------------------- |
| 发现机制         | 直接发现                                            | 间接发现               |
| 某种行为所需数据 | 储存在对象中, 或通过形参交付                        | 使用专用结构储存       |
| 所有数据         | 同时储存在内存中, 在需要时一起进行序列化 / 反序列化 | 不一定需要同时置于内存 |
| 内存占用         | 高                                                  | 低                     |
| 性能损耗         | 低                                                  | 高                     |

> 其实在发现机制这一点上, 当我们调用某个对象的某方法时
> 在JVM中是根据 **引用(reference)** 确定了相关对象的类型
> 然后找到了相关的方法和数据
> 某种意义上跟享元模式机制非常类似

> 确实, 使用一般的面向对象模式
> 也可以对数据存储进行类似于使用享元模式时的优化
> 比如某个对象某一时刻只持有部分自身数据等
> 但是这样的话往往每个子类都需要单独维护一套相关机制
> 而使用享元模式则可以集中对数据进行维护

> 比较来说, 享元模式是通过时间换空间的一种做法
> 但是有合理的组织结构的话, 使用享元模式并不会造成太大的性能损耗
> 至少这部分性能损耗是完全固定且完全可以接受的
>
