## EOS 源码学习之项目结构分析(plugins)

整体的代码位置主要位于plugins、libraries。

**eos项目采用了插件管理的结构**。



####  入口

programs的文件夹结构如下，在[eos官方wiki](https://github.com/EOSIO/eos/wiki/Programs-&-Tools)中可以查看具体每个目录的含义及作用。这里仅备注几个示例。

```
.. eos/programs:

├── CMakeLists.txt
├── cleos //命令行工具，如果把eos比作操作系统的话，cleos就类似终端作用  
├── debug_node 
├── eosio-abigen
├── eosio-applesedemo
├── eosio-launcher
├── genesis
├── keosd  //钱包
├── nodeos  //节点核心进程
└── snapshot
```

要启动一个节点，需要用到的便是nodeos，cleos中可使用的REST API便是nodeos中暴露出来的。那么，我们看看nodeos。

源代码main.cpp中，代码仅有105行。主要看看main方法吧（不想看代码可以略过代码部分 = =），通过代码可以看到，main方法作用包括设置版本、工作路径等基本信息，以及log的初始化，另外就是startup方法看起来就是关键！

```
int main(int argc, char** argv)
{
   try {
      app().set_version(eosio::nodeos::config::version);
      auto root = fc::app_path(); 
      app().set_default_data_dir(root / "eosio/nodeos/data" );
      app().set_default_config_dir(root / "eosio/nodeos/config" );
      if(!app().initialize<chain_plugin, http_plugin, net_plugin, producer_plugin>(argc, argv))
         return -1;
      initialize_logging();
      ilog("nodeos version ${ver}", ("ver", eosio::utilities::common::itoh(static_cast<uint32_t>(app().version()))));
      ilog("eosio root is ${root}", ("root", root.string()));
      //重要的代码!!!
      app().startup();
      app().exec();
   } catch (const fc::exception& e) {
   } .....(此处省略多个catch)
   return 0;
}
```

app()位置位于

```
./libraries/appbase/application.cpp
//app()方法返回application的实例(对c++的类的机制不了解，这个实例像是静态对象，反正就是通过返回的实例可以调用相关方法吧，包括startup这个看起来就跟关键的方法)
```

进入到application.cpp中，例举相关方法

+ initialize_..: 初始化插件，功能包括调用各个插件设置命令行参数，以及初始化方法，还有app自身对应初始化操作

- startup:  循环各插件，调用各插件的startup方法
- exec:  表示完全看不懂= =!!。猜测一下，应该是监听io事件，例如结束进程系列..

从这里可以看出，主要的具体的重要的内容都在各插件内部。



#### 插件

eos项目采用了插件管理的结构。插件的目录如下

```
.. eos/plugins:

├── CMakeLists.txt
├── account_history_api_plugin
├── account_history_plugin
├── chain_api_plugin
├── chain_plugin
├── eosio-make_new_plugin.sh
├── faucet_testnet_plugin
├── http_plugin
├── mongo_db_plugin
├── net_api_plugin
├── net_plugin
├── producer_plugin
├── template_plugin
├── txn_test_gen_plugin
├── wallet_api_plugin
└── wallet_plugin
```

完全可以通过命名来理解各插件的含义。要分析的话，直接从相应plugins入手。逐一观察..











