---
layout: post
title:  "使用Swagger编写API文档"
description: "介绍使用Swagger工具集定义API文档的过程"
tags: [API, Swagger]
categories: 设计
---

使用到的工具：
  1. swagger-editor: Web版API文档编辑器，可视化输出；
  2. swagger-codegen: 将编写好的文档文件生成服务代码；
  3. swagger-ui：自动生成的服务中集成了swagger-ui，用于展示API文档；
  4. swagger2markup: 将API文档(yaml格式)转换为asciidoc格式；
  5. 使用asciidoc预览工具(atom asciidoc preview或chrome asciidoc preview扩展)以html的形式预览，即可下载为pdf格式；
  
### 使用swagger-editor编写API定义文档
运行swagger-editor:
* 方式一：从DockerHub下载swagger-editor 镜像，启动容器：
<pre>docker pull swaggerapi/swagger-editor
docker run -d -p 8090:8080 swaggerapi/swagger-editor
</pre>
* 方式二：从swagger-editor源码build docker镜像，启动容器：
<pre>
git clone https://github.com/swagger-api/swagger-editor.git
cd swagger-editor
docker build -t swagger-editor .
docker run -d -p 8090:8080 swagger-editor
</pre>
swagger-editor运行之后，从浏览器访问本地服务，打开swagger-editor界面，如图：
![swagger_editor_web.png](/my_blog/images/swagger_editor_web.png)

在web面板中编写API定义，页面左侧为编辑器，默认使用yaml格式，编辑后会实时更新到右侧预览面板，格式或定义错误会在编辑器具体出错行和预览面板顶部给出提示。
  
API定义采用的 OpenAPI规范(默认2.0，最新3.0)，具体编写格式可以参考editor自带的例子以及[OpenAPI官方定义](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/2.0.md)。

### 使用swagger-codegen生成服务代码
当API定义编写完成后，可以直接在swagger-editor界面中生成服务代码，过程如下:
![swagger_code_generation.png](/my_blog/images/swagger_code_generation.png) 

点击语言及框架后，会开始下载对应版本的服务器代码。

### 发布API定义

当服务器代码下载完毕，可以准备开始发布API定义，过程如下：
1. 解压源码包；
2. 安装项目依赖(以nodejs-server为例，nodejs版本需要在4.0.0以上)：
   <pre>cd nodejs-server-server  && npm install</pre>
3. 确认服务端口并启动服务：
   服务入口即源码目录index.js文件，默认端口8080也在其中定义，按照需要修改即可，然后启动服务：node index.js
![api_server_run.png](/my_blog/images/api_server_run.png)

在浏览器中访问相应端口即可：
![show_api_in_browser.png](/my_blog/images/show_api_in_browser.png)

### 将API定义转换为其他格式

当前版本的swagger并没有提供相应的转换工具，如果需要将API定义输出为其他格式(如PDF)需要借助其他工具。
这里介绍swagger2markup的使用，swagger2markup可以将swagger格式(OpenAPI)的API定义转换为AsciiDoc或github风格的Markdown格式，swagger2markup需要java环境，这里仍然使用docker来处理，具体过程：

1. 下载swagger2markup镜像：
   <pre>docker pull swagger2markup/swagger2markup</pre>
2. 执行转换：
   <pre>docker run --rm -v $(pwd):/opt swagger2markup/swagger2markup convert -i /opt/swagger.yaml -f /opt/swagger </pre>
3. 将AsciiDoc转换为HTML或PDF:
  * 使用Atom asciidoc preview工具预览AsciiDoc文件：
    * 使用Atom打开AsciiDoc文件；
    * 启用预览：
    ![start_overview.png](/my_blog/images/start_overview.png)
    
    预览结果：
    ![overview_result.png](/my_blog/images/overview_result.png)
    * 保存为html或pdf(需要安装转换插件)：
    ![save_overview.png](/my_blog/images/save_overview.png)
    
  * 使用Chrom浏览器Asciidoctor.js Live Preview插件进行预览:
    
    * 安装Asciidoctor.js Live Preview插件后，使用Chrome打开AsciiDoc文件:
    ![asciidoctor_overview.png](/my_blog/images/asciidoctor_overview.png)
    * 开启插件进行预览：
    ![asciieditor_start.png](/my_blog/images/asciieditor_start.png)
    然后根据需要可以将页面打印为PDF格式。
    
以上为从定义API到按指定格式输出的完成过程，整个过程比较复杂，用到的组件比较多，如果仅仅是编写API文档可能看起来效率比较低；选择Swagger用来作为API定义工具，主要出于以下考虑：
  1. 规范：Swagger遵循OpenAPI  Specification，编写出的文档有章可循，可以减少不必要的沟通；
  2. 易于测试：编写好的文档可以生成服务器和客户端代码，直接用于Mock测试；