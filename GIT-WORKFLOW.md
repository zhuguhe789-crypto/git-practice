# Git 与 GitHub 标准工作流手册

从创建仓库到 Pull Request 与开源协作的完整操作说明

这份手册把日常会用到的 Git 操作整理成三个场景：从零创建自己的仓库、分支开发并提交 Pull Request、使用别人的仓库。每一步都给出命令、这条命令在做什么，以及执行后应该看到什么，可以照着做，也可以当成速查表随时翻。

读完并动手走一遍之后，你应该能独立完成一次从建仓到合并的完整流程，并且知道每一步失败时该看哪里。

| 项目 | 内容 |
| --- | --- |
| 适用对象 | 刚开始用 Git 管理代码、准备把项目放到 GitHub 上的学生与研究者 |
| 环境 | Windows 与 PowerShell，Git 2.4 及以上，GitHub 网页版 |
| 更新日期 | 2026 年 10 月 28 日 |

## 一 先理解模型

### 三个区域

Git 把一次改动拆成四层，每层只负责一件事，绝大多数命令都是在这些层之间搬运内容。

| 区域 | 里面放什么 | 相关命令 |
| --- | --- | --- |
| 工作区 | 你正在编辑的文件 | 直接编辑 |
| 暂存区 | 准备写进下一次提交的快照 | git add |
| 本地仓库 | 已经提交的历史 | git commit |
| 远端仓库 | GitHub 上的副本 | git push 与 git pull |

理解这张表，就能理解为什么改完文件还要 add 再 commit：add 决定这次提交包含哪些改动，commit 决定这些改动以什么说明进入历史。

### 一次完整改动的路径

```text
编辑文件 -> git add -> git commit -> git push -> Pull Request -> merge
```

对应的层次是：编辑发生在工作区，add 把改动放进暂存区，commit 写进本地历史，push 送到远端分支，Pull Request 负责讨论与审查，merge 之后改动才真正进入 main。

### 分支只是一个指针

分支不是文件夹的副本，它只是一个指向某个提交的指针。创建分支几乎不花时间和空间，所以可以放心地为每个小改动开一条分支。HEAD 表示你现在站在哪个分支上。

## 二 Pull Request 的作用

### 它到底是什么

Pull Request 简称 PR，是一次请求合并：请求把你这条分支上的改动合并进 main，同时提供一个页面用于讨论、审查和自动检查。合并之前，main 不会有任何变化。

它不是一个发布动作，也不是代码备份。它的本质是一个带 diff 的讨论区。

### 五个实际作用

- 审查：改动以逐行 diff 展示，可以在具体某一行留评论，而不是等合并完才发现问题。
- 留痕：为什么这么改、试过什么方案、有什么副作用，都写在 PR 里，和代码绑在一起。
- 门禁：接入 CI 之后，测试不过或格式错误会让 PR 标红，可以直接禁止合并，main 因此始终可用。
- 权限：团队里 main 通常设为受保护分支，任何人都不能直接推送，必须走 PR 并获得批准。
- 回溯：每个 PR 有编号，出问题时能顺着编号找到当时的讨论、提交和回滚点。

### 与直接推送的区别

| 对比项 | 直接推送到 main | 分支加 PR |
| --- | --- | --- |
| main 的状态 | 随时可能被半成品污染 | 只有通过检查的改动才进入 |
| 改动说明 | 只有一行提交信息 | 描述、讨论、审查记录齐全 |
| 出错处理 | 需要 revert 提交 | 关闭 PR 即可，main 从未受影响 |
| 协作方式 | 容易互相覆盖 | 分支隔离，冲突在合并前解决 |

### 三种合并方式

```text
Create a merge commit      Squash and merge        Rebase and merge
main   A---B-------M       main   A---B---S        main   A---B---C'---D'
            \     /               \  /                     \
feature      C---D                C-D                      C---D
```

