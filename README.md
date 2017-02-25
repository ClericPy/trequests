# torequests  - v3.0.3

## Inspired by [tomorrow](https://github.com/madisonmay/Tomorrow). To make async-coding EASY & smooth, nothing to learn.(It fits Windows, Python 2/3 compatible)

## Give one way to use async functions easily & make asynchronous requests by tPool.

> Abandon Tomorrow library, but use original **concurrent.futures** by default. Because NewFuture class is a child class of Future, it can use as_completed function to get future object sequence in finish-time sorting.

## Changelog:

>2017-02-20 01:47:19. 

> Use Session for tPool, for fixing a **not close connection** bug, 
and make it faster than creating a new connection each requests. Then tPool can be closed
explicitly by tPool instance.close, this run session.close and ThreadPoolExecutor._shutdown
together.

> Add context usage for tPool as **with** statement, for making sure release connection pool
and shutdown the thread pool. **AsyncPool** renamed as **Pool**.

```python
with tPool() as trequests:
    print(trequests.get('http://qq.com').headers)
    # {'Cache-Control': 'max-age=60', 'Date': 'Sun, 19 Feb 2017 18:11:06 GMT', 'Content-Encoding': 'gzip', 'X-Cache': 'HIT from tianjin.qq.com', 'Server': 'squid/3.5.20', 'Content-Type': 'text/html; charset=GB2312', 'Connection': 'keep-alive', 'Vary': 'Accept-Encoding, Accept-Encoding, Accept-Encoding', 'Expires': 'Sun, 19 Feb 2017 18:12:06 GMT', 'Transfer-Encoding': 'chunked'}
# OR
tt = tPool()
tt.get('http://baidu.com')
tt.close()
# AND
import time
pp = Pool()
ss = pp.map(lambda x:time.sleep(1),range(10))
print(*ss)
pp.close() # non-essential, will shutdown Pool automatic.
# OR
pp = Pool()
a = pp.submit(lambda: time.clock())
b = pp.submit(lambda x: time.clock(), 3)
c = pp.submit(lambda *args: time.clock(),1,2,3)
print([a.x,b.x,c.x])
```

> **timeout_return** arg can be set with a function for now, such as `lambda a,b,**c: (a,b,c)`

```python
trequests = tPool()
print(trequests.get('http://qq.com',timeout=.001,fail_return=lambda a,b,**c: (a,b,c)).x)
# ('http://qq.com', ConnectTimeout(MaxRetryError("HTTPConnectionPool(host='qq.com', port=80): Max retries exceeded with url: / (Caused by ConnectTimeoutError(<requests.packages.urllib3.connection.HTTPConnection object at 0x03D5DEF0>, 'Connection to qq.com timed out. (connect timeout=0.001)'))",),), {'timeout': 0.001})
```

------

>2016-4-26 21:53:18.Import Session for tPool. (in case of import requests.Session redundantly.)
```python
# from requests import Session
from torequests import Session
```

------

>2016-04-13 01:16:54. **Particular attention**, async function has been renamed as Async, it's not a class object but a function, though it indeed starts with an upper "A".This is to differ from the keyword **"async"** since python3.5+, for 'async' will be a keyword in python 3.7. So all the test code here is using **Async** instead of **async**, but async function is still be available(not suggested) for a while before python3.7 released.

```python
from torequests import Async
```
------

>2016-04-07 00:58:42. Add **get_by_time** function to play the **concurrent.futures.as_completed** role. This will return a generator which contains the results(i.x) one by one as completed order before time out, or timeout=None( default args ) for waiting till all finished. For example:

```python
from torequests import Async, get_by_time
import time

def func(n):
    time.sleep(n)
    return n
a_func = Async(func)
futures = [a_func(i) for i in [5,4,3,2,1]]
get_in_2s = get_by_time(futures, timeout=2)
for i in get_in_2s:
    print(i)
# Result:
# 0
# 1
# 2
# 2 (of 5) futures unfinished
```

