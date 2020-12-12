### 什么是uber-jar？

uber==super

打入所有依赖包的包

### 什么maven shade plugin？

maven-shade-plugin提供了两大基本功能：

1. 将依赖的jar包打包到当前jar包（常规打包是不会将所依赖jar包打进来的）；
2. 对依赖的jar包进行重命名（用于类的隔离）；

https://blog.csdn.net/yangguosb/article/details/80619481

https://maven.apache.org/plugins/maven-shade-plugin/examples/attached-artifact.html