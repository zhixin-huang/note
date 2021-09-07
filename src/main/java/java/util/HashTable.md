# HashTable
HashTable与HashMap结构差不多，只是HashTable的一些方法都是同步方法synchronized修饰，不建议使用HashTable，Oracle官方也将其废弃，建议在多线程环境下使用ConcurrentHashMap类。  
与HashMap的一些区别：  
1、默认容量是11  
2、扩容按照2n+1扩容  
3、一些方法时同步方法