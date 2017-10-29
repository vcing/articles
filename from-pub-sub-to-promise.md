看过很多的Promise规范的实现，近来研究了一些设计模式在JavaScript中的应用和实践，其中在JS中应用的最多的就应属发布/订阅模式(Pub-Sub)了。

而Promise的很多实现原理又与发布/订阅模式有着千丝万缕的联系。我将尝试着从一个最简化的Promise实现中追寻发布/订阅模式的痕迹。

# 1 准备工作
## 1.1 发布/订阅模式

首先用一个最简单的例子来建立一个基本的概念。上学的时候我们都干过一件事：当你正准备在课上睡觉之前，你会跟你的同桌说一声“老师来了叫我”。这就是一个最基本的发布订阅模式了，你作为**订阅者**向你的同桌**发布者**订阅了**老师来了**这个事件。

这是一个最简单的一对一订阅模型，实际上事情可能还会变得更有趣一些。也许你前面的同学也向你的同桌订阅了老师来了这个事情，或者你只向同桌订阅了班主任来了和教导主任来了这两个事件，因为其他老师并不care你。最后，当你睡醒的时候，你便自动在同桌那取消了这个事件订阅。

通常来说，发布/订阅模式具有两个非常实用的特点-异步和解耦
1. 异步:在我完成了订阅之后我就可以安心的睡觉了，不需要时刻盯着老师或者不停的问同桌老师来了没。
2. 解耦:无论我是否真的睡着了还是在下面偷偷玩手机，同桌并不关系我在做什么，而我也不管同桌是真的在听课还是也在看小说。只要他能准确的提醒我老师来了，那么我就是安全的。

## 1.2 Promise/A+规范核心

[Promise/A+](Promises/A+)([译文版](【翻译】Promises/A+规范-图灵社区))是一种异步模式编程规范。燃鹅,它只说了这东西应该是啥样的，有哪些行为，并没有给出一个标准实现。并且，规范中很多条目也都是在描述一些边界情况和行为细节。其实，我理解的整个Promise的实现就是围绕着than函数而展开的。

## 1.3 Promise的最简化核心实现

那么现在，让我们先暂且放下种种边界情况，异常处理，假设只有resolve这一条路(pending -> fulfilled)，这个时候Promise还剩下什么。

```
function Promise(fn) {
    var state = 'pending',
        value = null,
        callbacks = [];
    this.then = function (onFulfilled) {
        return new Promise(function (resolve) {
            handle({
                onFulfilled: onFulfilled || null,
                resolve: resolve
            });
        });
    };
    function handle(callback) {
        if (state === 'pending') {
            callbacks.push(callback);
            return;
        }
        //如果then中没有传递任何东西
        if(!callback.onFulfilled) {
            callback.resolve(value);
            return;
        }
        var ret = callback.onFulfilled(value);
        callback.resolve(ret);
    }
    
    function resolve(newValue) {
        if (newValue && (typeof newValue === 'object' || typeof newValue === 'function')) {
            var then = newValue.then;
            if (typeof then === 'function') {
                then.call(newValue, resolve);
                return;
            }
        }
        state = 'fulfilled';
        value = newValue;
        setTimeout(function () {
            callbacks.forEach(function (callback) {
                handle(callback);
            });
        }, 0);
    }
    fn(resolve);
}
```

在这就不来一步一步的实现了，直接上代码。这段代码来自[Promise原理解析](https://github.com/mengera88)。这也是我目前看过的最精良的Promise核心实现了。

粗略的看一眼，一共50行代码，Promise内主要分成了三个函数 `than`, `handle` 和 `resolve`。不管你一眼看没看懂，反正我是花了2天时间去充分理解他们的细节，内部关系，还有意图。当然，这其中包含了远远不止三对的发布/订阅模式关系。接下来，我们以函数为单位逐个浅析一下他们各自的功能。

# 2 对比剖析
## 2.0 场景和术语定义
## 2.1 then-订阅事件
## 2.2 resolve-发布事件
## 2.3 Promise本体-调度中心
## 2.4 链式Promise

# 3 感悟
## 3.1 Promise没有实现的发布/订阅模式部分
## 3.2 发布/订阅模式的特点
## 3.3 对于Promise中的映射
## 3.4 一些实践

# 4 总结

# 5 参考文献
