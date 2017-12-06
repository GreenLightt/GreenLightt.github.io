---
title: API 测试插件 Postman
date: 2017-12-06 17:26:41
description: 整理关于 Postman 的知识点
tags:
- Postman
categories:
copyright: false
---

具体使用参考官网 [https://www.getpostman.com/docs/](https://www.getpostman.com/docs/);

`Postman` 是 `Google` 开发的一款功能强大的网页调试与发送网页 `HTTP` 请求，并能运行测试用例的的 `Chrome` 插件。其主要功能包括

- 模拟各种 `HTTP` 请求
  从常用的 `GET`、`POST` 到 `RESTful` 的 `PUT` 、 `DELETE` … 等等。 甚至还可以发送文件、送出额外的 `header`。
- `Collection` 功能（测试集合）
  `Collection` 是 `requests` 的集合，在做完一个测试的時候， 你可以把这次的 `request` 存到特定的 `Collection` 里面，如此一来，下次要做同样的测试时，就不需要重新输入。而且一个 `collection` 可以包含多条 `request`，如果我们把一个 `request` 当成一个 `test case`，那 `collection` 就可以看成是一个`test suite`。通过`collection`的归类，我们可以良好的分类测试软件所提供的`API`。而且 `Collection` 还可以 `Import` 或是 `Share` 出来，让团队里面的所有人共享你建立起來的 `Collection`。
- 人性化的 `Response` 整理
  一般在用其他工具来测试的時候，`response` 的内容通常都是纯文字的 `raw`， 但如果是 `JSON`，就是塞成一整行的 `JSON`。这会造成阅读的障碍 ，而 `Postman` 可以针对 `response` 内容的格式自动美化。 `JSON`、 `XML` 或是 `HTML` 都會整理成我们可以阅读的格式。
- 内置测试脚本语言
  `Postman` 支持编写测试脚本，可以快速的检查 `request` 的结果，并返回测试结果。
- 设定变量与环境
  `Postman` 可以自由设定变量与 `Environment`，一般我们在编辑 `request`，校验 `response` 的时候，总会需要重复输入某些字符，比如`url`，`postman`允许我们设定变量来保存这些值。并且把变量保存在不同的环境中。比如，我们可能会有多种环境， `development` 、 `staging` 或 `local`， 而这几种环境中的 `request URL` 也各不相同，但我们可以在不同的环境中设定同样的变量，只是变量的值不一样，这样我们就不用修改我们的测试脚本，而测试不同的环境。

# 官方文档目录
- 运行 Postman
    - [安装和更新](https://www.getpostman.com/docs/postman/launching_postman/installation_and_updates)
    - [发送第一个请求](https://www.getpostman.com/docs/postman/launching_postman/sending_the_first_request)
    - [创建第一个collection](https://www.getpostman.com/docs/postman/launching_postman/creating_the_first_collection)
    - [Postman 面板](https://www.getpostman.com/docs/postman/launching_postman/navigating_postman)
    - [Postman 账户](https://www.getpostman.com/docs/postman/launching_postman/postman_account)
    - [同步数据](https://www.getpostman.com/docs/postman/launching_postman/syncing)
    - [Settings 设置](https://www.getpostman.com/docs/postman/launching_postman/settings)
    - [New 按钮](https://www.getpostman.com/docs/postman/launching_postman/newbutton)
- 发送 API 请求
    - [请求](https://www.getpostman.com/docs/postman/sending_api_requests/requests)
    - [响应](https://www.getpostman.com/docs/postman/sending_api_requests/responses)
    - [历史记录](https://www.getpostman.com/docs/postman/sending_api_requests/history)
    - [API 请求出错诊断](https://www.getpostman.com/docs/postman/sending_api_requests/troubleshooting_api_requests)
    - [调试与日志](https://www.getpostman.com/docs/postman/sending_api_requests/debugging_and_logs)
    - [权限认证](https://www.getpostman.com/docs/postman/sending_api_requests/authorization)
    - [cookie 管理](https://www.getpostman.com/docs/postman/sending_api_requests/cookies)
    - [SSL 证书](https://www.getpostman.com/docs/postman/sending_api_requests/certificates)
    - [捕捉 http 请求](https://www.getpostman.com/docs/postman/sending_api_requests/capturing_http_requests)
    - [拦截器扩展](https://www.getpostman.com/docs/postman/sending_api_requests/interceptor_extension)
    - [代理](https://www.getpostman.com/docs/postman/sending_api_requests/proxy)
    - [生成代码片段](https://www.getpostman.com/docs/postman/sending_api_requests/generate_code_snippets)
    - [发送 SOAP 请求](https://www.getpostman.com/docs/postman/sending_api_requests/making_soap_requests)
- 创建 collection
    - [创建 collection](https://www.getpostman.com/docs/postman/collections/creating_collections)
    - [共享 collection](https://www.getpostman.com/docs/postman/collections/sharing_collections)
    - [管理 collection](https://www.getpostman.com/docs/postman/collections/managing_collections)
    - [示例](https://www.getpostman.com/docs/postman/collections/examples)
    - [数据导入与导出](https://www.getpostman.com/docs/postman/collections/data_formats)
- 介绍脚本
    - [Postman 脚本的介绍](https://www.getpostman.com/docs/postman/scripts/intro_to_scripts)
    - [请求前的脚本](https://www.getpostman.com/docs/postman/scripts/pre_request_scripts)
    - [请求后的脚本](https://www.getpostman.com/docs/postman/scripts/test_scripts)
    - [Test 脚本常用示例](https://www.getpostman.com/docs/postman/scripts/test_examples)
    - [指定 Collection Runder 执行的顺序](https://www.getpostman.com/docs/postman/scripts/branching_and_looping)
    - [Postman 沙盒](https://www.getpostman.com/docs/postman/scripts/postman_sandbox)
    - [Postman 沙盒提供的 API](https://www.getpostman.com/docs/postman/scripts/postman_sandbox_api_reference)
- 环境变量和全局变量
    - [变量](https://www.getpostman.com/docs/postman/environments_and_globals/variables)
    - [环境变量管理](https://www.getpostman.com/docs/postman/environments_and_globals/manage_environments)
    - [全局变量管理](https://www.getpostman.com/docs/postman/environments_and_globals/manage_globals)
- 运行 collections
    - [介绍 collection run](https://www.getpostman.com/docs/postman/collection_runs/starting_a_collection_run)
    - [在 collection run 使用环境变量](https://www.getpostman.com/docs/postman/collection_runs/using_environments_in_collection_runs)
    - [使用 Data files](https://www.getpostman.com/docs/postman/collection_runs/working_with_data_files)
    - [多次执行](https://www.getpostman.com/docs/postman/collection_runs/running_multiple_iterations)
    - [建立工作流](https://www.getpostman.com/docs/postman/collection_runs/building_workflows)
    - [共享 collection run](https://www.getpostman.com/docs/postman/collection_runs/sharing_a_collection_run)
    - [调试 collection run](https://www.getpostman.com/docs/postman/collection_runs/debugging_a_collection_run)
    - [Newman](https://www.getpostman.com/docs/postman/collection_runs/command_line_integration_with_newman)
    - [和 Jenkins 集成](https://www.getpostman.com/docs/postman/collection_runs/integration_with_jenkins)
    - [和 Travis CI 集成](https://www.getpostman.com/docs/postman/collection_runs/integration_with_travis)
    - [docker 运行 newman](https://www.getpostman.com/docs/postman/collection_runs/newman_with_docker)
- 团队仓库
    - [设置一个团队仓库](https://www.getpostman.com/docs/postman/team_library/setting_up_team_library)
    - [共享](https://www.getpostman.com/docs/postman/team_library/sharing)
    - [活动摘要和保存 collections](https://www.getpostman.com/docs/postman/team_library/activity_feed_and_restoring_collections)
    - [搜索](https://www.getpostman.com/docs/postman/team_library/searching)
    - [冲突](https://www.getpostman.com/docs/postman/team_library/conflicts)
- API 文档
    - [API 文档的介绍](https://www.getpostman.com/docs/postman/api_documentation/intro_to_api_documentation)
    - [查看文档](https://www.getpostman.com/docs/postman/api_documentation/viewing_documentation)
    - [环境变量和环境模板](https://www.getpostman.com/docs/postman/api_documentation/environments_and_environment_templates)
    - [用 Markdown 书写文档](https://www.getpostman.com/docs/postman/api_documentation/how_to_document_using_markdown)
    - [发布公共文档](https://www.getpostman.com/docs/postman/api_documentation/publishing_public_docs)
    - [添加和验证自定义域名](https://www.getpostman.com/docs/postman/api_documentation/adding_and_verifying_custom_domains)
    - [添加团队名和 logo](https://www.getpostman.com/docs/postman/api_documentation/adding_team_name_and_logo)
- 监控
    - [对监控的简介](https://www.getpostman.com/docs/postman/monitors/intro_monitors)
    - [设置一个监控器](https://www.getpostman.com/docs/postman/monitors/setting_up_monitor)
    - [查看监控结果](https://www.getpostman.com/docs/postman/monitors/viewing_monitor_results)
    - [监控 API 和站点](https://www.getpostman.com/docs/postman/monitors/monitoring_apis_websites)
    - [设置接收警报](https://www.getpostman.com/docs/postman/monitors/integrations_for_alerts)
    - [关于监控的付费](https://www.getpostman.com/docs/postman/monitors/pricing_monitors)
    - [诊断监控](https://www.getpostman.com/docs/postman/monitors/troubleshooting_monitors)
    - [监控器的常见问题](https://www.getpostman.com/docs/postman/monitors/faqs_monitors)
- 模拟服务器
    - [设置一个模拟服务器](https://www.getpostman.com/docs/postman/mock_servers/setting_up_mock)
    - [示例](https://www.getpostman.com/docs/postman/mock_servers/mocking_with_examples)
    - [使用 Postman Pro API 来 Mock](https://www.getpostman.com/docs/postman/mock_servers/mock_with_api)
    - [匹配逻辑](https://www.getpostman.com/docs/postman/mock_servers/matching_algorithm)






