#### 环境：

- Windows 7 x64
- IDEA 2017.01
- JDK 1.8 x64
- Maven 3.5.0

#### 源码获取：

1. ###### [Skywalking Git](https://github.com/apache/incubator-skywalking)可以利用 Git Clone 或者 ZIP 下载到本地；
2. ###### 利用 IDEA 选择 Maven 导入项目
> ![项目结构](https://note.youdao.com/yws/api/personal/file/84696A635FE34A2AB96EA860C836EB4B?method=download&shareKey=38a78b93e713a290424c3aecb3dd1445)

#### 启动 Skywalking-collecor

1. ###### 在项目根目录下或者 IntelliJ IDEA Terminal 运行`mvn clean compile install -Dmaven.test.skip=true`；
2. ###### 编译完后设置 gRPC 自动生成代码的目录：
- apm-network/target/generated-sources/protobuf 下的`grpc-java`和`java`;
> ![GRPC设置](https://note.youdao.com/yws/api/personal/file/69097195DFDA4470BED95B98B7577091?method=download&shareKey=04343bdaab809a02dd32244f5f64c109)
> - apm-collector/apm-collector-remote/collector-remote-grpc-provider/target/generated-sources/protobuf 下的`grpc-java`和`java`;
> ![gRPC设置](https://note.youdao.com/yws/api/personal/file/0B361E94E06447ABBBDF3555DE653F91?method=download&shareKey=326d795355d6d6b49240a3a980783749)
> 设置方法：在 grpc-java 上右键-> Mark Directory as -> Generated Souces Root
>
> ![代码生成](https://note.youdao.com/yws/api/personal/file/30E1435B92304D458B3ABB339CD7A802?method=download&shareKey=97fa68bf2d2359a8b692b60f6ea5a148)
3. ###### 运行`org.skywalking.apm.collector.boot.CollectorBootStartUp`的`main(args)`方法，启动 Collector；
4. ###### 在浏览器中输入[http://127.0.0.1:10800/agent/jetty](http://127.0.0.1:10800/agent/jetty)地址，返回 `["localhost:12800/"]` ，说明启动**成功**。

#### 启动 Skywalking-Agent
1. ###### 在 Skywalking 项目的同级新建一个 Web 项目
> ![](https://note.youdao.com/yws/api/personal/file/51229E52D67F4992AEE5B630235EBD1E?method=download&shareKey=aa3c4f8faf2a951211a2ec7c07d5eb2b)
2. ###### 我这里新建了一个 SpringBoot 的项目（注意 SpringBootDemo 必须和 skywalking 项目平级，这样才可以调试 Agent）
> ![项目结构](https://note.youdao.com/yws/api/personal/file/43ACCBD85E4E4CC68A54D62AD7A43A9C?method=download&shareKey=b9b05f406b4823bfa2223127902db8cd)
3. ###### 在`org.skywalking.apm.agent.SkyWalkingAgent`的`premain()`方法里打上断点；
4. ###### 在自己新建的Web项目，配置启动参数`-javaagent:skywalking-agent.jar的路径`（这里的路径可以绝对路径也可以是相对路径）
> ![agentJarPath](https://note.youdao.com/yws/api/personal/file/273872A5D9594E75A9B52AFCCBBABB0E?method=download&shareKey=29fb5f5e1caed391b03654af671387d2)
> ![web项目启动配置](https://note.youdao.com/yws/api/personal/file/14334ADFF8CC48E7BE6C1770443974BA?method=download&shareKey=37b0f630ae742edee01460583f44d30b)
5. 启动Web项目，这里可能会出现`agent.application_code is missing`或者`collector.servers is missing.`的错误，我改了`org.skywalking.apm.agent.core.conf.Config`中的：
> ```java
> public static String APPLICATION_CODE = "SpringBootDemo";
> public static String SERVERS = "127.0.0.1:8080";
> ```
6. 此时停掉之前启动的 skywalking-collector，重新运行`mvn clean compile install -Dmaven.test.skip=true`进行编译；
7. 编译完成后再启动 skywalking-collector，然后启动自己的Web项目，如果程序进入了之前的`org.skywalking.apm.agent.SkyWalkingAgent`的`premain()`方法中的断点，并且 Web 项目启动成功则说明 Agent 模块启动成功。

<div style="text-align:center;margin-top:50px;margin-bottom:50px;">
    <img src="https://note.youdao.com/yws/api/personal/file/C2C6FCFDC10942B6A3532E6F0928E455?method=download&shareKey=c554dacfc5193c29d4b35682aa1226d9" />
</div>
