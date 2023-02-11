# 协程 & 异步

## 推导式&生成器

### 推导式
`python`中通过`生成式 / 推导式`可以生成`list`, `dict`, `set`。注意没有元组推导式，对应的语法用作描述生成器。
```python
# 列表生成器
[x for x in range(100) if x % 2]

[x + y for x in '123' for y in 'abc']

# 词典生成器
{x: x**2 for x in range(10)}

{x for x in 'abracadabra' if x not in 'abc'}
```

### 生成器
生成器是一种按需生成数据的类，可以一边循环一边按照某种算法生成后续元素，而不是直接创建完整的元素集合，一般用于描述巨大的数据序列。访问方式有2种：
- 通过`next(gen)`访问下一个元素，
- 使用`for`循环遍历所有元素。

注意通过`for`遍历时无法获得生成器的返回值，若要获得返回值只能通过`next`遍历并捕获`StopIteration`异常。**所有包含`yield`关键字的函数称为`生成器函数`，返回结果是一种被称为*生成器*的迭代器**。
```python
# 生成器语法，gen是一个生成器
gen = (x * x for x in range(10))
# <generator object <genexpr> at 0x1022ef630>

# 所有包含yield的函数都是一个生成器，每次调用都是一个新生成器！！！
def fibonacci(n: int):
    pre, cur = 0, 1
    idx = 0
    while idx < n:
        yield cur
        pre, cur = cur, pre + cur
        idx += 1
    return 'finished'
```

### yield & send
`python`最初提供`yield`是用于定义生成器，所有包含`yield`的函数都是生成器，语法规则为：
- 函数执行到`yield`处时停止，并返回`yield`后的值；当再次通过`next`调用时从上次暂停的地方继续执行。当不能再次被`next`调用时抛出`StopIteration`异常。

增强型生成器都可以通过`send()`方法向`yield`语句发送数据，使用方式为：`var = yield ret`，语句对应的2个功能为：
- 在`yield`处暂停并返回`ret`
- 接受调用方通过`send()`发送的内容并赋值给`var`

需要注意的是，`send()`方法发动的数据会成为`yield`表达式的值，因此**必须等生成器(协程)在GEN_SUSPENDED状态才可以使用`send`发送数据**，否则会出现`TypeError`，`GEN_CREATED`状态只能使用`send(None)`。即协程启动前必须要预激活，有2种方式，二者作用完全相同：
- next(coro)
- coro.send(Node)

todo: 除此之外，增强型生成器(协程)还有`throw / close`方法用于抛出特定异常。待补充。


基于以下共性，增强型生成器一般认为是简单的协程：
- 都可以`yield`多次
- 具有多个入口
- 执行可以被挂起

唯一的区别是生成器不能控制在`yield`后在哪里继续执行，控制权总是转移到生成器的调用者。

```python
# 增强型生成器，可以被认为是简单的协程
def consumer():
    while True:
        arg = yield resp
        if not arg:
            return
        print(f'[consumer] consuming {arg}')

def producer(c):
    c.send(None)
    for idx in range(10):
        print(f'[producer] producing {idx}')
        resp = c.send(idx)
        print(f'[producer] receive {resp}')
```

### async & await
使用`async def`定义的函数称为`异步生成器函数 / 协程函数`（即使函数体不包含`await, async`），每次调用产生一个`异步生成器`，和`生成器`是一个`迭代器`类似，它是一个`异步迭代器`。通过协程函数生成的对象（异步生成器）是一个协程。

## 协程
协程`coroutine`是一种运行在单线程中的并发模型，本质可以认为单线程内的控制权交替，是单线程内的调度技巧。协程省去了线程切换的开销，通常情况下有更高的效率。
不同于线程切换，协程的切换是程序自身控制（通过`yield`），因为协程运行在同一个县城内，因此没有数据访问的竞争问题，自然也不需要锁。单线程协程无法利用多核资源，如果要充分利用多核资源需要同时借助多进程技术。

`协程函数`调用的结果是返回一个协程对象，不能直接调用，需要通过`asyncio.run(co)`运行。


| 状态 | 含义 |
|---|---|
| GEN_CREATED | 等待执行 |
| GEN_RUNNING | 正在执行 |
| GEN_SUSPENDED | 暂停(yield处) |
| GEN_CLOSED | 执行完毕 |





todo: pyinstrument性能分析



