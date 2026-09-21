---
name: git-commit-emoji
description: 当编写 Git 提交信息并希望用 emoji 前缀标识提交类型、在 PR 标题中凸显类别时使用。
---

git commit emoji 使用指南
============================

#### 目录

<!-- vim-markdown-toc GFM -->

* [commit 格式](#commit-格式)
* [emoji 指南](#emoji-指南)
* [说明](#说明)
* [参考](#参考)
  * [git commit emoji](#git-commit-emoji)
  * [write a good commit message](#write-a-good-commit-message)

<!-- vim-markdown-toc -->

执行 `git commit` 时使用 emoji 为本次提交打上一个 "标签", 使得此次 commit 的主要工作得以凸现，也能够使得其在整个提交历史中易于区分与查找。

**注意：直接使用 emoji 字符本身（🎉、🐛 等），不使用 `:tada:` 这类 emoji 短代码格式；每次 commit 只使用一个 emoji，选择最贴合本次提交主要工作类型的那一个。**

### commit 格式

`git commit` 时，提交信息遵循以下格式：

```sh
🎉 不超过 50 个字的摘要，首字母大写，使用祈使语气，句末不要加句号

提交信息主体

引用相关 issue 或 PR 编号 <#110>
```

初次提交示例：

```sh
git commit -m "🎉 Initialize Repo"
```

### emoji 指南

emoji                                   | commit 说明
:--------                               | :--------
🎉 (庆祝)                                | 初次提交
🆕 (全新)                                | 引入新功能
🔖 (书签)                                | 发行/版本标签
🐛 (bug)                                 | 修复 bug
🚑 (急救车)                              | 重要补丁
🌐 (地球)                                | 国际化与本地化
💄 (口红)                                | 更新 UI 和样式文件
🎬 (场记板)                              | 更新演示/示例
🚨 (警车灯)                              | 移除 linter 警告
🔧 (扳手)                                | 修改配置文件
➕ (加号)                                | 增加一个依赖
➖ (减号)                                | 减少一个依赖
⬆️ (上升箭头)                            | 升级依赖
⬇️ (下降箭头)                            | 降级依赖
⚡ (闪电)<br>🐎 (赛马)                    | 提升性能
📈 (上升趋势图)                          | 添加分析或跟踪代码
🚀 (火箭)                                | 部署功能
✅ (白色复选框)                          | 增加测试
📝 (备忘录)<br>📖 (书)                    | 撰写文档
🔨 (锤子)                                | 重大重构
🎨 (调色板)                              | 改进代码结构/代码格式
🔥 (火焰)                                | 移除代码或文件
✏️ (铅笔)                                | 修复 typo
🚧 (施工)                                | 工作进行中
🗑️ (垃圾桶)                              | 废弃或删除
♿ (轮椅)                                | 可访问性
👷 (工人)                                | 添加 CI 构建系统
💚 (绿心)                                | 修复 CI 构建问题
🔒 (锁)                                  | 修复安全问题
🐳 (鲸鱼)                                | Docker 相关工作
🍎 (苹果)                                | 修复 macOS 下的问题
🐧 (企鹅)                                | 修复 Linux 下的问题
🏁 (旗帜)                                | 修复 Windows 下的问题
🔀 (交叉箭头)                            | 分支合并



### 说明

本指南直接使用 emoji 字符本身（如 🎉、🐛），而不是 `:tada:` 这类 emoji 短代码格式。直接使用 emoji 字符无需依赖 emojify 等工具转换，复制粘贴即可生效。

### 参考

#### git commit emoji

- [gitmoji](https://github.com/carloscuesta/gitmoji/)
- [emoji-cheat-sheet](http://www.webpagefx.com/tools/emoji-cheat-sheet/)
- [styleguide-git-commit-message](https://github.com/slashsBin/styleguide-git-commit-message)
- [atom git commit messages guide](https://github.com/atom/atom/blob/master/CONTRIBUTING.md#git-commit-messages)
- [An emoji guide for your commit messages](https://gitmoji.carloscuesta.me/)
- [程序员提交代码的 emoji 指南——原来表情文字不能乱用](https://www.h5jun.com/post/gitmoji.html)
- [Ant Design 更新日志 emoji 规范](https://github.com/ant-design/ant-design/wiki/%E8%BD%AE%E5%80%BC%E8%A7%84%E5%88%99%E5%92%8C%E7%89%88%E6%9C%AC%E5%8F%91%E5%B8%83%E6%B5%81%E7%A8%8B#emoji-for-changelog)

#### write a good commit message

- [A Note About Git Commit Messages](http://tbaggery.com/2008/04/19/a-note-about-git-commit-messages.html)
- [How to write a Git Commit Message (2014)](https://news.ycombinator.com/item?id=13889155)
- [how to write a good git commit message](https://github.com/joelparkerhenderson/git_commit_message)
- [5 Useful Tips For A Better Commit Message](https://robots.thoughtbot.com/5-useful-tips-for-a-better-commit-message)
- [Udacity Git Commit Message Style Guide](http://udacity.github.io/git-styleguide/)
- [How to commit a change with both “message” and “description” from the command line?](https://stackoverflow.com/questions/16122234/how-to-commit-a-change-with-both-message-and-description-from-the-command-li)
