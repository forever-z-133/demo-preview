# Git 操作规范

## 新建分支

通常新功能的分支都是基于 uat-sync 为副本新建；  
如果部分功能已上线却还未加入 uat-sync 中，则自行基于某分支或某次提交为副本新建。  
或请组长权衡何时更新一版 uat-sync，使 uat-sync 与 master 同步。

## 分支命名

如果团队使用流程管理工具，通常会有任务编号，如若无则以日记 20190318 为任务编号。  
新分支需命名为 `feature/#{id}-{name}-{description}`，  
其中 id 为任务编号，name 为开发者姓名，description 通常为任务标题或者自定义描述。

假如需求列表比较复杂，可再引入 devlop 分支，  
比如 feature 代表某次较大版本，devlop 是基于本版本的拆分，  
后续 devlop 的新建与合并也将是对焦对应 feature 而非 uat-sync。  
推荐 devlop 命名为 `devlop/#{id}-{name}-{description}`。

若为私人分支，推荐命名为 `custom/##-{name}-{description}`。  
若为紧急部署分支，如临时修复的 BUG，推荐命名为 `hotfixes-{20190101}`。

_当然，任务区分版本只是代码版本管理的方案之一，其他方案的规范不见得适用本节_

## 提交修改

`git commit -m {desc}` 提交前请务必自行 review 检查一次。  
其中 desc 描述推荐为 `#{id} {description}`。

## 请求合并

如果你使用着 GitLab 或 Github 作为仓库，  
在网页端则会有 “Create Merge Request” 或 “New pull request” 按钮，  
填写表单后确认，将链接复制给小组长进行 code review 后，才能由组长合并。

通常测试分支为 uat，如果有多个测试环境则 uat-1 等。  
部分公司在 uat 测试通过后还有 pre 测试，则由测试人员将修改的 feature 分支合并到 pre 分支。

如果使用上述机制，大多数情况下不允许直接合并到测试分支，统统发起合并请求。

## 合并分支

合并前不要盲目自信，比如没有删除测试代码、遗漏功能未完成等，  
在 code review 中需检查代码规范（如命名/公共方法/容错和性能考虑等）。

若在请求合并机制下，合并人员在 PC 上点击 `Accept Merge Request` 按钮即可。

## 处理冲突

若合并时存在冲突，需合并人员在本地切换至 uat 分支，  
手动本地将 feature 分支合并至 uat 分支，解决完冲突后进行 push 即可。 <br>

若合并部署后才发现有代码丢失等情况，  
小组长可在本地修改 uat 分支代码直接进行 push，并务必写清提交描述。 <br>

此过程中请特别注意要 git pull 保持本地 uat 代码与线上 uat 的同步才能 push。
