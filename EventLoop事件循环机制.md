- [1.EventLoop](#1eventloop)
- [2.Promise](#2promise)
- [3.await and async](#3await-and-async)
# 1.EventLoop
游览器加载页面，除了开辟堆和栈内存外，还要创建两个队列
  + WebAPI ：任务监听队列--监听异步任务是否可以执行了
  + EventQueue ：事件/任务队列

首先 ：当主线程自上而下执行代码的过程中，如果遇到”异步“代码，会把异步任务放到WebAPI中去监听
  + 游览器开辟新的线程去监听异步任务是否可以执行
  + 新线程不会阻碍主线程的渲染，他会继续向下执行同步代码

其次 ：但异步任务被监测到可以执行了，不会立即执行。而是把其放到EventQueue中排队等待执行
  + 根据微任务还是宏任务，放在对应队列中
  + 谁先进来排队，堆在各自队列的最前面
    PS：对于定时器来说，设定一个等待时间，到时间后并不一定会立即执行

最后 ：当同步代码（同步宏任务）执行完毕，主线程空闲下来，此时会去EventQueue中把正在排队的异步任务，按照顺序取出来执行
  + 异步的微任务优先级比宏任务高，不论其任务是先放入的，还是后放入的，只要有执行的异步微任务，永远先执行他
  + 同样级别的任务，谁先放入，谁先执行
  + 主线程把任务拿到栈中由主线程执行，所以只要拿出来的任务没有执行完，也就不会再去拿其他任务执行

# 2.Promise
+ 情况1 ：p.then(onfulfilled, onrejected), 已知实例p的状态和值，也不会立即把 onfufilled/onrejected 执行，而是创建”异步微任务“。任务先进入WebAPI中，发现状态是成功，则onfufilled可以被执行，把其再移入EventQueue中排队等候。
```
    let p = new Promise( resolve => resolve(10); )
    p.then(value => console.log('successfully', value))
    console.log(1)
```

+ 情况2 ：如果不知道实列p的状态，则先把unfulfilled/onrejected存储起来（理解为：进入WebAPI去监听，只有知道实例的状态，才可以执行）。resolve/reject执行，立即修改实例的状态和值，也决定了WebAPI中监听的方法（onfufilled/onrejected）哪一个去执行。把任务移到EventQueue中，异步微任务队列。等待其他同步代码执行完毕，再拿出来执行。
```
    let p = new Promise( resolve => {
        setTimeout(() => {
            resolve(10);
            console.log(p,2)
        }, 1000)
    })
    p.then(value => console.log('successfully', value))
    console.log(1)
```
# 3.await and async
  遇到await
  + 会立即执行其后面的代码，返回promise实例是否成功（如果不是promise实例也会转变为promise实例）
  + 会把当前上下文中，await下面代码当作异步任务放入WebAPI中监听。
  + 只有promise实例状态是成功的，才可以把异步任务放入EventQueue中排队，等待执行

