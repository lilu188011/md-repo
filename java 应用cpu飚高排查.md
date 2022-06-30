## java 应用cpu飚高排查

### 一、查询出cpu占有率高的java程序进程

top命令

![image-20220630100741543](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220630100741543.png)

### 二、  查询出该进程id中的线程信息

```shell
ps -mp 2308 -o THREAD,tid,time
```

![image-20220630100202919](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220630100202919.png)

### 三、 将线程id转换为16进制

```
printf "%x\n" 2320

910
```

### 四、 使用jstack排查进程中的线程

```shell
jstack 2308 | grep 910 -A100
```

![image-20220630100515296](https://raw.githubusercontent.com/lilu188011/img-repo/master/image-20220630100515296.png)