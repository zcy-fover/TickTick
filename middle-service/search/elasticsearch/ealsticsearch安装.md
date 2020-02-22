## 环境

- 系统：MACOS
- Java 1.8+

## 版本

---

- 版本历史：1.x -> 2.x -> 5.x -> 6.x
- 下载的是 6.5.4

## 安装

---

​	 首先在官网上下载压缩包，下载地址[官网](https://www.elastic.co/downloads/elasticsearch#ga-release)，或者在命令行使用`wget`命令获取，下载后解压即可。

### 目录介绍

​	`bin`存放相关脚本命令

​	`config`启动相关配置文件

​	`lib`依赖的第三方库

​	`modules`模块目录

​	`plugins`第三方插件

​	`logs`运行后会默认产生的日志文件夹

​	`data`运行后产生的数据文件夹

### 单实例安装

​        打开`terminal`，目录切换至解压后的 elasticsearch 文件夹的 bin 目录下，输入`java -version`检查 java 版本是否符合，要求在1.8以上。

​	  键入`sh elasticsearch`后回车，启动服务，默认端口 9200，在浏览器输入[localhost:9200](localhost:9200)出现以下信息，启动成功。

```json
{
  "name" : "Bc8O_aI",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "xYAoAJt1QFihcu9U-TRQXg",
  "version" : {
    "number" : "6.5.4",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "d2ef93d",
    "build_date" : "2018-12-17T21:17:40.758843Z",
    "build_snapshot" : false,
    "lucene_version" : "7.5.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

### 插件安装

- #### Head插件

  