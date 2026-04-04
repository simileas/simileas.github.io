+++
title = "Spec Kit 的一些探索"
date = "2025-12-06"
draft = true
tags = ["SPEC"]
categories = ["开发备忘"]
+++

自从 LLM 表现出不可思议的智能，我一直思考我的工作在某一天会以什么样的方式被取代。

参考 Specification-Driven Development，对于产品经理来说，AI 与程序员功能相近；输入一个产品需求文档，输出应用程序代码或者二进制发行包。最终上线的程序形态不一定与产品经理的最初的想象一致，这是一个渐进的过程，需要多次迭代和更新，我想这是 SDD 的核心思想之一。

# 简单的流程描述

安装并创建 Spec Kit 项目：

```bash
uv tool install specify-cli --from git+https://github.com/github/spec-kit.git
uv specify init import-contact
```

用 vscode 打开项目，然后在 CHAT 面板中选择 Spec Kit。

**1. 创建章程**

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

**2. 指定需求**

接下来，可以指定需求，主要描述我们想做什么功能，及其动机：

```
/speckit.specify 创建一个聚焦于人际关系推荐的客户管理的 Web 应用程序，允许用户存储和管理客户信息，以提高客户管理的效率；主要用于为人际关系顾问提供工具，帮助他们为用户匹配到合适的联系人。管理客户基础信息、跟进记录和联系历史、标签和合同管理、续费提醒等功能。
```

**3. 指定技术架构**

使用 /speckit.plan 命令，指定技术架构：

```
/speckit.plan 为这个客户管理应用程序设计一个技术架构，使用 React + Ant Design 作为前端框架，Spring Boot、Kotlin 作为后端技术栈，MySQL 作为数据库
```

**4. 创建任务**

使用 /speckit.tasks 命令，创建任务列表：

```
/speckit.tasks
```

**5. 生成实现代码**

使用 /speckit.implement 命令，生成实现代码：

```
/speckit.implement
```

这些命令更多的是一个约定，用于规范化我们与 AI 工具的交互方式；现实开发中，我们也需要严格执行对功能的规划，让开发过程更加可控。

我们认为 AI 不能完全区分哪些是规划，哪些是对任务的控制，哪些是对实现方式的修正，需要明确地指定功能和任务的边界。

当然最高级的 AI 可能只需要指定功能和任务的边界，就能够自动完成规划、任务控制和实现方式的修正，但是需求在使用自然语言传递的过程中，仍然存在歧义和不确定性，需要通过迭代和反馈不断完善。

那么这里就有一些问题存在：

**1. 在 specify 阶段，我们先指定一个大的功能范围，然后逐步细化功能和任务的边界，即功能是循环细化和验证的过程，那么这个过程我们如何控制？**

**2. AI 在理解需求的过程中，添加了一些不需要实现的功能或任务，我如何让 AI 删除这些不必要的功能或任务？**

**3. AI 在执行任务的过程中生成了大量的输出性的描述，我是否可以让 AI 整理这些描述，删除不必要的过程性描述？相当于简化物理上的上下文，从而更清晰地呈现功能和任务的边界。**

**4. 在现阶段 Vibe Coding 的过程中，我们还需要手动改写代码，确保代码能正常工作吗？如果我与 AI 混合开发，那么我必须能够理解任意一行代码的作用和逻辑**

**5. 在混合开发过程中，如何有效地分配 AI 和人类开发者的工作，以提高开发效率和代码质量？**

