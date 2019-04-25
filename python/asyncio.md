
# Ref
* [Python黑魔法 --- asyncIO](https://www.jianshu.com/p/b5e347b3a17c)  
* [Async/await Tutorial](https://stackabuse.com/python-async-await-tutorial)  
* [Develop with asyncio](https://docs.python.org/3/library/asyncio-dev.html#asyncio-dev)  


# FAQ
* [asyncio.ensure_future vs. BaseEventLoop.create_task vs. simple coroutine?](https://stackoverflow.com/questions/36342899/asyncio-ensure-future-vs-baseeventloop-create-task-vs-simple-coroutine)
* [Asyncio.gather vs asyncio.wait](https://stackoverflow.com/questions/42231161/asyncio-gather-vs-asyncio-wait)  
  * [asyncio.gather()](https://docs.python.org/3/library/asyncio-task.html#asyncio.gather)
    * Mainly focuses on gathering the results. it waits on a bunch of futures and return their results in a given order.
    * The type of return value type is result.
  * [asyncio.wait()](https://docs.python.org/3/library/asyncio-task.html#asyncio.wait)  
    * Asyncio.wait is more low level than asyncio.gather.
    * Asyncio.wait just waits on the futures. and instead of giving you the results directly, it gives done and pending tasks. you have to mannually collect the values.
    * Moreover, you could specify to wait for all futures to finish or the just the first one with wait.
    * The type of return value is future.


# Example
* Get result From the task (python3.6)
  ```python
  import asyncio

  async def do_some_work(x):
      print('Waiting: ', x)
      await asyncio.sleep(x)
      return 'Done after {}s'.format(x)

  loop = asyncio.get_event_loop()

  coroutine = do_some_work(2)
  task = asyncio.ensure_future(coroutine)

  loop.run_until_complete(task)

  print('Task result: ', task.result())

  loop.close()
  ```
* Get result From the task (python3.7)
  ```python
  import asyncio

  async def do_some_work(x):
    print('Waiting: ', x)
      await asyncio.sleep(x)
      return 'Done after {}s'.format(x)

  coroutine = do_some_work(2)
  task = asyncio.ensure_future(coroutine)

  asyncio.run(task)

  print('Task result: ', task.result())
  ```

* Add callback function
  ``` python
  import time
  import asyncio

  now = lambda : time.time()
  start = now()

  async def do_some_work(x):
      print('Waiting: ', x)
      return 'Done after {}s'.format(x)

  def callback(future):
      print('Callback: ', future.result())

  loop = asyncio.get_event_loop()

  coroutine = do_some_work(2)
  task = asyncio.ensure_future(coroutine)
  task.add_done_callback(callback)

  loop.run_until_complete(task)

  print('Task result: ', task.result())
  ```  
* Using asyncio.gather for concurrent asyncio
  ``` python
  import asyncio
  import time

  now = lambda: time.time()

  async def do_some_work(x):
      print('Waiting: ', x)

      await asyncio.sleep(x)
      return 'Done after {}s'.format(x)

  start = now()

  coroutine1 = do_some_work(1)
  coroutine2 = do_some_work(1)
  coroutine3 = do_some_work(2)

  tasks = [
      asyncio.ensure_future(coroutine1),
      asyncio.ensure_future(coroutine2),
      asyncio.ensure_future(coroutine3)
  ]

  loop = asyncio.get_event_loop()
  loop.run_until_complete(asyncio.gather(*tasks))

  for task in tasks:
      print('Task ret: ', task)

  print('TIME: ', now() - start)
  ```

* Using asyncio.wait for concurrent asyncio
  ```python
  import asyncio
  import random


  async def coro(tag):
      print(">", tag)
      await asyncio.sleep(random.uniform(0.5, 5))
      print("<", tag)
      return tag


  loop = asyncio.get_event_loop()

  tasks = [coro(i) for i in range(1, 11)]

  print("Get first result:")
  finished, unfinished = loop.run_until_complete(
      asyncio.wait(tasks, return_when=asyncio.FIRST_COMPLETED))

  for task in finished:
      print(task.result())
  print("unfinished:", len(unfinished))

  print("Get more results in 2 seconds:")
  finished2, unfinished2 = loop.run_until_complete(
      asyncio.wait(unfinished, timeout=2))

  for task in finished2:
      print(task.result())
  print("unfinished2:", len(unfinished2))

  print("Get all other results:")
  finished3, unfinished3 = loop.run_until_complete(asyncio.wait(unfinished2))

  for task in finished3:
      print(task.result())

  loop.close()
  ```
* Run as completed
  ``` python
  import asyncio
  import time

  now = lambda: time.time()

  async def do_some_work(x):
      print('Waiting: ', x)

      await asyncio.sleep(x)
      return 'Done after {}s'.format(x)

  async def main():
      coroutine1 = do_some_work(1)
      coroutine2 = do_some_work(2)
      coroutine3 = do_some_work(4)

      tasks = [
          asyncio.ensure_future(coroutine1),
          asyncio.ensure_future(coroutine2),
          asyncio.ensure_future(coroutine3)
      ]
      for task in asyncio.as_completed(tasks):
          result = await task
          print('Task ret: {}'.format(result))

  start = now()

  loop = asyncio.get_event_loop()
  done = loop.run_until_complete(main())
  print('TIME: ', now() - start)
  ```