------

# Quick Start & Introduction [ [中文简介](#cn) ]

```python
from torequests import tPool
import time

start_time = time.time()
trequests = tPool(30)  # you may use it without session either.
list1 = [trequests.get(url) for url in ['http://p.3.cn/prices/mgets?skuIds=J_1273600']*500]
# If failed, i.x may return False object by default, or you can reset the fail_return arg.
list2 = [i.x.status_code if i.x.status_code else 'fail' for i in list1]
end_time = time.time()
print(list2[:10], '\ntimeused:%s s' % (end_time-start_time))
```

>result:[200, 200, 200, 200, 200, 200, 200, 200, 200, 200] 
timeused:1.418180227279663 s

# Tutorial

> To get the **Returned Value**, use the **.x** property, but it will block the threads, so you can push the threads first but not use `.x` until really need the value.
> In other words, **`.x` should not be used until you need it**. The usage of **(async_func(i) for i in range(10)) or [async_func(i) for i in range(10)]** will cost time which equals to the most-cost-time function, because it's a sequence.

**first of all:**

>*pip install torequests -U*

## 1. tPool:
*make requests Async(and retry/log)*

### The args:

num means Pool size; session is requests.Session; retry is the times when exception raised; retrylog is one bool object and determined whether show the log when retry occured; logging args will show what you want see when finished successfully; delay will run after some seconds, so it only fit float or int.

-------
#####Usage:

```python
from torequests import tPool
import requests

s = requests.Session()
trequests = tPool(30, session=s)  # you may use it without session either.
list1 = [trequests.get(url, timeout=1, retry=1, retrylog=1, fail_return=False, logging='finished') for url in ['http://127.0.0.1:8080/']*5]
list2 = [i.x.text if i.x else 'fail' for i in list1]
print(list2)
```

-------

result:

```

http://127.0.0.1:8080/ finished
http://127.0.0.1:8080/ finished
http://127.0.0.1:8080/ finished
retry http://127.0.0.1:8080/ for the 1 time, for the Exception: HTTPConnectionPool(host='127.0.0.1', port=8080): Read timed out. (read timeout=1)
retry http://127.0.0.1:8080/ for the 1 time, for the Exception: HTTPConnectionPool(host='127.0.0.1', port=8080): Read timed out. (read timeout=1)
http://127.0.0.1:8080/ finished
retry http://127.0.0.1:8080/ for the 2 time, for the Exception: HTTPConnectionPool(host='127.0.0.1', port=8080): Read timed out. (read timeout=1)
[b'success', b'success', b'success', b'success', 'fail']

```

-------
>PS:
http://127.0.0.1:8080/ is one server that route a function like:

```python
@app.get('/')
def function():
    aa=random.randint(0,1)
    if aa:
        print(aa)
        return 'success'
    time.sleep(5)
    return 'fail'
```

-------


As it's async, you can use print function as logging. 

## 2. threads & Async:

>make functions asynchronous,very similar to original Tomorrow's threads.

#####Normal usage:

```python
# transform a function asynchronous

from torequests import Async
async_function = Async(old_function) # pool size is 30 for default, or set a new one async_function = Async(old_function,40)
# this step will not block.
func_list = [async_function(i) for i in argvs] # or func_list = map(async_function, argvs)
# this step will block.
results = [i.x for i in func_list]


# original usage
from torequests import threads
newfunc = threads(10)(rawfunc)
# or Decorator, but it influences source function.

@threads(10)
def rawfunc():
    pass
```
> when you need the real value returned from funtions, use `funtion().x`.

#### good & bad usage
```python
from torequests import Async

# GOOD. Only generate one thread-pool here.
async_pool = Async(lambda x: x())
funcs = [async_pool(function) for function in functions]
results = [i.x for i in funcs]

# BAD. This way may open too many thread-pool here.
funcs = [Async(function)() for function in functions]
results = [i.x for i in funcs]
```

##### demo:

