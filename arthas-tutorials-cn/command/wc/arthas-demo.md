下载 `math-game.jar` - [[源码]](https://github.com/alibaba/arthas/blob/af70d95383ddc3692dbcd0e9c1cbfc201ae43a98/math-game/src/main/java/demo/MathGame.java)，再用 `java -jar` 命令启动：

```
wget https://arthas.aliyun.com/math-game.jar
java -jar math-game.jar
```{{execute T1}}

`math-game` 是一个很简单的程序，它随机生成整数，再执行因式分解，把结果打印出来。如果生成的随机数是负数，则会打印提示信息。