- Create a merge commit：保留分支上每个提交，并额外生成一个合并提交。适合提交本身有保留价值的情况。
- Squash and merge：把分支上的所有提交压成一个新提交。个人项目和碎提交较多的分支首选，main 的历史最干净。
- Rebase and merge：把提交逐个搬到 main 顶端，不生成合并提交，历史是一条直线。适合团队要求线性历史时。

### 一个人开发为什么也要用

因为它强迫你做两件事：合并前在 Files changed 里把改动完整读一遍，以及把为什么改写下来。这两件事能挡掉大量低级错误，三个月后的你会感谢现在的自己。

## 三 场景一 从零创建自己的仓库

### 第一步 在 GitHub 上建空仓库

打开 github.com/new，填写仓库名，Owner 选择你的账号。三个可选项，也就是 Add a README、Add .gitignore 和 Choose a license，都不要勾选。

勾选任意一项，远端就会先有一个提交。你本地再提交就会形成两条互不相关的历史，第一次 push 会被拒绝，还需要额外处理。空仓库才能一次推干净。

创建成功后会出现 Quick setup 页面，列出 HTTPS 与 SSH 两种地址，说明仓库是空的。

### 第二步 本地初始化

```powershell
mkdir E:\github\my-project
cd E:\github\my-project
git init
```

git init 只做一件事：在当前目录创建 .git 文件夹。它不联网，也不产生任何提交。执行后应该看到 Initialized empty Git repository。

### 第三步 写 README 与 .gitignore

```powershell
code README.md .gitignore
```

README 是别人打开仓库看到的第一屏，.gitignore 决定哪些文件永远不进仓库。这两个文件先写，因为它们定义了仓库的边界。

README 至少包含五节：一句话定位、快速开始、项目结构、方法说明、结果与参考。

### 第四步 暂存

```powershell
git status
git add README.md .gitignore
```

git add 把工作区的改动放进暂存区。status 里绿色的 A 表示新增，M 表示修改，都已经进入暂存区。

### 第五步 提交

```powershell
git commit -m "chore: 初始化仓库，添加 README 与 .gitignore"
```

提交会把暂存区打包成一个不可变的快照，带上作者、时间和说明。执行后会打印提交哈希和前几个文件的统计。

### 第六步 关联远端

```powershell
git remote add origin git@github.com:用户名/仓库名.git
git remote -v
```

origin 只是这个地址的别名，不是关键字。remote -v 用来确认地址里的用户名和仓库名没有写错，写错会导致推送失败。

### 第七步 首次推送

```powershell
git push -u origin main
```

-u 把本地 main 和远端 main 绑定，之后直接 git push 即可。成功时会看到 new branch 和 set up to track 两行提示。

### 第八步 验证

刷新仓库页面，能看到刚才的文件；执行 git log --oneline，能看到提交；git status 显示 working tree clean。到这里，建仓流程结束。

## 四 场景二 分支开发与 Pull Request

### 开分支

```powershell
git switch main
git pull --ff-only
git switch -c feature/readme-content
```

第一条切回 main，第二条只做快进式更新，有分叉会报错，这是保护，第三条创建并切换到新分支。老教程里的 git checkout -b 效果相同。

命名建议：feature 表示新功能，fix 表示修缺陷，docs 表示文档，refactor 表示重构，test 表示测试，后面接两三个英文词。

### 修改并提交

```powershell
git status
git diff
git add -p
git commit -m "docs(readme): 填写项目简介与快速开始"
```

git diff 看的是工作区相对暂存区的差异，提交前一定要过一遍。git add -p 会逐块询问是否暂存，适合把混在一起的改动拆成多个提交。

提交信息写成类型加范围加描述，首行 50 字以内，用祈使句，一次提交只做一件事。

### 推送

```powershell
git push -u origin feature/readme-content
```

第一次推送这条分支要带 -u，之后直接 git push。成功时终端会打印一条带链接的提示，指向 PR 创建页。

### 在网页上发起 PR

点推送输出里的链接，或者回仓库主页点黄色横幅的 Compare and pull request。

