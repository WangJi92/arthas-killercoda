> 方法执行数据观测

让你能方便的观察到指定方法的调用情况。能观察到的范围为：`返回值`、`抛出异常`、`入参`，通过编写 OGNL 表达式进行对应变量的查看。

### 参数说明

watch 的参数比较多，主要是因为它能在 4 个不同的场景观察对象

|参数名称|参数说明|
|---:|:---|
|*class-pattern*|类名表达式匹配|
|*method-pattern*|方法名表达式匹配|
|*express*|观察表达式，默认值：`{params, target, returnObj}`|
|*condition-express*|条件表达式|
|[b]|在**方法调用之前**观察|
|[e]|在**方法异常之后**观察|
|[s]|在**方法返回之后**观察|
|[f]|在**方法结束之后**(正常返回和异常返回)观察|
|[E]|开启正则表达式匹配，默认为通配符匹配|
|[x:]|指定输出结果的属性遍历深度，默认为 1|

这里重点要说明的是观察表达式，观察表达式的构成主要由 ognl 表达式组成，所以你可以这样写`"{params,returnObj}"`，只要是一个合法的 ognl 表达式，都能被正常支持。

观察的维度也比较多，主要体现在参数 `advice` 的数据结构上。`Advice` 参数最主要是封装了通知节点的所有信息。请参考[表达式核心变量](advice-class.md)中关于该节点的描述。

