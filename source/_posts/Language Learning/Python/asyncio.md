

# Python Asynchronous I/O - Asyncio

## intro：

> asyncio is a library to write **concurrent** code using the **async/await** syntax.

## Detail：

example01:

```python
import asyncio
import random

async def myCoroutine(id):
    process_time = random.randint(1,5)
    await asyncio.sleep(process_time)
    print("Coroutine: {}, has successfully completed after {} seconds".format(id, process_time))

async def main():
    tasks = []
    for i in range(10):
        tasks.append(asyncio.ensure_future(myCoroutine(i)))

    await asyncio.gather(*tasks)


loop = asyncio.get_event_loop()
try:
    loop.run_until_complete(main())
finally:
    loop.close()
```

explanation:





example02:

```python
import asyncio
import time

async def say_after(delay, what):
    await asyncio.sleep(delay)
    print(what)
    print(f"end: {time.strftime('%X')}")

async def main():
    print(f"started at {time.strftime('%X')}")
    task1 = asyncio.create_task(say_after(1, 'hello'))
    task2 = asyncio.create_task(say_after(2, 'world'))
    # Wait until both tasks are completed (should take around 2 seconds.)

    await task1
    # time.sleep(1)
    # await say_after(2, f"{time.strftime('%X')}") # 延迟一秒运行，与上方task1耗时一直
    # await say_after(2, f"{time.strftime('%X')}")
    await task2
    await say_after(2, f"{time.strftime('%X')}") # 延迟两秒运行，task1与2并行执行，总用时2s，故有2s
    await say_after(2, f"{time.strftime('%X')}")
	# “coroutines”互相被等待，同时其并不会加入并行的对列，会等到其上方所有任务完成再开始执行
    
    print(f"finished at {time.strftime('%X')}")

asyncio.run(main())
```

























## Link：

1. [asyncio — Asynchronous I/O — Python 3.10.7 documentation](https://docs.python.org/3/library/asyncio.html)