```python
import time
import requests
import sys
from torequests import threads, Async
s = requests.Session()
counting = 0
# @threads(10) # the Decorator style
def download(url):
    global counting
    for _ in range(5):
        try:
            aa = s.get(url)
            counting += 1
            sys.stderr.write('%s  \r' % counting) 
            break
        except:
            pass
    return aa

urls = ['http://p.3.cn/prices/mgets?skuIds=J_1273600']*1000

if __name__ == "__main__":
    start = time.time()
    dd = Async(download)  # no difference to threads(30)(download)
    responses = [dd(url) for url in urls]
    html = [response.x.text for response in responses]
    end = time.time()
    print("Time: %f seconds" % (end - start))

# (time cost) Time: 2.321494 seconds

```

---------
<h2 id="cn">中文介绍</h2>
#### 类似于tomorrow的修饰器方式,使 [requests](https://github.com/kennethreitz/requests) 变得异步，并且加入更多功能（重试/默认错误返回值/log等）.

这个的唯一用处估计是比较无脑，并且还可以支持Windows吧。python2/3兼容。

>如果想返回 **真正的值（而不是NewFuture对象）**, 通过使用 **.x** 属性即可, 但要注意的一点是，虽然函数执行是异步的，但通过.x得到返回值却会block住整个进程。所以简单的办法就是，一上来把所有函数都异步出去，用到谁再**点**谁。（我比较习惯用列表解析把函数全放进去，然后要用到谁了取出来.x，其他的还在继续跑，不耽误）。注：切忌一次异步出去七八万项，这每个都是个独立的线程，会悲剧的，先分段然后再执行比较妥当。

# 快速开始

```python
from torequests import tPool
import time


start_time = time.time()
trequests = tPool(30)  # you may use it without session either.
list1 = [trequests.get(url) for url in ['http://p.3.cn/prices/mgets?skuIds=J_1273600']*500]
# 如果函数执行失败（或超过重试次数）, i.x 默认会返回False对象, 除非你自定义去修改 fail_return 参数.
list2 = [len(i.x.content) if i.x else 'fail' for i in list1]
end_time = time.time()
print(list2[:10], '\ntimeused:%s s' % (end_time-start_time))

```

>result:[51, 51, 51, 51, 51, 51, 51, 51, 51, 51] 
timeused:1.488060712814331 s

# 简单使用

当然是先 **pip** 安装:

>*pip install torequests -U*

## 1. tPool:
*让原生的 requests 变的异步，并且支持重试等功能。*

### The args:

num 参数是指线程池大小（同时执行多少任务）; session 其实就是 requests.Session(); retry 参数指的是出错重试的次数; retrylog 是指当重试发生时是否打印出来; logging 参数就是完成时候打印出来; delay 参数很少使用，一般就是给个随机时间让它等待然后再执行（貌似没什么卵用）.

-------
#####用法:

```python
from torequests import tPool
import requests

s = requests.Session()
trequests = tPool(30, session=s)  # Session的好处是性能比每次都重新发送请求快，并且能保留Cookies等（但我基本从来没用过这参数）.
list1 = [trequests.get(url, timeout=1, retry=1, retrylog=1, fail_return=False, logging='finished') for url in ['http://127.0.0.1:8080/']*5]
list2 = [i.x if i.x else 'fail' for i in list1]
print(list2)
```

-------

result:

```

http://127.0.0.1:8080/ finished
http://127.0.0.1:8080/ finished
http://127.0.0.1:8080/ finished
retry http://127.0.0.1:8080/ for the 1 time, for the Exception: HTTPConnectionPool(host='127.0.0.1', port=8080): Read timed out. (read timeout=1)
retry http://127.0.0.1:8080/ for the 1 time, for the Exception: HTTPConnectionPool(host='127.0.0.1', port=8080): Read timed out. (read timeout=1)
http://127.0.0.1:8080/ finished
retry http://127.0.0.1:8080/ for the 2 time, for the Exception: HTTPConnectionPool(host='127.0.0.1', port=8080): Read timed out. (read timeout=1)
[b'success', b'success', b'success', b'success', 'fail']

```

