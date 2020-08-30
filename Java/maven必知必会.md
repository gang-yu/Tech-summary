> Q: maven中的dependencies 与dependencyManagement 有什么区别？

所有声明在父项目中 dependencies 里的依赖都会被子项目自动引入，并默认被所有的子项目继承。
dependencies 即使在子项目中不写该依赖项，那么子项目仍然会从父项目中继承该依赖项（全部继承）

dependencyManagement 里只是声明依赖，并不实现引入，因此子项目需要显示的声明需要用的依赖。如果不在子项目中声明依赖，是不会从父项目中继承下来的；只有在子项目中写了该依赖项，并且没有指定具体版本，才会从父项目中继承该项，并且 version 和 scope 都读取自父 pom; 另外如果子项目中指定了版本号，那么会使用子项目中指定的jar版本。

**dependencyManagement声明jar版本供参考，不会下载**
**dependencies强制下载jar**
dependencyManagement起到管理依赖jar版本号的作用，一般只会在项目的最顶层的pom.xml中使用到，所有子module如果想要使用到这里面声明的jar，只需要在子module中添加相应的groupId和artifactId即可，并不需要声明版本号 



> Q: Maven依赖中要掌握哪些核心概念？

依赖的调解

​	短路径优先/第一声明原则

依赖范围/依赖的传递

​	compile\provided\runtime\test\system\import

依赖管理

​	dependencyManagement

导入依赖

系统依赖

http://ifeve.com/maven-dependency-mechanism/




> Q: 为什么要用BOM？

​        使用BOM除了可以方便使用者在声明依赖的客户端时不需要指定版本号外，最主要的原因是可以解决依赖冲突。用以统一间接或者直接依赖的类库版本，强制某个类库使用某一个统一的版本。

考虑以下的依赖场景：

```
项目A依赖项目B 2.1和项目C 1.2版本： 
项目B 2.1依赖项目D 1.1版本； 
项目C 1.2依赖项目D 1.3版本；
```

​        在该例中，项目A对于项目D的依赖就会出现冲突，按照`maven dependency mediation`的规则，最后生效的`可能`是:项目A中会依赖到项目D1.1版本（就近原则，取决于路径和依赖的先后,和Maven版本有关系）。
​       在这种情况下，由于项目C依赖1.3版本的项目D，但是在运行时生效的确是1.1版本，所以在运行时很容易产生问题，如 NoSuchMethodError, ClassNotFoundException等。



> Q: BOM如何使用？

​	BOM本质上是一个普通的POM文件，区别是对于使用方而言，生效的只有`<dependencyManagement>`这一个部分。

​	BOM的维护方负责版本升级，并保证BOM中定义的jar包版本之间的兼容性。

​	父工程packaging不一定是pom，也可以是jar和war。

​	子模块则无需指定版本信息；子模块可以继承多个父模块。

https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#bill-of-materials-bom-poms

> 这里有一个例子，3个工程的parent关系是
>
> xx-bom(bom)  <---  xx-parent(pom)  <--- xx-project1 (jar)

也有

![img](https://img-blog.csdn.net/20150721204949922?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)



> Q: provided会从父工程传递到子工程么？

​	父模块定义的是provided时，子模块在引用时要小心。

provided是没有传递性的，也就是说，如果你依赖的某个jar包，它的某个jar的范围是provided，那么该jar不会在你的工程中依靠jar依赖传递加入到你的工程中。



> Q: BOM指定了版本，子工程要写明type么？

如果是非jar，要指明。

```xml
<project>  ...  
  <dependencies>    
    <dependency>      
      <groupId>group-c</groupId>      
      <artifactId>artifact-b</artifactId>
      <!-- This is not a jar dependency, so we must specify type. -->
      <type>war</type>    
    </dependency>     
    <dependency>      
      <groupId>group-a</groupId>      
      <artifactId>artifact-b</artifactId>  
      <!-- This is not a jar dependency, so we must specify type. -->    
      <type>bar</type>   
    </dependency>  
  </dependencies>
</project>
```

*与依赖管理元素匹配的依赖引用最小信息集是{groupId, artifactId, type,  classfier}。许多情况下，依赖指向的jar不需要指定classfier。因为默认type是jar，默认classfiler为空，所以我们可以把信息集设置为{groupId, artifactId}。*

http://ifeve.com/maven-dependency-mechanism/

https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html



> Q: 依赖关系要注意什么？

注意保证各个jar的version的稳定。尽量明确version是最好的。保证每次build都是使用预想的version。





