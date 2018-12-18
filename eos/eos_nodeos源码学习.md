## eos/nodeos学习



#### main.cpp

```

      //设置版本
      app().set_version(eosio::nodeos::config::version);
      auto root = fc::app_path(); 
      //设置工作路径
      app().set_default_data_dir(root / "eosio/nodeos/data" );
      app().set_default_config_dir(root / "eosio/nodeos/config" );
      //单节点需要的插件包括以下四个
      if(!app().initialize<chain_plugin, http_plugin, net_plugin, producer_plugin>(argc, argv))
         return -1;
       
      initialize_logging();
      ilog("nodeos version ${ver}", ("ver", eosio::nodeos::config::itoh(static_cast<uint32_t>(app().version()))));
      ilog("eosio root is ${root}", ("root", root.string()));
      app().startup();
      app().exec();
      
      
```



主要是配置和app的一些调用。然后转到app中。

```
./libraries/appbase/application.cpp
```

+ app:  返回application的实例（实例只是声明，木有初始化，对c++类机制不太了解，需进一步了解，目前就理解成这个实例就可访问application::method方法）

+ initialize_..: 初始化插件，功能包括调用各个插件设置命令行参数，以及初始化方法，还有app自身对应初始化操作

+ startup:  循环各插件，调用各插件的startup方法

+ exec:  表示完全看不懂= =!!。猜测一下，应该是监听io事件，例如结束进程系列..

  ​

那么针对单个节点启动必要的4个插件，进行逐个学习。

首先是插件父方法：（在application中调用的方法名直接是start_up，没有plugin前缀，这个问题有待解决..）

+ set_program_options //设置命令行参数

+ plugin_initialize //初始化操作

+ plugin_startup  //启动

+ plugin_shutdown  //退出

  ​



#### Chain_plugin





#### Producer_plugin



```
 // dpos共识  地方 -> get_scheduled_producer的定义在chain_controller
 auto scheduled_producer = chain.get_scheduled_producer( slot );
   // we must control the producer scheduled to produce the next block.
   if( _producers.find( scheduled_producer ) == _producers.end() )
   {
      capture("scheduled_producer", scheduled_producer);
      return block_production_condition::not_my_turn;
   }
```





```go
//chain_controller的一些相关producer方法

//大意猜测更新新区块，包括新区块和签名新区块的producer
update_signing_producer(const producer_object& signing_producer, const signed_block& new_block)

//根据新的producers更新db表？疑问是代码中只有create，没有删除操作
update_or_create_producers( const producer_schedule_type& producers)

//好像这个是关键！详细看一下，对方法的解释为，从投票排名中取出前m个producer,并排除block_signing_key是null的那些，m为配置中的producer_count。然而代码似乎没全..
_calculate_producer_schedule() {
  //get_global_properties返回为global_property_object对象，new_active_producers为对象的熟悉
  //据观察，new_active_producers的赋值目前只出现在wasm的api中，那是不是就可以得出结论new_active_producers是所有的参选producers..这个方法(_calculate_producer_schedule)要做的就是从所有参选者中按排名取前m个，然后去掉block_signing_key是null的那些
   producer_schedule_type schedule = get_global_properties().new_active_producers;
   const auto& hps = _head_producer_schedule();
   schedule.version = hps.version;
   if( hps != schedule )
      ++schedule.version;
   return schedule;
}

//没看懂..好像是给所有producers怎么搞了一下认证(更新到_db某表)
_update_producers_authority()

//返回当前区块头的producer
head_block_producer()

//根据account_name/ownername获取对于producer
get_producer(const account_name& ownername)


//这个方法也蛮重要的，根据slot_num获取已计划好的producer.其中，slot_num始终对应于未来的某一时刻；另外，对于slot_num的值，相对当前多少区块的间隔，例如，slot_num == 1 则为下一个producer, slot_num == 2，则为在一个区块间隔后的下一个producer。注意的是slot_num代表的是区块间隔，非producer间隔
get_scheduled_producer(uint32_t slot_num) {
   const dynamic_global_property_object& dpo = get_dynamic_global_properties();
  //注意点1，最后用来计算位置的值是一个当前的绝对位置 + slot_num  ？具体回头再看
   uint64_t current_aslot = dpo.current_absolute_slot + slot_num;
   ...
  //位置对producer*重复次数的积取模
   auto index = current_aslot % (number_of_active_producers * config::producer_repetitions);
}

//计算producer过去生产的参与率，代码未详
producer_participation_rate()



//大概看了以上方法，有点乱。现在还不能连成线，只是点
关键
update_last_irreversible_block
```



