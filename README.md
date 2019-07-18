# impala_udaf
impala的聚合函数实现

impala还是很不错的，效率很高，秒杀hive,spark.MPP果然是把cpu利用到极致。
最近在实现一个自定义聚合函数，用来计算每个房间的上麦时长。因为可能同时有多个人在麦上，但是时长不能累加，要去重。
整体思路
* 1.先按房间和用户分组 select cid,uid, udaf(time,true) as session_stat from table group by cid,uid
* 2.将分组的数据送给udaf自定义聚合函数，在函数中计算出指定时间内该用户的在线时长状态
* 3.第二层sql对2的结果再进去按房间号分组,group by cid,然后再调用udaf自定义聚合函数，对该房间的所有用户的状态进行合并。
* 4.第三层就出来结果了，cid,times

效果还不错，下一步要用scala重写，支持sparkSQL
impala的自定义函数要注意以下几点：
* 1.不要使用malloc,new,free,delete之关，应该全部用它的FunctionContext来做内存的管理，因为FunctionContext中自己实现了内存池，
* 2.要在Serialize和Finalize方法中进行内存清理
* 3.在以上两个方法的入口一定要检查是否is_null!!!
