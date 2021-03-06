ORACLE图片数据抽取工具
======================
开发目标
----------------------
由于ORACLE的图片数据抽取不可以使用游标遍历+并行的方式抽取。所以需要通过消息队列或者kafka来实现达到并行的方式。
那么，自然就得把图片抽取拆分为两步：

    1、主进程获取图片ID放入队列或kafka
    2、从进程从队列或kafka中读取图片ID，通过图片ID从数据库获取图片，进而写入目标数据环境（文件系统或目标数据库）
    
如果数据抽取过程中断，需要通过在Redis中比对出未抽取过的图片ID继续抽取或者通过DBLINK关联目标表和源表过滤出未抽取的图片


软件架构：
----------------------
    1、整个框架是一个任务列表界面管理所有任务，一个任务编辑界面对单一任务进行管理;
    2、在任务列表管理界面要实现添加新任务，及对某个任务进行编辑、删除、停止、启动、同步功能;
    3、单任务管理界面的提供数据源及相应SQL的配置、目标系统或数据库的配置、自动生成相应的SQL;


原理说明：
----------------------
    1、源端数据：数据抽取要实现在源表是一表张(视图)或两张表(视图)的情况，如何生成关于获取图片ID的SQL以及关于获取图片的SQL
    2、抽取任务中断后数据同步的方法：
       1）预加载已抽取的图片的ID到Redis中，获取源表中的图片ID来进行比对，进而是否需要同步
       2）通过DBLINK关联已抽取的表和源端的图片ID的表，进而判断是否需要同步
    3、目标数据：数据抽取到文件系统或目标数据库
