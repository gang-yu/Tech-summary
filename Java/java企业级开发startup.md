
现在的java企业开发，有一套标准化的要求。
比如，包管理都使用maven、nexus仓库等来管理。如此诸类。

## Maven
## Nexus
## java工程BOM
## java工程骨架

约定大于配置

src          ——>     				源代码和测试代码的根目录

​			main              			应用代码的源目录

​					java           		源代码

​								controller

​								facade

​								service

​								repository

​								dao

​								infrastructure

​								util

​					resources      	项目的资源文件

​			test                			 测试代码的源目录

​					java           		 测试代码

​					resources      	测试的资源文件

target                  					编译后的类文件、jar文件等	

## java部署的assembly结构

​	/opt

​			/lib

​			/bin

​			/conf

​			/logs