* 特殊用法请参考：[https://github.com/alibaba/arthas/issues/71](https://github.com/alibaba/arthas/issues/71)
* OGNL表达式官网：[https://commons.apache.org/proper/commons-ognl/language-guide.html](https://commons.apache.org/proper/commons-ognl/language-guide.html)

**特别说明**：

* watch 命令定义了4个观察事件点，即 `-b` 方法调用前，`-e` 方法异常后，`-s` 方法返回后，`-f` 方法结束后
* 4个观察事件点 `-b`、`-e`、`-s` 默认关闭，`-f` 默认打开，当指定观察点被打开后，在相应事件点会对观察表达式进行求值并输出
* 这里要注意`方法入参`和`方法出参`的区别，有可能在中间被修改导致前后不一致，除了 `-b` 事件点 `params` 代表方法入参外，其余事件都代表方法出参
* 当使用 `-b` 时，由于观察事件点是在方法调用前，此时返回值或异常均不存在

### 使用参考

#### 观察方法出参、this对象和返回值

> 观察表达式，默认值是`{params, target, returnObj}`。

`watch demo.MathGame primeFactors -x 2`{{execute T2}}

按 `Q`{{exec interrupt}} 或者 `Ctrl+c`{{exec interrupt}} 退出

```bash
$ watch demo.MathGame primeFactors -x 2
Press Q or Ctrl+C to abort.
Affect(class count: 1 , method count: 1) cost in 32 ms, listenerId: 5
method=demo.MathGame.primeFactors location=AtExceptionExit
ts=2021-08-31 15:22:57; [cost=0.220625ms] result=@ArrayList[
    @Object[][
        @Integer[-179173],
    ],
    @MathGame[
        random=@Random[java.util.Random@31cefde0],
        illegalArgumentCount=@Integer[44],
    ],
    null,
]
method=demo.MathGame.primeFactors location=AtExit
ts=2021-08-31 15:22:58; [cost=1.020982ms] result=@ArrayList[
    @Object[][
        @Integer[1],
    ],
    @MathGame[
        random=@Random[java.util.Random@31cefde0],
        illegalArgumentCount=@Integer[44],
    ],
    @ArrayList[
        @Integer[2],
        @Integer[2],
        @Integer[26947],
    ],
]
```

* 上面的结果里，说明函数被执行了两次，第一次结果是`location=AtExceptionExit`，说明函数抛出异常了，因此`returnObj`是null
* 在第二次结果里是`location=AtExit`，说明函数正常返回，因此可以看到`returnObj`结果是一个ArrayList

#### 观察方法入参

`watch demo.MathGame primeFactors "{params,returnObj}" -x 2 -b`{{execute T2}}

按`Q`{{execute T2}}或者`Ctrl+c`退出

```bash
$ watch demo.MathGame primeFactors "{params,returnObj}" -x 2 -b
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 50 ms.
ts=2018-12-03 19:23:23; [cost=0.0353ms] result=@ArrayList[
    @Object[][
        @Integer[-1077465243],
    ],
    null,
]
```

* 对比前一个例子，返回值为空（事件点为方法执行前，因此获取不到返回值）


#### 同时观察方法调用前和方法返回后

`watch demo.MathGame primeFactors "{params,target,returnObj}" -x 2 -b -s -n 2`{{execute T2}}

按`Q`{{execute T2}}或者`Ctrl+c`退出

```bash
$ watch demo.MathGame primeFactors "{params,target,returnObj}" -x 2 -b -s -n 2
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 46 ms.
ts=2018-12-03 19:29:54; [cost=0.01696ms] result=@ArrayList[
    @Object[][
        @Integer[1],
    ],
    @MathGame[
        random=@Random[java.util.Random@522b408a],
        illegalArgumentCount=@Integer[13038],
    ],
    null,
]
ts=2018-12-03 19:29:54; [cost=4.277392ms] result=@ArrayList[
    @Object[][
        @Integer[1],
    ],
    @MathGame[
        random=@Random[java.util.Random@522b408a],
        illegalArgumentCount=@Integer[13038],
    ],
    @ArrayList[
        @Integer[2],
        @Integer[2],
        @Integer[2],
        @Integer[5],
        @Integer[5],
        @Integer[73],
        @Integer[241],
        @Integer[439],
    ],
]
```

* 参数里`-n 2`，表示只执行两次

* 这里输出结果中，第一次输出的是方法调用前的观察表达式的结果，第二次输出的是方法返回后的表达式的结果

* 结果的输出顺序和事件发生的先后顺序一致，和命令中 `-s -b` 的顺序无关

#### 调整`-x`的值，观察具体的方法参数值

`watch demo.MathGame primeFactors "{params,target}" -x 3`{{execute T2}}

按`Q`{{execute T2}}或者`Ctrl+c`退出

```bash
$ watch demo.MathGame primeFactors "{params,target}" -x 3
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 58 ms.
ts=2018-12-03 19:34:19; [cost=0.587833ms] result=@ArrayList[
    @Object[][
        @Integer[1],
    ],
    @MathGame[
        random=@Random[
            serialVersionUID=@Long[3905348978240129619],
            seed=@AtomicLong[3133719055989],
            multiplier=@Long[25214903917],
            addend=@Long[11],
            mask=@Long[281474976710655],
            DOUBLE_UNIT=@Double[1.1102230246251565E-16],
            BadBound=@String[bound must be positive],
            BadRange=@String[bound must be greater than origin],
            BadSize=@String[size must be non-negative],
            seedUniquifier=@AtomicLong[-3282039941672302964],
            nextNextGaussian=@Double[0.0],
            haveNextNextGaussian=@Boolean[false],
            serialPersistentFields=@ObjectStreamField[][isEmpty=false;size=3],
            unsafe=@Unsafe[sun.misc.Unsafe@2eaa1027],
            seedOffset=@Long[24],
        ],
        illegalArgumentCount=@Integer[13159],
    ],
]
```

* `-x`表示遍历深度，可以调整来打印具体的参数和结果内容，默认值是1。


#### 条件表达式的例子

`watch demo.MathGame primeFactors "{params[0],target}" "params[0]<0"`{{execute T2}}

按`Q`{{execute T2}}或者`Ctrl+c`退出

```bash
$ watch demo.MathGame primeFactors "{params[0],target}" "params[0]<0"
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 68 ms.
ts=2018-12-03 19:36:04; [cost=0.530255ms] result=@ArrayList[
    @Integer[-18178089],
    @MathGame[demo.MathGame@41cf53f9],
]
```

* 只有满足条件的调用，才会有响应。

* `watch-express` 单个值可以不加'{}'，多个值需要加'{a,b,c}'。

* `condition-express` 不能加'{}'，可以使用逗号分隔子表达式，取表达式最后一个值来判断。

如果watch的方法存在同名的其它重载方法，可以通过下面的办法进行过滤：

  * 根据参数类型进行过滤

   `watch demo.MathGame primeFactors '{params, params[0].class.name}' 'params[0].class.name == "java.lang.Integer"'`{{execute T2}}

   按`Q`{{execute T2}}或者`Ctrl+c`退出
   
 * 根据参数个数进行过滤
   
  `watch demo.MathGame primeFactors '{params, params.length}' 'params.length==1'`{{execute T2}}

  按`Q`{{execute T2}}或者`Ctrl+c`退出

#### 观察异常信息的例子

`watch demo.MathGame primeFactors "{params[0],throwExp}" -e -x 2`{{execute T2}}

按`Q`{{execute T2}}或者`Ctrl+c`退出

```bash
$ watch demo.MathGame primeFactors "{params[0],throwExp}" -e -x 2
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 62 ms.
ts=2018-12-03 19:38:00; [cost=1.414993ms] result=@ArrayList[
    @Integer[-1120397038],
    java.lang.IllegalArgumentException: number is: -1120397038, need >= 2
	at demo.MathGame.primeFactors(MathGame.java:46)
	at demo.MathGame.run(MathGame.java:24)
	at demo.MathGame.main(MathGame.java:16)
,
]
```

* `-e`表示抛出异常时才触发
* express中，表示异常信息的变量是`throwExp`

根据异常类型或者message进行过滤：

`watch demo.MathGame primeFactors '{params, throwExp}' '#msg=throwExp.toString(), #msg.contains("IllegalArgumentException")' -e -x 2`{{execute T2}}

按`Q`{{execute T2}}或者`Ctrl+c`退出

#### 按照耗时进行过滤

`watch demo.MathGame primeFactors '{params, returnObj}' '#cost>200' -x 2`{{execute T2}}

按`Q`{{execute T2}}或者`Ctrl+c`退出

```bash
$ watch demo.MathGame primeFactors '{params, returnObj}' '#cost>200' -x 2
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 66 ms.
ts=2018-12-03 19:40:28; [cost=2112.168897ms] result=@ArrayList[
    @Object[][
        @Integer[1],
    ],
    @ArrayList[
        @Integer[5],
        @Integer[428379493],
    ],
]
```

* `#cost>200`(单位是`ms`)表示只有当耗时大于200ms时才会输出，过滤掉执行时间小于200ms的调用


#### 观察当前对象中的属性

`watch demo.MathGame primeFactors 'target'`{{execute T2}}

按`Q`{{execute T2}}或者`Ctrl+c`退出

如果想查看方法运行前后，当前对象中的属性，可以使用`target`关键字，代表当前对象

```bash
$ watch demo.MathGame primeFactors 'target'
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 52 ms.
ts=2018-12-03 19:41:52; [cost=0.477882ms] result=@MathGame[
    random=@Random[java.util.Random@522b408a],
    illegalArgumentCount=@Integer[13355],
]
```

然后使用`target.field_name`访问当前对象的某个属性

`watch demo.MathGame primeFactors 'target.illegalArgumentCount'`{{execute T2}}

按`Q`{{execute T2}}或者`Ctrl+c`退出

```bash
$ watch demo.MathGame primeFactors 'target.illegalArgumentCount'
Press Ctrl+C to abort.
Affect(class-cnt:1 , method-cnt:1) cost in 67 ms.
ts=2018-12-03 20:04:34; [cost=131.303498ms] result=@Integer[8]
ts=2018-12-03 20:04:35; [cost=0.961441ms] result=@Integer[8]
``` 
