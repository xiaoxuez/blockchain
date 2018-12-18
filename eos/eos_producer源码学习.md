## producer源码学习

代码位置为..eos/plugins/producer_plugin/

文件夹结构如下，producer_plugin.hpp为方法的声明， producer_plugin.cpp为具体实现

```
├── CMakeLists.txt
├── include
│   └── eosio
│       └── producer_plugin
│           └── producer_plugin.hpp
└── producer_plugin.cpp

```



虽然producer_plugin.cpp的代码也只有不到400行，但节省时间和资源，就挑出块相关的说吧。方法调用如下

```
plugin_startup -> schedule_production_loop(到没到出块时间，到了就出块,这个方法不断循环) -> block_production_loop -> maybe_produce_block
```

maybe_produce_block方法是最后决定是不是出块的。以下主要解释maybe_produce_block方法

+ 首先会检查区块是否同步到最新。

+ 接下来这段代码会判断是否到达出块时间。

  ```
   uint32_t slot = chain.get_slot_at_time( now );
     if( slot == 0 )
     {
        capture("next_time", chain.get_slot_time(1));
        return block_production_condition::not_time_yet;
     }
  ```


+ 然后就选择出块producer了。

  ```
  auto scheduled_producer = chain.get_scheduled_producer( slot );
  ```

完了判断这个scheduled_producer是不是自己，是的话，再检查点别的(例如私钥啊等)，最后就出块了。

那么，关键代码就是get_scheduled_producer方法。get_scheduled_producer的追踪要到chain_controller.cpp文件中了。位置位于./eos/libriaries/chain/chain_controller.cpp。以下代码未做特殊说明，都位于chain_controller.cpp中。

首先看get_scheduled_producer的代码，可以看到，③通过从db中读出global_property_object对象gpo（下面会有介绍），gpo.active_producers为当前活跃的producers，然后④⑤为算得当前索引下标index，最后根据index在gpo.active_producers中取出相应producer即为返回值。下标计算的方式就是数学算式index=当前区块位置 % （producers总数 * 每个producer重复出多少块）。最后再除以每个producer重复出多少块。比较不理解的是当前区块位置(代码中的current_aslot)的计算方式。

```
//具体代码
/*
* 1. slot_num始终对应于未来的某一时刻
* 2.对于slot_num的值，相对当前多少区块的间隔，
*     例如，slot_num == 1 则为下一个区块的producer, 
*          slot_num == 2，则为在一个区块间隔后的下一个producer。slot_num即为区块间隔数
* 3.使用方法get_slot_time和get_slot_at_time作为slot_num和时间戳的转换
* 4.slot_num == 0，返回EOS_NULL_PRODUCER
*/
account_name chain_controller::get_scheduled_producer(uint32_t slot_num)const
{
   const dynamic_global_property_object& dpo = get_dynamic_global_properties(); //①
   uint64_t current_aslot = dpo.current_absolute_slot + slot_num; //②
   const auto& gpo = _db.get<global_property_object>(); //③
   auto number_of_active_producers = gpo.active_producers.producers.size();
   auto index = current_aslot % (number_of_active_producers * config::producer_repetitions);  //④
   index /= config::producer_repetitions; //⑤
   FC_ASSERT( gpo.active_producers.producers.size() > 0, "no producers defined" );

   return gpo.active_producers.producers[index].producer_name;
}
```



关于global\_property\_object主要属性有active\_producers、new\_active\_producers、pending\_active\_producers。

active\_producers是当前轮的producers, pending\_active\_producers为满足排名等要求的producers, new\_active\_producers为所有可投票(或者是所有备选)的producer。

在controller_chain.cpp中有如下几个方法：

```
//从投票排名中取出前m个producer,并排除block_signing_key是null的那些，m为配置中的producer_count。然而代码似乎没全.
_calculate_producer_schedule() {...}

//更新global_properties
update_global_properties(){...}

//上面提到过的有可能更新active_producers的方法
update_last_irreversible_block(){...}

 /*
 *  After applying all transactions successfully we can update
 *  the current block time, block number, producer stats, etc
 */
_finalize_block(){...}
```

主要通过上面列举的方法介绍active\_producers、new\_active\_producers、pending\_active\_producers是如何关联的。下面一段如果觉得各个方法调用比较蒙的话，可以直接看三个加粗句子，可以了解到active\_producers、new\_active\_producers、pending\_active\_producers之间的关联。

在\_finalize\_block调用的时候，首先调用update\_global\_properties，在update\_global\_properties方法中调用了_calculate\_producer\_schedule方法，calculate\_producer\_schedule方法实现的是**从new\_active\_producers中选出符合条件的producers**，然后回到update\_global\_properties，**根据选出的producers,更新pending\_active\_producers**，然后回到\_finalize\_block方法，调用update\_global\_properties之后调用update_last_irreversible_block方法，**通过pending_active_producers适当更新active\_producers**。

然后，new\_active\_producers是通过wasm的api进行更新。这样就缕通了，通过投票客户端，进行投票，并控制更新new\_active\_producers参数选手，和相应票数。然后根据new\_active\_producers和票数决定blocker producers。

那么，blocker producers的顺序呢？?? 不知道是被我疏忽了还是真的没看到。



producer源码中，可以看到，上层逻辑在 producer_plugin.cpp中，基本的dpos等逻辑还是在chain_controller.cpp中。







