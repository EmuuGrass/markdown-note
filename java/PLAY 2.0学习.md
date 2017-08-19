##PLAY 2.0学习

Tips：

1. play eclipse：生成能导入eclipse的框架工程
2. Play分为开发模式和生产模式两种，而切换的配置在工程下conf/application.conf中：Application.mode=dev 生产模式改为:prod （主要区别在于开发模式中您无需重启server，每次请求都会查看是否有文件发生改变，改变即编译，这对于传统Java EE开发人员无疑是相当敏捷的。而这种方式同样会导致性能下降，所以生产模式中就不会这样了，而是采用预编译机制。 ）
3.
 