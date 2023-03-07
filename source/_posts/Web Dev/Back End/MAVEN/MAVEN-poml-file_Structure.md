# pom.xml Structure

![(project .  CInodeIVersion>4. O.  i transwarD. I earn  (version> I.  <pmpert  (dependenci  (dependency)  d) groupld)  (art i fact I t Id)  (version) I. ](C:\Users\m1518\OneDrive\文档\Document\Web Dev\Back End\MAVEN\MAVEN-poml-file_Structure_PIC00.png)

其中，groupId类似于Java的包名，通常是公司或组织名称

artifactId类似于Java的类名，通常是项目名称

再加上version

一个Maven工程就是由groupId，artifactId和version作为唯一标识。

 

我们在引用其他第三方库的时候，也是通过这3个变量确定。

例如，依赖commons-logging：![ep end enc y)  (art fac t art fac t  (version>l.  c/dependency> ](C:\Users\m1518\OneDrive\文档\Document\Web Dev\Back End\MAVEN\MAVEN-poml-file_Structure_PIC01.png)

使用<\dependency\>声明一个依赖后，Maven就会自动下载这个依赖包并把它放到classpath中。

但是version target is not necessary！

 

但是，并不是所有依赖都需要声明version，