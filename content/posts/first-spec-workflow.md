+++
title = "Spec Kit 的一些探索"
date = "2025-12-06"
draft = true
tags = ["SPEC"]
categories = ["开发备忘"]
+++

自从 LLM 表现出不可思议的智能，我一直思考我的工作在某一天会以什么样的方式被取代。

参考 Specification-Driven Development，对于产品经理来说，AI 与程序员功能相近；输入一个产品需求文档，输出应用程序代码或者二进制发行包。最终上线的程序形态不一定与产品经理的最初的想象一致，这是一个渐进的过程，需要多次迭代和更新，我想这是 SDD 的核心思想之一。

安装并创建 Spec Kit 项目：

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
uv specify init import-contact
```

用 vscode 打开项目，然后在 CHAT 面板中选择 Spec Kit。

# 创建章程

在 CHAT 面板中输入以下内容：

```
/speckit.constitution Create principles focused on code quality, testing standards, user experience consistency, and performance requirements
```

/speckit.constitution 是一个 Spec Kit 命令，本来应该在 AI 工具的 CLI 中执行，例如 Gemini CLI / Copilot CLI，但 Spec Kit 允许在 VS CODE CHAT 面板中直接使用。

这句话的意思是「创建一套专注于代码质量、测试标准、用户体验一致性和性能要求的原则」。

也可以使用中文输入：

```
/speckit.constitution 创建一套专注于代码质量、测试标准、用户体验一致性和性能要求的原则
```

章程创建完成后，可以在 `spec/constitution.md` 文件中看到生成的章程内容。

# 指定需求

接下来，可以指定需求，主要描述我们想做什么功能，及其动机：

```
/speckit.specify 创建一个聚焦于人际关系推荐的客户管理的 Web 应用程序，允许用户存储和管理客户信息，以提高客户管理的效率；主要用于为人际关系顾问提供工具，帮助他们为用户匹配到合适的联系人。管理客户基础信息、跟进记录和联系历史、标签和合同管理、续费提醒等功能。
```

# 指定技术架构

使用 /speckit.plan 命令，指定技术架构：

```
/speckit.plan 为这个客户管理应用程序设计一个技术架构，使用 React + Ant Design 作为前端框架，Spring Boot、Kotlin 作为后端技术栈，MySQL 作为数据库
```

# 创建任务

使用 /speckit.tasks 命令，创建任务列表：

```
/speckit.tasks
```

# 生成实现代码

使用 /speckit.implement 命令，生成实现代码：

```
/speckit.implement
```

