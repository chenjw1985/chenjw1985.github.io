---
layout: post
title: Team development guidelines
subtitle: ""
date: 2020-2-14 10:00
author: "David Chen"
tags:
	- Work
	- Team
---

> 为了团队开发的规范化，会对团队使用的工具有一定的约束性。
> 本文档基于 Markdown 编写。

## Table of contents
* [Scrum 实践](#Scrum-实践)
* [版本管理工具](#版本管理工具-Git)
* * [Git flowchart](#Git-flowchart)
* * [分支管理](#分支管理)
* * [版本号规范](#版本号规范)
* * [Commit messages规范](#Commit-messages规范)
* * [发布流程](#发布流程)
* [开发工具](#开发工具)
* * [Java 开发](#Java-开发)
* * [前端开发](#前端开发)
* * [Go 开发](#Go-开发)

## Scrum 实践

1. 两周为一个 sprint 周期，当前 sprint 为全开发周期。
2. QA 在 sprint 周期内负责编写 test point 和测试上一个 sprint 的功能。
3. Dev 基于 develop 分支开发当前 sprint 的功能，基于 release 分支进行上一个 sprint 的 bugfix。
4. sprint 周期的最后一天需要进行 review meeting，dev 向 qa 演示新功能。
5. 对于 task 的时间评估，需要考虑包含 50% 单元测试覆盖率和 code review 的时间。

---
## 版本管理工具 `Git`

1. 建议大部分操作都在命令行终端进行，如 `pull`、`push`、`commit`、`checkout` 等操作。
2. 严格遵循使用规范。
3. 使用 `git merge --no-ff` 合并代码。
4. 任何合并到公共分支的代码都需要提交 `pull request`,需要经过 `code review`。
5. 建议先 `create draft pull request` 再正式邀请 reviewer。
6. 合并的时候请选择 rebase merge。
7. develop -> release 需要 project owner 执行，每个 project 至少会有三个 owner。

---
### Git flowchart

![gitflow-model.src.002.jpeg](../pics/gitflow-model.jpeg)

---
### 分支管理

Core team 主要使用 2 个实体分支（`master`,`develop`）和 3 个临时性分支（`feature`,`release`,`hotfix`）。

`master` 分支
> master 为主分支，始终保持最后一次 release 发布的代码。不会基于该分支进行功能开发，该分支基于 release 创建。

`develop` 分支
> develop 为开发分支，始终保持最新完成以及 bug 修复后的代码。develop 分支是创建 feature 的基线分支。

`feature` 分支
> 开发新功能时，以 develop 为基础创建 feature 分支。Select 使用 JIRA 进行任务管理，feature 分支的命名以 JIRA 任务 ID 为命名。例如：SELECT-2263。
> 强制要求每 2 天同步一次 develop 的代码到 feature 分支，避免最后提交时出现比较大的冲突。
> 在提交你的代码到 develop 分支前，请先确认三件事：
> 1. `该功能是经过测试的。`
> 2. `该功能的单元测试覆盖率不低于 50%。`
> 3. `该功能的代码符合遵循的代码规范。`

`release` 分支
> release 为预上线分支，发布测试阶段，会从 release 分支代码为基准进行测试和发布。
> 每次 sprint 结束时，project owner 会基于 develop 分支创建用于 QA 测试的 release 分支。
> 正式发布 release 后，需要将 release 的代码推送到 master 分支并创建版本 tag。

`hotfix` 分支
> 线上出现紧急问题时，基于对应版本 tag 创建 hotfix 分支进行代码修复。完成修复后需要合并代码到 develop 分支，并创建一个包含`新修正版本号`的 tag 分支。
> 如其它 tag 有同样问题，则需要从目标 tag 建立分支，然后 cherry-pick hotfix 的代码到 tag 代码进行修复，并生成一个包含`新修正版本号`的 tag 分支。

---
### 版本号规范

规范：`主版本号 . 子版本号 . 修正版本号 [. 编译版本号 ]`
- 主版本号：第一个数字，产品改动较大，可能无法向后兼容（要看具体项目）。
- 子版本号：第二个数字，增加了新功能，向后兼容。
- 修正版本号：第三个数字，修复 bug 或优化代码，一般没有添加新功能，向后兼容。
- 编译版本号：通常是系统自动生成，每次代码提交都会导致自动加 1。

例如：
- 1.0
- 2.14.0.1478
- 3.2.1 build-354

版本号修饰词：
- alpha: 内部测试版本，bug 较多，一般用于开发人员内部交流。
- beta: 测试版，bug 较多，一般用于热心群众测试，并向开发人员反馈。
- rc: release candidate，即将作为正式版发布，正式版之前的最后一个测试版。
- ga：general availability，首次发行的稳定版。
- r/release/ 或干脆啥都不加：最终释放版。
- lts: 长期维护，官方会指定对这个版本维护到哪一年，会修复所有在这个版本中发现的 BUG。

版本号管理策略：
- 项目初始版本号可以是 0.1 或 1.0。
- 项目进行 BUG 修正时，修正版本号加 1。
- 项目增加部分功能时，子版本号加 1，修正版本号复位为 0。
- 项目有重大修改时，主版本号加 1。
- 编译版本号一般是编译器在编译过程中自动生成的，只需要定义格式，并不需要人为控制。

---
### Commit messages规范

> 为保证团队合作的高效性，core team 需要遵循可读友好的 commit messages 日志规范。Core module 会基于 Github 的 webhook 实现对 commit messages 的自动检查，自动reject 不符合 commit 规范的 pull request。

Commit messages 格式要求: `{Type}:{50个字符以内，描述主要变更内容}`，
具体的Type类别说明：
- feat: 添加新特性。
- fix: 修复 bug。
- docs: 仅仅修改了文档。
- style: 仅仅修改了空格、格式缩进、都好等等，不改变代码逻辑。
- refactor: 代码重构，没有加新功能或者修复 bug。
- perf: 增加代码进行性能测试。
- test: 增加测试用例。
- chore: 改变构建流程、或者增加依赖库、工具等。
- revert: 撤销。
- close: 关闭 issue。
- release: 发布版本。

---
### 发布流程

Core module 基于 `develop -> release -> master/tag`的推送流进行版本发布管理：
- `develop` 分支用于 Dev 进行开发和测试。
- `release` 分支用于 QA 测试及 bugfix。
- `master/tag` 分支用于产品发布和 hotfix。
- Jenkins 需要支持基于 `tag` 进行版本发布的流程设置。
- Core module 固定发布周期（2周），非发布窗口期不 release 版本。
- Core module 需要保证在每个大版本号内 API 向下兼容。

---
## 开发工具

> Core team 并不限制开发工具的使用，但是需要保证代码风格的一致性。以下为建议使用的工具和插件。
> **代码规范**为强制执行。

### Java 开发

建议统一使用 `IDEA` 作为 java 开发工具，代码规范遵循 Google java 开发规范。
- [了解更多 IDEA Java CheckStyle Guide！](https://docs.google.com/document/d/15xq3wQG-CQMCv4_n-eDN6A5mo-sr-2g1D_6uZ1sH2MY/edit?usp=sharing)
- [了解更多 Google Java Style Guide！](https://google.github.io/styleguide/javaguide.html)

---
### 前端开发

建议统一使用 `VSCode` 作为前端开发工具，代码规范遵循前端团队构建的规范。
- [了解更多 NT Frontend Coding Style Guide！](https://docs.google.com/document/d/1WIAK7YYFaVTTsV4vsDu9JrU_oj0iq-_rZ4i1Nk715PM/edit?usp=sharing)

---
### Go 开发

建议统一使用 `GoLand` 作为 go 开发工具，代码规范遵循 Uber 构建的规范。
- [了解更多 Uber Go语言编程规范！](https://drive.google.com/open?id=1yLx2OLeC0rj5A6uSuEbkAQeKOmjK6WlX) 