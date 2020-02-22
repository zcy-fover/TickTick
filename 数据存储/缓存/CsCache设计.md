## CsCache缓存实现

### 缓存架构介绍

> ![CsCache缓存分层图](https://note.youdao.com/yws/api/personal/file/WEB428ba4f09ab7b0b0e322cddf281d927f?method=download&shareKey=b5328a35916b4989be2b2393ca0d6ce8)
>
> - 客户端层：使用者直接通过该层与数据进行交互
> - 缓存提供层：对缓存管理层的生命周期进行维护，负责缓存管理层的创建、保存、获取和销毁
> - 缓存管理层：对缓存客户端的生命周期进行维护，负责客户端的创建、保存、获取以及销毁
> - 缓存存储层：负责以什么样的形式存储数据
>
> > - 基本存储层：以普通的ConcurrentHashMap为存储核心，不淘汰数据
> > - LRU存储层：以最近最少使用原则进行数据存储和缓存淘汰
> > - Weak存储层：以弱引用为原则的数据存储和缓存淘汰机制

<div style="text-align:center;margin-top:50px;margin-bottom:50px;">
    <img src="https://note.youdao.com/yws/api/personal/file/C2C6FCFDC10942B6A3532E6F0928E455?method=download&shareKey=c554dacfc5193c29d4b35682aa1226d9" />
</div>