-------
>PS:
http://127.0.0.1:8080/ 是我架的一个临时服务器，用来随机性地出错来测试重试功能:

```python
@app.get('/')
def function():
    aa=random.randint(0,1)
    if aa:
        print(aa)
        return 'success'
    time.sleep(5)
    return 'fail'
```

-------

TIPS：由于这里的requests变成异步了，所以可以用 print 来查看进度。
注意：这里的异步只在请求的时候异步（因为这种I/O操作最费时），而取值的时候则不是，比如： 
```[trequests.get(url, timeout=1, retry=1, retrylog=1, fail_return=False, logging='finished').x for url in ['http://127.0.0.1:8080/']*5]```并不能异步来提高性能，换句话说，它变成串行执行了（所以平时我经常拿单个的代替原生的 requests …）。

## 2. threads & Async:
 
>这两个就是 Tomorrow 的神奇之处(把一个指定函数转成ThreadPoolExecutor)，用法和原生的没什么区别。简而言之就是把普通函数变成异步函数，你**把它撒出去它就是异步的非阻塞状态，直到你要取它的返回值 —— NewFuture.x**。

#####Normal usage:

```python
from torequests import Async
async_function = Async(old_function) # 线程池大小默认30，也可以设置个40 async_function = Async(old_function,40)
# 这一步非阻塞.
func_list = [async_function(i) for i in argvs] # or func_list = map(async_function, argvs)
# 下一步开始阻塞.
results = [i.x for i in func_list]


# 原生用法
from torequests import threads
# 这个用法用的是新函数，不会影响到原来的函数
newfunc = threads(10)(rawfunc)

# 也可以用修饰器，但是会对原函数造成影响。
@threads(10)
def rawfunc():
    pass
```
> 函数执行时候非阻塞，会直接跳过去，直到你调用它的返回值 —— `funtion().x`. 如下傻瓜式用法：

```python
from torequests import Async
import time


def function():
    time.sleep(3)
    return 'function 执行完毕'

a_func = Async(function)

result1 = a_func()
print('现在是异步的，所以它只是个 NewFuture 对象：', result1)
print('虽然a_func在执行，但我可以print出来，所以确实是异步了')
print('现在会阻塞住三秒，等待返回结果：')
[(time.sleep(1), print(3-i)) for i in range(3)]
print(result1.x)

print('然后实验一次错误用法，这里我将直接使用函数的值，比如print它：print(a_func().x)')
print(a_func().x)
print('所以被阻塞住了，等待了3秒。')
```
#### 注：每个函数异步后是一个线程池，如果需要同时得到多个函数的返回结果，则有两种好与坏的用法：
```python
from torequests import Async

# GOOD. Only generate one thread-pool here.
async_pool = Async(lambda x: x())
funcs = [async_pool(function) for function in functions]
results = [i.x for i in funcs]

# BAD. This way may open too many thread-pool here.
funcs = [Async(function)() for function in functions]
results = [i.x for i in funcs]
```

##### demo:

```python
import time
import requests
import sys
from torequests import threads, Async
s = requests.Session()
counting = 0
# @threads(10) # 这里可以用原生的修饰器方法
def download(url):
    global counting
    for _ in range(5):
        try:
            aa = s.get(url)
            counting += 1
            sys.stderr.write('%s  \r' % counting) 
            break
        except:
            pass
    return aa

urls = ['http://p.3.cn/prices/mgets?skuIds=J_1273600']*1000

if __name__ == "__main__":
    start = time.time()
    dd = Async(download)  # 其实就是 threads(30)(download)，但这样子更好看，虽然可能和python3.5关键词重名
    responses = [dd(url) for url in urls]
    html = [response.x.text for response in responses]
    end = time.time()
    print("Time: %f seconds" % (end - start))

# (耗费时间) Time: 2.321494 seconds

```


