# serverless了解及初步实践
## 一、概念
- serveless是指构建和运行不需要服务器管理的应用程序，从技术角度就是FaaS + BaaS。
- FaaS（Function as a Service函数即服务），通常指将一个无状态的应用代码上传到云端，调用及执行。
- BaaS（Backend as a Service后端即服务），比如云数据库、对象存储、消息队列，这些有状态的服务。
- ”即服务“指的是不以“服务器”的方式来提供服务。
- Serverless 则可以理解为运行在 FaaS 中的，使用了 BaaS 的函数。
- 前端通过函数实现服务端逻辑，后端更靠后。

[![](https://github.com/MangoTi/MangoTi.github.io/blob/master/img/serverless1.jpg)](组成)

## 二、优势
- 开发不需要了解服务器部署和配置，只需要编写函数，可快速迭代，部署函数。
- 平台根据请求自动平行调整服务资源，拥有近乎无限的扩容能力，按需收费，资源空闲不收费。

## 三、运用场景
- 基于 Serverless 的 BFF（Backends For Frontends 其实就是对接口/服务进行管理封装）

[![](https://github.com/MangoTi/MangoTi.github.io/blob/master/img/serverless2.jpg)](BFF)
- 基于 Serverless 的服务端渲染，将路由拆分成函数，部署到FaaS
- 基于 Serverless 的小程序开发（全栈），例如支付就直接提供了很多SDK直接请求BaaS，ctx.mpserverless.db等

## 四、小实践
1. npm install -g serveless 安装成功后会有明显提示
2. 前端/后端项目开发
3. 配置serverless.yml文件
```yml
frontend:
  component: "@serverless/tencent-website"
  inputs:
    region: ap-guangzhou
    code:
      src: dist
      root: frontend
      envPath: src # 相对于 root 指定目录，这里实际就是 frontend/src
      hook: npm run build
    env:
      # 依赖后端部署成功后生成的 url
      apiUrl: ${backend.url}
    protocol: https

backend:
  component: "@serverless/tencent-egg"
  inputs:
    region: ap-guangzhou
    code: ./backend
    functionName: admin-system
    functionConf:
      timeout: 120
    apigatewayConf:
      protocols:
        - https
      environment: release
```
4. serverless --debug部署 项目第一次部署需要扫码授权，腾讯云需开通云函数（小量暂不收费），上传成功后会返回访问地址及其他信息

[![](https://github.com/MangoTi/MangoTi.github.io/blob/master/img/serverless3.png)](结果展示)
[yml配置文档](https://github.com/serverless-components/tencent-scf/blob/master/docs/configure.md "yml配置文档")

## 五、现存问题
- 依赖第三方服务，安全/容量
- 后台语言冷启动时间较长
- 缺乏调试和开发工具
- 存量应用迁移问题

## 参看文献
- [Serverless 掀起新的前端技术变革](https://zhuanlan.zhihu.com/p/65914436 "Serverless 掀起新的前端技术变革")
- [Serverless For Frontend 前世今生](https://zhuanlan.zhihu.com/p/77095720 "Serverless For Frontend 前世今生")
- [权威指南：Serverless 未来十年发展解读 — 伯克利分校实验室分享](https://serverlesscloud.cn/blog/2020-09-24-slsdays-johann-1/ "权威指南：Serverless 未来十年发展解读 — 伯克利分校实验室分享")
- [Serverless + Egg.js 后台管理系统实战](https://serverlesscloud.cn/best-practice/2020-02-07-serverless-admin-system "Serverless + Egg.js 后台管理系统实战")
- [Serverless + Egg.js + 对象存储 COS 构建图片上传应用](https://serverlesscloud.cn/blog/2020-03-31-serverless-egg "Serverless + Egg.js + 对象存储 COS 构建图片上传应用")
- [Serverless——前端的3.0时代](https://zhuanlan.zhihu.com/p/84054729 "Serverless——前端的3.0时代")