PR 有方向。页面顶部显示 base: main 与 compare: feature/readme-content，意思是把 feature 的改动合进 main。方向反了就是反向合并。

### PR 页面的四个标签

| 标签 | 用途 |
| --- | --- |
| Conversation | 描述、评论、审查结论和合并按钮 |
| Commits | 这条分支上的所有提交，能看出碎提交有没有整理 |
| Checks | 自动化检查结果，没接 CI 时为空 |
| Files changed | 逐行 diff，真正审查的地方，行号左侧可以留评论 |

### 描述写什么

```markdown
## 做了什么
## 为什么这么做
## 如何验证
## 遗留问题
```

这四段是给未来的自己看的。合并之后，除了一行提交信息，就只剩下这段描述能解释当时的决定。

### 收到意见之后

不要重开 PR。继续在同一条分支上修改、提交、推送，PR 会自动带上新提交，页面上会显示新增了几个提交。这是 PR 最省事的地方。

### 合并与清理

在 Conversation 里点 Merge pull request 旁的下拉箭头，选择 Squash and merge，确认后再点 Delete branch 删除远端分支。

```powershell
git switch main
git pull --ff-only
git branch -D feature/readme-content
git fetch --prune
```

squash 之后原提交不在 main 的历史里，git branch -d 会提示 not fully merged，这是正常现象，用大写 -D 强制删除。fetch --prune 清理已经不存在的远端分支引用。

## 五 场景三 使用别人的仓库

### 只想读代码或跑起来

```powershell
git clone https://github.com/作者/仓库.git
cd 仓库
```

clone 会拉取完整历史。你可以在这里修改和提交，但 git push 一定失败，因为你对别人的仓库没有写权限。这是权限设计，不是配置问题。

### 想改成自己的项目

```powershell
git clone 别人的仓库地址
cd 仓库
git remote remove origin
git remote add origin git@github.com:你的用户名/你的新仓库.git
git push -u origin main
```

关键是换掉 origin，否则你会一直往原作者那里推，而且以后 pull 也会拉回别人的更新。动手前先看 LICENSE：没有许可证的仓库默认保留所有权利，不适合直接当自己的作品发布。

### 想给原作者贡献代码

这是 fork 流程，也是 GitHub 协作的标准姿势。

1. Fork。在对方仓库页面点右上角 Fork，GitHub 会在你名下复制一份，你有完整的写权限。
2. clone 自己的 fork，注意是自己的用户名。

```powershell
git clone git@github.com:你的用户名/对方仓库名.git
cd 对方仓库名
```

3. 添加 upstream，用来跟踪原仓库的更新。

```powershell
git remote add upstream https://github.com/原作者/仓库.git
git remote -v
```

origin 指向你的 fork，upstream 指向原仓库。

4. 从最新代码开分支。

```powershell
git fetch upstream
git switch -c fix/some-bug upstream/main
```

5. 修改、提交、推送到你的 fork。

```powershell
git add -p
git commit -m "fix(module): 修复某处空指针"
git push -u origin fix/some-bug
```

6. 向上游发 PR。打开原仓库页面，点击 Compare and pull request；或者在 PR 页面选择 compare across forks，base 选原仓库的 main，head 选你 fork 的分支。

7. 跟进审查。维护者提意见就在同一分支继续提交推送；如果被要求整理提交，用交互式 rebase 压缩后，用 --force-with-lease 推送。

8. 保持 fork 同步。网页上 fork 主页有 Sync fork 按钮，最省事；命令行版本如下。

```powershell
git fetch upstream
git switch main
git merge --ff-only upstream/main
git push origin main
```

### 开源礼仪

先读 CONTRIBUTING.md 和 issue 列表，确认没人已经在做；一个 PR 只解决一件事；不要顺手格式化整个文件，否则 diff 会炸；提交信息跟随项目既有风格。

## 六 分支命名与提交信息

### 提交类型

