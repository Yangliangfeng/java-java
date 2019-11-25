### java 扩展

* jsoup扩展
```
1. 下载地址
    https://jsoup.org/download

2. 文档地址
    https://github.com/jhy/jsoup
```
* 安装maven（maven是一个若干插件和工具组件组成的一个框架）
```
1. 下载地址
    http://maven.apache.org/download.cgi

2. 配置环境变量
    MAVEN_HOME   D:\tool\apache-maven-3.5.0
    PATH 加入 ;%MAVEN_HOME%\bin
    
    查看版本信息： mvn -v
    
3. 创建项目
    mvn -B archetype:generate -DgroupId=com.jtthink -DartifactId=htmlparser
    
    GroupID:项目唯一的标识符，对应初始项目包
    ArtifactID:项目唯一的标识符，好比项目名称

4. 进入项目目录---下载依赖
    mvn  dependency:copy-dependencies 
    
    Windows下 默认会下载到 
    C:\Users\Administrator\.m2\repository下   ，这叫做本地仓库

5. 查看maven的插件
    http://maven.apache.org/plugins/index.html

6. 修改本地仓库位置
    maven目录下的config文件夹中,里面有个默认的配置文件叫做settings.xml
    加入配置节：
    <localRepository>F:\pro\mvnpro\mvnrepo</localRepository>
    
    再次执行：
    mvn  dependency:copy-dependencies
    就可以看见，maven的中央仓库变成我们自定义的了

7. 默认的中央仓库
    http://repo1.maven.org/maven2/ 

8. 搜索相关软件的地址
   http://search.maven.org/ 
   http://mvnrepository.com/  
   http://www.mojohaus.org/plugins.html

9. 项目的编译
    进入项目目录：
    mvn compile

10. 项目的执行
    mvn exec:java -Dexec.mainClass="com.jtthink.App" 
    
11. maven的生命周期
    1）Clean  构建之前进行一些清理工作
        （1）pre-clean  执行一些需要在clean之前完成的工作
        （2）clean  移除所有上一次生成的文件 （mvn clean 就是清理，不会帮我们编译）
        （3）post-clean  执行一些需要在clean之后立刻完成的工作
    
    2) Default 如编译，测试，打包，部署等等
       （1） process-resources     复制、打包资源文件
       （2） compile   编译项目
       （3） test      运行测试（需要配置测试框架。暂不指定）
       （4） package      打包成可发布的格式，如 JAR 、war
       （5） install     将包安装至本地仓库 
       （6） deploy     将最终的包复制到远程的仓库

    3）Site  生成项目报告，站点，发布站点
```
