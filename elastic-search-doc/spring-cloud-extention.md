# spring cloud

* ##config
  * spring boot actuator系统监控与管理(应用配置类，度量指标类，操作控制类)<br>
    1：应用配置类<br>
    &nbsp;&nbsp;&nbsp;&nbsp;A、/autoconfig：获取应用的自动化配置报告，其中包括所有自动化配置的候选项，同时还列出了每个候选项是否满足自动化配置的各个先决条件。<br>
    &nbsp;&nbsp;&nbsp;&nbsp;B、/positiveMatches：返回的是条件匹配成功的自动化配置<br>
    &nbsp;&nbsp;&nbsp;&nbsp;C、/negativeMatches：返回的是条件匹配不成功的自动化配置<br>
    &nbsp;&nbsp;&nbsp;&nbsp;D、/beans：该端点用来获取上下文中创建的所有的ban，bean：bean的名称；scope：bean的作用域；type：java类型；resource：class的文件具体路径；dependencies：依赖的bean名称；<br>
    &nbsp;&nbsp;&nbsp;&nbsp;E、/configprops：获取应用上配置的属性信息报告。<br>
    &nbsp;&nbsp;&nbsp;&nbsp;F、/env：获取应用所有可用的环境属性报告，包括环境变量、JVM属性、应用的配置属性、命令行中的参数。它可以方便的看到当前应用加载的配置信息。<br>
    &nbsp;&nbsp;&nbsp;&nbsp;G、/mappings：返回所有spring MVC的控制器映射关系报告<br>
    &nbsp;&nbsp;&nbsp;&nbsp;H、/info：返回一些应用的自定义信息。<br>
    2：度量指标类<br>
    &nbsp;&nbsp;&nbsp;&nbsp;A、/metrics：返回当前应用的种类重要度量指标，如：内存信息、线程信息、垃圾回收信息等。<br>
    &nbsp;&nbsp;&nbsp;&nbsp;B、/health：获取应用的各类健康指标信息<br>
    &nbsp;&nbsp;&nbsp;&nbsp;C、/dump：暴露程序运行中的线程信息。<br>
    &nbsp;&nbsp;&nbsp;&nbsp;D、/trace：返回基本的http跟踪信息。保留最近的100条请求记录<br>
    3：操作控制类<br>
    