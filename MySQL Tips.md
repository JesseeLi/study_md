## MySQL Tips

#### insert/update json 格式数据

当我们在修改或添加 json格式 的数据时，`mysql会自动进行一层转义`。

使用命令行修改数据时，如果直接 update json 数据，则 mysql会自动进行一层转义 ，导致更新的数据和我们预期不符，json格式错误。

解决方案：

​	使用编程语言，双引号”“ json 数据。

​	将要更新的数据，再次进行一次转义存储。

#### Mysql的缓存

innodb 存储引擎设计了一个 `缓冲池(Buffer Pool)`。

当有了缓冲池后：

- 读取数据时，如果数据存于 `缓冲池(Buffer Pool)` 中，客户端就会直接读取 `缓冲池(Buffer Pool)` ，否则再去磁盘读取。
- 修改数据时，首先修改 `缓冲池(Buffer Pool)` 中所在的页，然后将其设置为 `脏页`，最后由后台线程将脏页写入磁盘。



##### 传统的 LRU算法

- 当访问的页在内存里，就直接把该页对应的 LRU 链表节点移动到链表的头部。
- 当访问的页不在内存里，除了要把该页放入到 LRU 链表的头部，还要淘汰 LRU 链表末尾的页。

无法避免一下问题：

- `预读失效`导致缓存命中率下降；
  - 

- `缓存污染`导致缓存命中率下降；
  - 当我们在批量读取数据的时候，由于数据被访问了一次，这些大量数据都会被加入到「活跃 LRU 链表」里，然后之前缓存在活跃 LRU 链表（或者 young 区域）里的热点数据全部都被淘汰了，**如果这些大量的数据在很长一段时间都不会被访问的话，那么整个活跃 LRU 链表（或者 young 区域）就被污染了**。




##### 预读失效解决方案：

- Linux操作系统实现了 `两个LRU链表`,`活跃 LRU 链表(active_list)` 和 `非活跃 LRU 链表(inactive_list)`。

  - **active_list**，活跃内存页链表，这里存放的是最近被访问过（活跃）的内存页；
  - **inactive_list** 不活跃内存页链表，这里存放的是很少被访问（非活跃）的内存页；

  有了这两个 LRU 链表后，**预读页就只需要加入到 inactive list 区域的头部，当页被真正访问的时候，才将页插入 active list 的头部**。如果预读的页一直没有被访问，就会从 inactive list 移除，这样就不会影响 active list 中的热点数据。

- Mysql 的 InnnoDB 存储引擎是在一个链表上划分两个区域，`young区域` 和 `old区域`。

  - young 区域与 old 区域在 LRU 链表中的占比关系并不是一比一的关系，而是 63:37（默认比例）的关系。

  **划分这两个区域后，预读的页就只需要加入到 old 区域的头部，当页被真正访问的时候，才将页插入 young 区域的头部**。如果预读的页一直没有被访问，就会从 old 区域移除，这样就不会影响 young 区域中的热点数据。

##### 缓存污染解决方案：

- **Linux 操作系统**：在内存页被访问**第二次**的时候，才将页从 inactive list 升级到 active list 里。
- **MySQL Innodb**：在内存页被访问**第二次**的时候，并不会马上将该页从 old 区域升级到 young 区域，因为还要进行**停留在 old 区域的时间判断**：
  - 如果第二次的访问时间与第一次访问的时间**在 1 秒内**（默认值），那么该页就**不会**被从 old 区域升级到 young 区域；
  - 如果第二次的访问时间与第一次访问的时间**超过 1 秒**，那么该页就**会**从 old 区域升级到 young 区域；

提高了进入活跃 LRU 链表（或者 young 区域）的门槛后，就很好了避免缓存污染带来的影响。