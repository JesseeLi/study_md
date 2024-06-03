## MySQL Tips

#### insert/update json 格式数据

当我们在修改或添加 json格式 的数据时，`mysql会自动进行一层转义`。

使用命令行修改数据时，如果直接 update json 数据，则 mysql会自动进行一层转义 ，导致更新的数据和我们预期不符，json格式错误。

解决方案：

​	使用编程语言，双引号”“ json 数据。

​	将要更新的数据，再次进行一次转义存储。

