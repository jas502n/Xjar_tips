# Xjar_tips

#### 简介：

```
Spring Boot JAR 安全加密运行工具, 同时支持的原生JAR.
基于对JAR包内资源的加密以及拓展ClassLoader来构建的一套程序加密启动, 动态解密运行的方案, 避免源码泄露以及反编译.

功能特性
  无代码侵入, 只需要把编译好的JAR包通过工具加密即可.
  完全内存解密, 降低源码以及字节码泄露或反编译的风险.
  支持所有JDK内置加解密算法.
  可选择需要加解密的字节码或其他资源文件.
  支持Maven插件, 加密更加便捷.
  动态生成Go启动器, 保护密码不泄露.

其中xjar 二进制是go编写的，里面存储了秘钥等信息，xxx-1.0-SNAPSHOT.xjar 是AES 加密后的文件。

目的：为了获取xjar启动器 传给 jvm的参数，可以使用标准输出流打印出来。
```





正常xjar 运行加密的jar的命令：

```bash
$ ps -ef |grep java |grep xjar

$ file xjar
xjar: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically linked, Go BuildID=xx-C, not stripped

$ ./xjar java -jar xxx-1.0-SNAPSHOT.xjar
```





#### java版：

```java
public class f {

    static {
        try {
            java.io.BufferedReader br = new java.io.BufferedReader(new java.io.InputStreamReader(System.in));

            System.out.println("[+] Get Jvm Stdin Input: ");
            String input = br.readLine();
            while (input != null) {
                System.out.println("args = " + input);
                input = br.readLine();
            }


        } catch (Exception e) {
            e.printStackTrace();
        }

    }

    public static void main(String[] args) {

    }

}
```

编译：

```
javac f
```

##### 获取jvm参数

```
./xjar java f java -jar ./xxx-1.0-SNAPSHOT.xjar
```

##### 输出信息:

```
[+] Get Jvm Stdin Input:
args = AES/CBC/PKCS5Padding
args = 128
args = 128
args = Passw0rd
```

对应xjar go代码：

https://github.com/core-lib/xjar/blob/master/src/main/resources/xjar/xjar.go

```
var xKey = XKey{
	algorithm: []byte{#{xKey.algorithm}}, =>> AES/CBC/PKCS5Padding
	keysize:   []byte{#{xKey.keysize}},   =>> 128
	ivsize:    []byte{#{xKey.ivsize}},    =>> 128
	password:  []byte{#{xKey.password}},  =>> Passw0rd
}
```



#### 题外话：

这个密码可以通过strings xjar查找关键词找出来

```
$ strings xjar |grep -B 10 "AES/CBC/PKCS5Padding"
```

ida分析：

```
.text:00000000004AB7BB	main.main	                mov     rbx, cs:off_573F48 ; "Passw0rd\t"
.noptrdata:0000000000560120	0000000A	C	Passw0rd\t
```

在main.main中

```
lea     rbx, [rsp+170h+var_126]
mov     [rsp+170h+var_50], rbx
mov     rbx, cs:off_573F48 ; "Passw0rd\t"
```



### 利用密码进行解密：

主要分 springboot 版jar 和 单个jar



springboot版解密：

```java
String password = "Passw0rd"; //设置密码
XKey xKey = XKit.key(password);
XBoot.decrypt("/path/to/read/encrypted.jar", "/path/to/save/decrypted.jar", xKey);
```



单个jar包解密：

```java
String password = "Passw0rd";
XKey xKey = XKit.key(password);
XJar.decrypt("/path/encrypted.jar", "/path/decrypted.jar", xKey);
```





参考链接：代码审计 - xjar加密破解

https://mp.weixin.qq.com/s/donLtYHXz5BWsF5AnOOmYA

#### go 版本：

```go
package main

import (
	"bufio"
	"fmt"
	"os"
)

func main() {
	input := bufio.NewScanner(os.Stdin) //初始化一个扫表对象
	for input.Scan() {                  //扫描输入内容
		line := input.Text() //把输入内容转换为字符串
		fmt.Println(line)    //输出到标准输出
	}
}
```

##### 对应的命令：

```
./xjar ./xjarCrack java -jar xxx-1.0-SNAPSHOT.xjar
```



#### Php 版本：

```
<?php var_dump(file_get_contents('php://stdin'));
```

##### 对应的命令：

```
./xjar php test.php java -jar xxx-1.0-SNAPSHOT.xjar
```


