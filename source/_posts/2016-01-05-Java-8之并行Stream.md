---
title: Java 8之并行Stream
date: 2016-01-05 23:09:53
tags: [Java8,Java]
categories: Java
description: Java 8之并行Stream
---

Java 8中的Stream的操作可以通过并行的操作执行，通过`Stream.parallel`或者`Collection.parallelStream`方法可以获取并行操作的流。

并行流需要满足以下2个要求：

  * 恒等性：初始值必须为组合函数的恒等值，拿恒等值和其他值做reduce操作时，其他值保持不变。

  * 组合律：改变组合操作的顺序不会影响最终的结果。

<!-- more -->

在书中有个蒙特卡洛模拟法的例子，为了学习，参考书中，写个一样的例子如下

Java 8之前的版本

```java
public class MonteCarloMethod {

    private ConcurrentMap<Integer, Double> map = new ConcurrentHashMap<>();

    private int threadCount = Runtime.getRuntime().availableProcessors();

    private ExecutorService pool = Executors.newFixedThreadPool(threadCount);

    public static void main(String[] args) {
        // 执行次数
        long count = 100000000L;

        MonteCarloMethod bean = new MonteCarloMethod();
        bean.loop(count);
        bean.print();
    }

    private void loop(long count) {
        // 每个线程执行的个数
        long taskPerThread = count / threadCount;

        List<Future> futures = new ArrayList<>(threadCount);
        for (int i = 0; i < threadCount; i++) {
            futures.add(pool.submit(getTask(count, taskPerThread)));
        }

        // 等待执行完成
        futures.forEach(future -> {
            try {
                future.get();
            } catch (ExecutionException | InterruptedException e) {
                e.printStackTrace();
            }
        });

        pool.shutdown();
    }

    /**
     * 获取每个线程的执行任务
     *
     * @param count
     *         总的任务数
     * @param taskPerThread
     *         每个线程的任务数
     * @return 执行任务
     */
    private Runnable getTask(long count, long taskPerThread) {
        return () -> {
            double data = 1.0 / count;
            for (int i = 0; i < taskPerThread; i++) {
                map.compute(doTask(), (key, value) -> {
                    if (value == null)
                        return data;
                    else
                        return value + data;
                });
            }
        };
    }

    /**
     * 获取2个随机1到6的数字之和
     *
     * @return 随机数之和
     */
    private int doTask() {
        ThreadLocalRandom random = ThreadLocalRandom.current();
        int first = random.nextInt(1, 7);
        int second = random.nextInt(1, 7);
        return first + second;
    }

    /**
     * 打印map内容
     */
    private void print() {
        map.entrySet().forEach(System.out::println);
    }
}
```

Java 8版本如下：

```java
public class MonteCarloMethodStream {

    public static void main(String[] args) {
       new MonteCarloMethodStream().loop(100000000L);
    }

    public void loop(long count) {
        double data = 1.0 / count;
        LongStream.range(0, count).parallel()
                .mapToObj(origin -> random())
                .collect(Collectors.groupingBy(origin -> origin, Collectors.summingDouble(origin -> data)))
                .entrySet().forEach(System.out::println);
    }

    /**
     * 获取2个随机1到6的数字之和
     *
     * @return 随机数之和
     */
    private int random() {
        ThreadLocalRandom random = ThreadLocalRandom.current();
        int first = random.nextInt(1, 7);
        int second = random.nextInt(1, 7);
        return first + second;
    }
}
```

在Java 8中，并行流替我们做了一些并发处理的逻辑，代码显得简洁很多。
