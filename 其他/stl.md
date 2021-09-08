# 1.空间配置器

## 1. new and delete

new中包含两个操作:(1) 调用`::operator new`配置内存 (2) 调用构造函数

delete: (1) 调用析构函数 (2) 调用`operator::delete`释放内存

