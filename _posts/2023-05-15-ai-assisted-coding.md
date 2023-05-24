---
layout: post
title:  "AI assisted coding"
date:   2023-05-15 17:10:00
categories: ai
tags: vscode ai copilot codeium
---

* content
{:toc}


随着 ChatGPT 等一系列颠覆性 AI 技术的出现，我们的日常工作生活与 AI 的联系也日渐紧密。AI 辅助编码技术就是其中的一个例子。作为一名长期使用 Copilot 的用户，应公司领导和同事的邀请，分享我的经验和总结，以帮助大家更好地了解和使用 AI 辅助编码技术，在工作中更加得心应手。





### 摘要

本文旨在通过对ai辅助编码的发展脉络、基本原理、应用场景、使用技巧等方面的介绍，深入了解和使用ai辅助编码技术。

### 发展历程

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/ai_assisted_coding/AI%20Assisted%20Coding%20development%20history-Map%201.png)

### 基本原理（第四代）

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/ai_assisted_coding/AI%20assisted%20coding%20basic%20principle-Map%201.png)

### 常见应用场景

![image](https://raw.githubusercontent.com/zengzzzzz/zengzzzzz-img/main/ai_assisted_coding/ai%20assisted%20coding-apllication%20scene.png)

### 对比分析

#### 对比分析表格

| Parameter                       | Tabine                                                       | Kite                                                       | Github Copilot                                               | Codeium                                                      | AWS Whisperer                                                |
| ------------------------------- | ------------------------------------------------------------ | ---------------------------------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| Offical website                 | https://www.tabnine.com                                      | https://www.kite.com/blog/product/kite-is-saying-farewell/ | https://github.com/features/copilot                          | https://codeium.com/                                         | https://aws.amazon.com/cn/codewhisperer/                     |
| Price                           | Basic：免费 只提供短代码补全 <br> Pro: $12/month 全行/完整方法代码补全 | 免费 November 16, 2022 停止维护                            | $10/month $100/year                                          | 个人用户免费                                                 | 个人用户免费 / 安全扫描（50次/月）                           |
| Open source compliance          | 训练模型代码完全透明，提供完整归属，允许用户查看并避免未来法律问题 | https://github.com/kiteco open-source / 训练代码不透明     | openai codex 不透明                                          | NOT GPL                                                      | 不透明                                                       |
| Code privacy                    | 不存储共享用户代码                                           | 非私有部署会上传用户代码                                   | 可选                                                         | 可选                                                         | 不清楚                                                       |
| Language support                | C, C++, C#, CSS, Dart, Go, Haskell, Java, JS, Kotlin, Perl, PHP, Python, Ruby, Rust, Scala, TS | C, C++, C#, Go, Java, JS, Python, Ruby, TS，python（best） | C, C++, C#, Go, Java, JS, PHP, Python, Ruby, Scala, TS, more with potentially lower quality | Assembly, C, C++, C#, Clojure, CMake, CoffeeScript, CSS, CUDA, Dart, Delphi, Dockerfile, Elixir, F#, Go, Groovy, Haskell, HCL, HTML, Java, JavaScript, Julia, JSON, Kotlin, LISP, Less, Lua, Makefile, MATLAB, Objective-C, pbtxt, PHP, Protobuf, Python, Perl, Powershell, R, Ruby, Rust, Sass, Scala, SCSS, shell, Solidity, SQL, Starlark, Swift, Typescript, TSX, VBA, Vue, YAML, more with potentially lower quality | python、java、JS、TS、C#                                     |
| IDE support                     | VSCode, JetBrains, Jupyter / Colab / Deepnote / Databricks Notebooks, Vim / Neovim, | VS Code、JupyterLab、Vim、Sublime、Atom、JetBrains         | VSCode, Visual Studio, Vim/Neovim, JetBrains                 | VSCode, JetBrains, Jupyter / Colab / Deepnote / Databricks Notebooks, Vim / Neovim, Emacs, VSCode Web IDEs (ex. Gitpod), Chrome Extension | pycharm 、 vscode、in telliJ、AWS cloud9、AWS lambda Console |
| Code completion                 | whole-line / full-function / Natural language                | 全代码补全、文档检索                                       | whole-line / full-function                                   | whole-line / full-function                                   | whole-line / full-function                                   |
| Train Ai models on private code | Y                                                            | Y                                                          | N                                                            | Y                                                            | N                                                            |
| Generate comments               | N                                                            | N                                                          | Y                                                            | Y                                                            | Y                                                            |
| Code refator                    | N                                                            | N                                                          | N                                                            | Y                                                            | N                                                            |
| Generate Unitest                | N                                                            | N                                                          | Y                                                            | Y                                                            | N                                                            |
| Code example                    | Y （java / javascript）                                      | Y                                                          | Y                                                            | Y                                                            | Y                                                            |
| Explian code                    | N                                                            | N                                                          | N                                                            | Y                                                            | N                                                            |

#### 结论

上述表格列举了第四代主流的AI assisted Coding 的一些情况， 结合我们现状而言，Kite 已停止更新，自然不在我们的选择内；AWS whisperer 会上传用户代码进行分析，也需要排除掉；Tabine 为 Codota 旗下产品自然对java支持最完善，故不会是我们首选；则在我们的选择范围内仅有Github Copilot 与 codeium，综合 codeium 免费且可私有化部署，具有强大的功能（应该与copilot X 是同一代产品），则为我们的首选。此外还有一些其他的AI assisted Coding 工具，类似微软的IntelliCode，阿里云Cosy等。

### 未来发展

1. AI 会自动检测和修复常见的bug。通过分析大量代码库,AI 系统可以检测出常见的 anti-pattern 和 bug,并提供自动修复建议。这可以减少 debug 的时间。
2. 整个代码库的检测和重构。AI 可以分析整个代码库,检测出不合理的架构、重复代码、未使用的变量等,并提供重构方案。这可以提高代码的整体质量和可维护性。
3. AI will 辅助生成测试用例。通过分析代码和业务逻辑,AI 可以自动生成覆盖面的测试用例。这可以减轻测试工作量,同时提高测试质量。
4. 基于需求代码生成。对于一些模板化的代码,AI 会学习并理解开发者的编程风格,然后基于需求自动生成草稿代码。开发者只需要调整和完善,可以显著提高工作效率。

### 参考文章

https://github.com/features/copilot/

https://docs.github.com

https://books.google.com.hk/books?hl=zh-CN&lr=&id=NuxcEAAAQBAJ&oi=fnd&pg=PA1&dq=AI+Assisted+Coding+development+history&ots=TkmVmKaW9T&sig=ZK837PIEVDdJACSVmNgGDJOlfaE&redir_esc=y#v=onepage&q&f=false

https://www.tabnine.com/

https://www.tabnine.com/blog/tabnine-vs-codeium/

https://www.tabnine.com/blog/github-copilot-alternatives/

https://zhuanlan.zhihu.com/p/456957593

https://devclass.com/2022/11/21/kite-ai-coding-pulled-down-to-earth-because-our-500k-developers-would-not-pay-to-use-it-now-open-source/

https://www.prnewswire.com/news-releases/kite-releases-team-server-an-enterprise-grade-self-hosted-machine-learning-engine-that-provides-personalized-code-completion-at-scale-301221151.html

https://codeium.com/blog/copilot-trains-on-gpl-codeium-does-not

https://codeium.com/security

https://codeium.com/blog/code-assistant-comparison-copilot-tabnine-ghostwriter-codeium

https://codeium.com/playground

https://aws.amazon.com/cn/codewhisperer/

https://github.com/features/preview/copilot-x

https://developer.aliyun.com/tool/cosy?channel=plugin_contact

https://visualstudio.microsoft.com/zh-hans/services/intellicode/

claude

chatGPT-3.5