---------


# Some explanation for blocking while using Async.
# 有关 Async 函数阻塞的一些说明。
#####example.py
```python
from torequests import Async
import time


# Usage #0, limit the Pool size as 2.
print(
    '# Usage #0, limit the Pool size as 2, this will cost 2s * 2 instead of 2s, for it can only run 2 tasks per time.')


def function_pool_2(n):
    time.sleep(2)
    return n

start_time = time.time()
async_func = Async(function_pool_2, 2)
# This will cost 4s.
push_funcs = [async_func(i) for i in range(4)]
results = [i.x for i in push_funcs]
print(results, 'time passed :', time.time()-start_time)


# Usage #1, one function with many args for multi-missions
print(
    '# Usage #1, one function with many args for multi-missions, this will cost 2s.')


def function(n):
    time.sleep(2)
    return n

start_time = time.time()
async_func = Async(function)
# This will cost 2s.
push_funcs = [async_func(i) for i in range(10)]
results = [i.x for i in push_funcs]
print(results, 'time passed :', time.time()-start_time)


# Usage #2, multi-functions with different args
print('# Usage #2, multi-functions with different args, this will cost 2s.')


def func1(n):
    time.sleep(2)
    return n


def func2(n):
    time.sleep(2)
    return n


def func3(n):
    time.sleep(2)
    return n
start_time = time.time()
async_func = Async(lambda x, i: x(i))
push_funcs = [async_func(func, i)
              for func, i in zip([func1, func2, func3], range(3))]
results = [i.x for i in push_funcs]
print(results, 'time passed :', time.time()-start_time)


# Usage #3, usage of timeout_return
print(
    '# Usage #3, usage of timeout_return. This will block for 3s, but gotcha value in 1s.')


def function_timeout(n):
    time.sleep(n)
    print('This is still running even gotcha a TimeoutError!', n)
    return n

start_time = time.time()
async_func = Async(
    function_timeout, timeout=1, timeout_return='timeout lalala')
push_funcs = [async_func(i) for i in range(3)]
results = push_funcs[1].x
# Even though it will get value in 1 second, the functions will not be end
# until all the push_funcs finished.
print(results, 'time passed :', time.time()-start_time)
time.sleep(2)  # wait for the functions above...

# Usage #4, filter for TimeoutErrors
print('# Usage #4, filter for TimeoutErrors')


def function_timeout_filter(n):
    time.sleep(n)
    return n

start_time = time.time()
async_func = Async(function_timeout_filter, timeout=1, timeout_return='')
# This will cost 4 seconds obviously, because of function_timeout_filter(4).
push_funcs = [async_func(i) for i in range(5)]
results = [i.x for i in push_funcs]
filter_results = [i for i in results if i]
print(filter_results, 'time passed :', time.time()-start_time)


```
#####result:
```python
# Usage #0, limit the Pool size as 2, this will cost 2s * 2 instead of 2s, for it can only run 2 tasks per time.
[0, 1, 2, 3] time passed : 4.0016865730285645
# Usage #1, one function with many args for multi-missions, this will cost 2s.
[0, 1, 2, 3, 4, 5, 6, 7, 8, 9] time passed : 2.003317356109619
# Usage #2, multi-functions with different args, this will cost 2s.
[0, 1, 2] time passed : 2.0016467571258545
# Usage #3, usage of timeout_return. This will block for 3s, but gotcha value in 1s.
This is still running even gotcha a TimeoutError! 0
This is still running even gotcha a TimeoutError! 1
1 time passed : 1.002580165863037
This is still running even gotcha a TimeoutError! 2
# Usage #4, filter for TimeoutErrors
[1] time passed : 4.002671718597412
```

# 很明显的，因为是线程，所以没有什么好办法像os.kill让进程自杀一样来终止掉超时的线程，
# 所以，万万不要写那种不会自己停止的程序。
# 等有时间用 ProcessPoolExecutor 写一个进程版的，再让超时的进程强制结束吧。
