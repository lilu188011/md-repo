## CompletableFuture 线程异步编排

### 一、创建方式

```java
  1. 指定线程池 异步执行 无返回值
 CompletableFuture<Void> completableFuture = CompletableFuture
                .runAsync(() -> System.out.println(""), executors);
  2. 指定线程池 异步执行 含返回值    
    CompletableFuture<String> completableFuture = CompletableFuture.supplyAsync(() -> {
                System.out.println("--------");
                return "ok";
            }, executors);
```

### 二、线程串行化方法

![image-20220608102703132](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220608102703132.png)

```
thenRun方法： 只要任务执行完成，就开始执行thenRun
thenAccept方法： 消费处理结果
thenApply 方法： 获取上一个任务返回值 并且返回当前任务返回值
带有Asyn表示异步执行
```

### 三、两任务组合 - 都要完成

![image-20220608103134129](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220608103134129.png)

```
thenCombine: 組合两个future，获取两个future的返回结果，并返回当前任务的返回值
thenAcceptBoth: 组合两个future，获取两个future任务的返回结果，然后处理当前任务，无返回值
runAfterBoth: 组合两个future，不需要获取future的结果，只需两个future处理完任务，处理该任务
```

### 四、两任务组合 - 一个完成

![image-20220608104007421](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220608104007421.png)

```
applyToEither: 組合两个future，获取两个future的返回结果，并返回当前任务的返回值
acceptEither: 组合两个future，获取两个future任务的返回结果，然后处理当前任务，无返回值
runAfterEither: 组合两个future，不需要获取future的结果，只需两个future处理完任务，处理该任务
```

### 五、多任务组合

![image-20220608104438707](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220608104438707.png)

```
allof： 所有任务都完成
anyof:  只要一个任务完成
```





```
 public static void test() throws Exception {
        ExecutorService executors = Executors.newFixedThreadPool(10);
        // task 3 4 5 依赖task1  task2 独立
        CompletableFuture<String> task1Future = CompletableFuture.supplyAsync(() -> {
            System.out.println("task1");
            return "ok";
        }, executors);

        CompletableFuture<Void> task2Future = CompletableFuture.runAsync(() -> {
            System.out.println("task2");
        }, executors);

        CompletableFuture<Void> task3Future = task1Future.thenAcceptAsync((result) -> {
            System.out.println(result + "task3");
        });

        CompletableFuture<Void> task4Future = task1Future.thenAcceptAsync((result) -> {
            System.out.println(result + "task4");
        });

        CompletableFuture<Void> task5Future = task1Future.thenAcceptAsync((result) -> {
            System.out.println(result + "task5");
        });
        CompletableFuture.allOf(task2Future,task3Future,task4Future,task5Future).get();

    }
```

