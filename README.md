#一、前言#

   Java Agent基于字节码增强技术研发，支持自动埋点完成数据上报，Java Agent包含(并二次分发)opentelemetry-java-instrumentation CNCF的开源代码，遵循Apache License 2.0协议，在Java Agent包中对opentelemetry License进行了引用。

说明：
   OpenTelemetry是工具、API 和 SDK 的集合。使用它来检测、生成、收集和导出遥测数据（指标、日志和跟踪），以帮助您分析软件的性能和行为。OpenTelemetry社区活跃，技术更迭迅速，广泛兼容主流编程语言、组件与框架，为云原生微服务以及容器架构的链路追踪能力广受欢迎。通过对Java字节码的增强技术OpenTelemetry-java-instrumentation可以实现自动埋点上报数据,且腾讯云APM基于OpenTelemetry-java-instrumentation进行二次开发,可以让您拿到更完善的调用琏数据及其对应的行号信息, 本文将通过相关操作在腾讯云平台上使用OpenTelemetry-java-instrumentation上报Java应用数据。

#二、前置工作#

在使用OpenTelemetry-java-instrumentation上报Java应用数据之前，您需要准备以下几项工作：
1.登陆应用性能观测，点击最左侧工具栏中的“探针部署”选项：

![image](https://user-images.githubusercontent.com/64143982/180122557-89c204e4-926a-4840-a556-f8cb272e9890.png)

2.在右侧弹出的界面中，依次进行以下操作：
选择部署地点以及上报应用数据所在的业务系统ID。
选择使用语言“Java”。
选择接入方式。“OpenTelemetry”。
选择上报方式：
    内网上报：使用此上报方式，您的服务需运行在腾讯云VPC。通过VPC直接联通，在避免外网通信的安全风险同时，可以节省上报流量开销。
    外网上报：当您的服务部署在本地或非腾讯云VPC内，可以通过此方式上报数据。请注意外网通信存在安全风险，同时也会造成一定上报流量费用。
    自研VPC上报：如果您的服务运行在腾讯自研VPC内，推荐使用此上报方式。自研网络内部直接上报，规避安全风险同时获得高数据通信效率。

点击“下一步，开始部署”:

![image](https://user-images.githubusercontent.com/64143982/180122863-98f71860-47ea-4eae-946b-f11581d0585f.png)

3.进入探针部署页面，获取接入点和Token信息：

![image](https://user-images.githubusercontent.com/64143982/180122905-3a51c21b-5d22-4fe9-9f19-e748c09c8349.png)

三、使用OpenTelemetry Java Agent自动埋点
OpenTelemetry-java-instrumentation支持数十种框架自动埋点能力。更多信息，请参见OpenTelemetry官方文档。
获取Java Agent
打开链接Java Agent，通过命令下载对应的jar包:git clone https://github.com/TencentCloud/tencentcloud-opentelemetry-java.git下载opentelemetry-javaagent.jar：


                                              

        PS:前置要求，如果agent是运行在容器里，需要将宿主机的/usr/opentelemetry/agent/目录挂载至容器内，数据才能上报成功

       使用方式
         2.通过修改Java启动的VM参数上报链路数据。

-javaagent:/path/to/opentelemetry-javaagent.jar    //请将路径修改为您文件下载的实际地址。
-Dotel.resource.attributes=service.name=<appName>,token=<token>
-Dotel.exporter.otlp.endpoint=<endpoint>
如果您选择直接上报数据，请将<token>替换成从前提条件中获取的Token，将<endpoint>替换成对应地域的Endpoint。

注意：替换对应参数值时，“< >”符号需删去，仅保留文本。

例如：

-javaagent:/Users/Downloads/opentelemetry-javaagent.jar
-Dotel.resource.attributes=service.name=ot-java-agent-sample,token=oSmwaUr************ZiNtSv
-Dotel.exporter.otlp.endpoint=http://ap-guangzhou.apm.tencentcs.com:4317
注意：需要输入正确的url。

如果您选择使用OpenTelemetry Collector转发，则需删除-Dotel.exporter.otlp.headers=Authentication=<token>并修改<endpoint>为您本地部署的服务地址。

配置项说明 #
必填项：

otel.resource.attributes=service.name :服务名,如果是spring cloud/dubbo服务，最好与其服务名保持一致
otel.resource.attributes=token :实例token码
otel.exporter.otlp.endpoint：实例上报地址


        3.启动应用。

       登录应用性能观测后，在应用列表页面选择新创建的应用，查看上报数据。

FAQ #
数据上报问题 #
如果碰到平台上没有相关服务的数据上报，可以通过以下几个途径排查一下

服务是否正常成功启动
容器内/usr/opentelemetry/agent/目录是否挂载成功，目录下是否有文件opentelemetry-javaagent.jar
启动参数加上-Dotel.javaagent.auditmode=true，开启debug日志，查看logs目录下的agent日志是否有异常
四、查看结果
注意：当完成应用接入后，您的应用需要有数据请求接入上传后，才可以在应用性能观测页面查看到相应结果。

登入应用性能观测，点击左侧“资源管理”
选择部署应用所在地
单击部署的业务ID
                                  

最终便可以在业务所在的应用列表里看到上报的服务数据：