| 类型 | 含义 | 例子 |
| --- | --- | --- |
| feat | 新功能 | feat(data): 添加缺失值填充策略 |
| fix | 修复缺陷 | fix(train): 修复早停判断错误 |
| docs | 文档 | docs(readme): 补充快速开始 |
| refactor | 重构 | refactor(model): 拆分训练与评估 |
| test | 测试 | test(utils): 补充边界用例 |
| chore | 杂务 | chore: 更新依赖版本 |

### 好与坏的提交信息

| 写法 | 问题 |
| --- | --- |
| update | 更新了什么，完全看不出来 |
| 修改 | 同上，而且没有范围 |
| fix bug | 哪个 bug，为什么会出现 |
| docs(readme): 补充环境依赖与运行命令 | 具体、可检索、能当变更日志 |

## 七 提交前自检

- git status 里没有 .env、数据集、模型权重、缓存目录这类不该进仓库的文件。
- git diff 完整读过，没有残留的调试打印和注释掉的死代码。
- 提交信息说明的是为什么改，而不是只写改了啥。
- 没有密钥、密码和 token。
- 代码至少能跑通，或者至少能正常导入。

## 八 常用命令速查

| 命令 | 作用 | 使用时机 |
| --- | --- | --- |
| git clone 地址 | 完整复制远端仓库 | 接手项目、使用别人的代码 |
| git switch -c 分支 | 新建并切换分支 | 开始一个新改动 |
| git status | 看工作区与暂存区状态 | 提交前必看 |
| git diff | 看未暂存的改动 | 提交前必看 |
| git add -p | 分块暂存 | 改动较多、需要拆分提交 |
| git commit -m 信息 | 生成提交快照 | 完成一个可描述的小改动 |
| git push -u origin 分支 | 推送并绑定远端 | 分支第一次推送 |
| git pull --ff-only | 只做快进式更新 | main 上同步远端 |
| git rebase main | 把分支挪到 main 最新处 | 分支落后、合并前整理 |
| git fetch --prune | 清理已删除的远端引用 | 删除分支之后 |
| git reflog | 查看 HEAD 的移动记录 | 找回丢失的提交 |
| git branch -D 分支 | 强制删除本地分支 | squash 合并后清理 |

## 九 故障处理

| 现象 | 原因 | 处理 |
| --- | --- | --- |
| not a git repository | 当前目录不在仓库内 | cd 进仓库，或用 git -C 路径 |
| Permission denied publickey | SSH 公钥没加到账号 | 检查 github.com/settings/keys |
| Repository not found | 远端地址或仓库名错误 | 用 git remote -v 核对 |
| non-fast-forward | 远端有你本地没有的提交 | 先 git pull --rebase 再推送 |
| not fully merged | squash 合并后原提交不在主线 | 确认无误后用 git branch -D |
| 想撤销未暂存的修改 | 改动还没进暂存区 | git restore 文件 |
| 想取消暂存 | 已 add 但还没 commit | git restore --staged 文件 |
| rebase 卡住 | 有冲突尚未解决 | 改文件、add、rebase --continue |
| rebase 彻底乱了 | 想回到起点 | git rebase --abort |
| 误提交了大文件 | 已进入历史 | git rm --cached 文件，补进 .gitignore |

## 十 附录 本仓库历史复盘

这是 git log --oneline --graph --all 的真实输出。

```text
* e645902 (HEAD -> main, origin/main) docs(readme): 填写项目简介与快速开始 (#1)
| * f6e6b6d (feature/readme-content) docs(readme): 填写项目简介与快速开始
|/
* 4a8af66 chore: 初始化仓库，添加 README 与 .gitignore
```

逐行读：e645902 是 squash 合并后在 main 上生成的新提交，括号里的 #1 表示它来自第 1 号 PR；f6e6b6d 是分支上原来的提交，squash 只复制改动内容，不复制历史，所以它没有进入 main；4a8af66 是两条线的共同起点。

这也解释了为什么本地还留着 feature/readme-content 分支。远端分支已经在网页上删除，本地分支却还指向那个旧提交。执行 git branch -D 和 git fetch --prune 就能清理干净。
