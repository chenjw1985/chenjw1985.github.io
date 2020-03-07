# Select Team Git Guidelines

## Git 使用规范

1. 建议大部分操作都在命令行终端进行，如 `pull`、`push`、`commit`、`checkout` 等操作。
2. 任何临时分支合并到固定分支的代码都需要提交 `pull request`,需要经过 `code review`。
3. 建议先 `create draft pull request` 自己检查下，，再正式邀请 reviewer。
4. `develop -> release -> master/tag` 需要 project owner 执行，每个 project 至少会有 2 个 owner。
5. `develop -> feature` 使用 `rebase merge` 合并代码。
6. `feature -> develop` 使用 `squash merge` 或者 `merge --no-ff` 合并代码。
7. `develop -> release` 使用 `merge --no-ff` 或者 `squash merge` 合并代码。
8. 时间跨度大于 `2` 天的 `feature`，需要每 2 天合并一次 `develop` 分支。
9. `hotfix` 需要 `cherry-pick` 回 `develop` 分支和 `release`（如存在）。
10. `release` 用于 `QA、CERT、PROD` 发布，发布完成后合并到 `master` 并创建 `tag`。

---

### Git flowchart

![Alt text](../pics/gitflow-model.jpeg)

---
### 分支管理

Select team 主要使用 2 个实体分支（`master`,`develop`）和 3 个临时性分支（`feature`,`release`,`hotfix`）。

#### `master` 分支
master 为主分支，始终保持最后一次 release 发布的代码。不会基于该分支进行功能开发，该分支基于 release 合并。

#### `develop` 分支
develop 为开发分支，始终保持最新完成以及 bug 修复后的代码。develop 分支是创建 feature 的基线分支。

#### `feature` 分支
开发新功能时，以 develop 为基础创建 feature 分支。Select 使用 JIRA 进行任务管理，feature 分支的命名以 JIRA 任务 ID 为命名。例如：SELECT-2263。
对于超过2天的 feature，建议每 2 天同步一次 develop 的代码到 feature 分支，避免最后提交时出现比较大的冲突。
在提交你的代码到 develop 分支前，请先确认三件事：
> `该功能是经过测试的。`
> `该功能的单元测试覆盖率不低于 50%。`
> `该功能的代码符合遵循的代码规范。`

#### `release` 分支
release 为预上线分支，QA 会使用 release 分支代码为基准进行测试和发布。
每次 sprint 结束时，project owner 会基于 develop 分支创建用于 QA 测试的 release 分支。正式发布 release 后，需要将 release 的代码推送到 master 分支并创建版本 tag。
release 分支的命名基于`版本号规范`执行，release 分支在测试时可以存在多个版本，正式发布后需要合并到 master 分支。
> 示例：
> 如当前需要在 QA 测试 2.4.0 的代码，则从 develop 创建的 release 分支名称为：release-2.4.0。
> 当 release-2.4.0 正式发布后推送代码到 master，并基于 master 创建 tag v2.4.0。

#### `hotfix` 分支
线上出现紧急问题时，基于对应版本 tag 创建 hotfix 分支进行代码修复。完成修复后需要合并代码到 develop 分支，并创建一个包含`新修正版本号`的 tag 分支。
如其它 tag 发现有同样问题，则需要从目标 tag 建立分支，然后 `cherry-pick` hotfix 的代码到 tag 代码进行修复，并生成一个包含`新修正版本号`的 tag 分支。
> 示例：
> 2.4.0 的线上代码发现有 bug，则基于 tag v2.4.0 创建 hotfix-2.4.0 临时分支进行修复。修复完成 cherry-pick 修复代码到 develop，并创建新的 tag v2.4.1。
> 如果在修复期间已经存在 release-2.5.0，代码同样存在 bug，则 cherry-pick 修复代码的 commit 到 release-2.5.0。

---
### 版本号规范

规范：`主版本号 . 子版本号 . 修正版本号 [ 版本号修饰词 ]`
- 主版本号：第一个数字，产品改动较大，可能无法向后兼容（要看具体项目）。
- 子版本号：第二个数字，增加了新功能，向后兼容。
- 修正版本号：第三个数字，修复 bug 或优化代码，一般没有添加新功能，向后兼容。

例如：
- 1.0
- 2.14.0
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

---
### Commit messages规范

> 为保证团队合作的高效性，Select team 需要遵循可读友好的 commit messages 日志规范。Repo 后面会实现基于 Github 的 webhook 实现对 commit messages 的自动检查，自动reject 不符合 commit 规范的 pull request。

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

基于 `develop -> release -> master/tag`的推送流进行版本发布管理：
- `develop` 分支用于 Dev 进行开发和测试。
- `release` 分支用于 QA/Cert/Prod 及 bugfix。
- `master/tag` 分支用于产品 hotfix。
