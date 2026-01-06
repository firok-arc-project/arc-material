---
title: 关于 JavaFX WebView 内调用 Java 耗时方法的线程阻塞问题
id: eaf8f445-9c9c-4f34-982e-af00c84bd5ed
createTimestamp: 2024-02-22T17:47:00+08:00
updateTimestamp: 2024-02-22T17:47:00+08:00
sortTimestamp: 2024-02-22T17:47:00+08:00
tags:
  - doc
  - tech
---

在 JavaFX WebView 中很多时候需要调用一些耗时 Java 方法, 直接在 JavaScript 代码中调用相关方法会导致 WebView 整个卡死.

比如下面的代码:

```java
public class Controller
{
    // 定义一个耗时方法
    public String call() throws Exception
    {
        Thread.sleep(5000);
        return "123";
    }
}

// 把 call() 放到 window 里面
var window = (JSObject) wevView.getEngine().executeScript("window");
window.putMember("controller", new Controller());
```

```javascript
// 下面每种写法都会导致 WebView 卡死
window['controller']?.call()

await window['controller']?.call()

await new Promise((resolve) => {
    resolve(call())
})
```

为此, 我们需要在 WebView 中对耗时方法进行一些处理.

处理思路是让被调用的 Java 方法尽快结束, 将耗时任务在子线程内完成而不阻塞 WebView 线程.

比较简单两种方式有:

1. 提供一个 callback 方法给 Java 上下文, 耗时任务子线程完成后 `Platform.runLater(() -> window.executeScript(callback))`
2. 耗时方法直接返回一个在子线程中运行的 `javafx.concurrent.Task<?>`, 由 JavaScript 上下文轮询此 `Task<?>` 实例的状态

第一种方式实现较为简单, 但是 callback 模式不太符合现代 JavaScript 开发模式, 会使代码变得支离破碎.

第二种方法可以使用 `Promise` 对代码进行封装, 观感会好很多.

下面是对第二种方法的一个示例实现:

```java
// 将耗时方法封装成 Task 实例
public Task<String> call()
{
    var task = new Task<String>() {
        @Override
        public String call()
        {
            try { Thread.sleep(5000); } catch (Exception _) { }
            return "finish: " + new Date().toLocaleString();
        }
    };
    new Thread(task).start(); // 让子线程运行任务
    return task; // 当前线程立刻返回
}
```

```javascript
/**
 * 将 Java 方法返回的 Task<?> 实例封装成 Promise,
 * 轮询其状态并进行适当处理
*/
async function promiseOf(task)
{
    return new Promise((resolve, reject) => {
        const threadTask = setInterval(() => {
            switch (task.getState().name())
            {
                case 'READY':
                case 'SCHEDULED':
                case 'RUNNING':
                    break
                case 'SUCCEEDED':
                    clearInterval(threadTask)
                    resolve(task.getValue())
                    break
                case 'FAILED':
                    clearInterval(threadTask)
                    reject(task.getException())
                    break
                case 'CANCELLED':
                    clearInterval(threadTask)
                    reject(null)
                    break
            }
        }, 500) // 轮询间隔看个人喜好定, 这里随便写个 500ms
    })
}

// 然后像下面这样调用就不再会导致 WebView 阻塞
const result = await promiseOf(window['controller']?.call())
```
