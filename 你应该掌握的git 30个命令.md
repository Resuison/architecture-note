### 引言

Git 是世界上最流行的分布式版本控制系统。Linux 内核的创建者 Linus Torvalds 早在 2005 年就构建了这个工具，该工具目前是一个积极维护的开源项目。大量开源和商业项目依赖 Git 进行版本控制。

在本文中，我将列出作为开发人员应该知道的最基本的命令，并成为处理 GitHub 存储库的大师。初学者和经验丰富的开发人员可以从本文中受益。

**1.设置您的用户名和电子邮件**

需要用户名才能将提交与您的姓名联系起来。它与您用于登录 GitHub 个人资料的 GitHub 帐户用户名不同。您可以使用该`git config`命令设置或更新您的用户名。新名称将自动反映在您从命令行推送到 GitHub 的任何未来提交中。如果你想隐藏你的真实姓名，你可以使用任意文本作为你的 Git 用户名。

```bash
git config --global user.name "Tara Routray"
```

您还可以使用该`git config`命令更新要与 Git 提交关联的电子邮件地址。新的电子邮件地址将自动反映在您从命令行推送到 GitHub 的任何未来提交中。

```bash
git config --global user.email "dev @t araroutray.com"
```

**2.缓存您的登录凭据**

您可以使用`config`参数和`--global`标志来缓存您的登录凭据。它有助于避免每次执行提交时重新输入用户名和密码。

```bash
git config --global credential.helper cache
```

**3.初始化一个仓库**

`init`您可以使用该参数创建一个空的 Git 存储库或重新初始化现有的存储库。它将在初始化时创建一个隐藏目录。该目录包含 Git 使用和创建的所有对象和引用，作为项目历史的一部分。

```bash
git init
```

**4.将单个文件或所有文件添加到暂存区**

`add`您可以使用参数和文件名将单个文件添加到暂存区。只需`somefile.js`用您的文件名替换。

```bash
git add somefile.js
```

您还可以通过提供通配符`.`而不是文件名来将所有文件和目录添加到暂存区。

```bash
git add .
```

**5.检查存储库状态**

```bash
git status
```

**6. 使用单行消息或通过编辑器提交更改**

`commit`您可以使用参数和`-m`标志在存储库中进行提交时添加单行消息。此消息应立即跟随标志并用引号引起来。

```bash
git commit -m "Your short summary about the commit"
```

您还可以在终端中打开文本编辑器来编写完整的提交消息。此消息可以包含多行文本，以便您详细说明对存储库所做的更改。

```bash
git commit
```

**7.查看提交历史和变化**

`log`您可以使用该参数查看在存储库中所做的更改。它将按顺序显示最新提交的列表。您还可以通过添加`-p`标志来检查每个文件的详细更改。

```bash
git log -p
```

**8. 查看特定的提交**

`show`您可以通过使用参数并提供提交的 ID 或哈希来查看特定提交的详细更改。对于您的存储库中所做的每个提交，此哈希都是唯一的。

```bash
git show 1af17e73721dbe0c40011b82ed4bb1a7dbe3ce29
```

您还可以提供一个简短的哈希来获得相同的结果。

```bash
git show 1af17e
```

**9. 提交前查看更改**

`diff`您可以使用该参数查看对存储库所做的更改列表。默认情况下，它只会显示未暂存的更改。

```bash
git diff
```

如果要查看分阶段的更改，可以添加`--staged`标志。

```bash
git diff --staged
```

您还可以提供文件名作为参数以仅查看特定文件的更改。

```bash
git diff somefile.js
```

**10.从当前工作树中删除跟踪文件**

您可以使用该`rm`参数从当前工作树中删除文件。此操作还将从索引中删除文件。

```bash
git rm dirname/somefile.js
```

您还可以提供文件 glob（例如 *.js）来删除所有匹配的文件。

```bash
git rm dirname/*.html
```

**11.重命名文件**

您可以使用该`mv`参数重命名文件或目录。此参数需要 a`<source>`和 a `<destination>`。源必须存在并且可以是文件或目录，而目标必须是现有目录。

```bash
git mv dir1/somefile.js dir2
```

执行上述命令会将源移动到目标目录。索引将得到更新，但您必须提交更改。

**12. 恢复未分阶段和分阶段的更改**

您可以使用该`checkout`参数恢复未暂存的工作树文件。您需要提供一个文件路径来更新它。如果未设置文件路径，`git checkout`则将更新 HEAD 以将指定分支设置为当前分支。

```bash
git checkout somefile.js
```

要恢复暂存的工作树文件，您可以使用该`reset`参数。您需要提供文件路径才能将其从暂存区域中删除。这不会删除对文件所做的任何更改或修改，相反，该文件将被视为未暂存文件。

```bash
git reset HEAD somefile.js
```

如果要取消暂存所有暂存文件，则不要提供文件路径。

```bash
git reset HEAD
```

**13. 修改最近的提交**

您可以使用带有标志的`commit`参数来更改最近的提交。`--amend`例如，您刚刚提交了一些文件，并且您记得您在提交消息中犯了一个错误。在这种情况下，您可以执行此命令来编辑先前提交的消息，而无需修改其快照。

```bash
git commit --amend -m "上一次提交的更新信息"
```

也可以对先前提交的文件进行更改。例如，您已经更新了多个文件夹中的一些文件，您想在一个快照中提交这些文件，但是您忘记添加一个文件夹来提交。修复此类错误只需暂存其他文件或文件夹，并使用`--amend`and`--no-edit`标志进行提交。

```bash
git add dir1 
git commit
# 这里忘记添加dir2提交，可以执行以下命令修改其他文件和文件夹。
git add dir2 
git commit --amend --no-edit
```

该`--no-edit`标志将允许您在不更改其提交消息的情况下对您的提交进行更正。最后的提交将替换不完整的提交，看起来我们已经在一个快照中提交了对所有更新文件和文件夹的更改。

> **警报 ！！！不要修改公共提交。
> 
> **使用 amend 更正本地提交非常好，您可以将其推送到共享存储库。但是你应该避免修改已经公开的提交。请记住，修改后的提交是全新的提交，之前的提交在您当前的分支上将不可用。它与重置公共快照具有相同的后果。

**14. 回滚上次提交**

`revert`您可以使用该参数回滚上次提交。这将创建一个新的提交，与之前的提交相反，并将其添加到当前的分支历史记录中。

```bash
git revert HEAD
```

**恢复与重置**

该`git revert`命令仅撤消单个提交。它不会通过删除所有后续提交来回到项目的先前状态，这是在`git reset`使用时完成的。

与重置相比，恢复有两个主要好处。首先，它不会改变项目历史，这使它成为提交的安全操作。其次，它能够在历史的任何时候定位特定的提交，而 git reset 只能从当前提交向后工作。例如，如果您想使用 git reset 撤消旧提交，则必须删除在目标提交之后发生的所有提交，然后重新提交所有后续提交。因此，`git revert`撤消更改是一种更好、更安全的方法。

**15.回滚特定的提交**

`revert`您可以使用参数和提交 ID回滚到特定提交。它将创建一个新的提交，即提供的提交 ID 的副本，并将其添加到当前分支历史记录中。

```bash
git revert 1af17e
```

**16. 创建并切换到新分支**

您可以使用`branch`参数和分支名称来创建新分支。

```bash
git branch new_branch_name
```

但是 Git 不会自动切换到它。如果你想让 Git 自动切换，你必须传递`-b`标志和`checkout`参数。

```bash
git checkout -b new_branch_name
```

**17.列出所有分支**

您可以使用该`branch`参数查看所有分支的列表。它将显示所有分支并用星号 (*) 标记当前分支并突出显示它。

```bash
git branch
```

您还可以使用该`-a`标志列出所有远程分支。

```
git branch -a
```

**18. 删除一个分支**

您可以使用`branch`参数、`-d`标志和分支名称来删除分支。如果您已经完成了一个分支的工作并将其合并到主分支中，您可以删除该分支而不会丢失任何历史记录。但是，如果分支尚未合并，则删除命令将输出错误消息。这可以保护您不会失去对文件的访问权限。

```bash
git branch -d existing_branch_name
```

如果要强制删除分支，可以使用大写`-D`标志。这将删除分支，尽管其当前状态且没有任何警告。

```bash
git branch -D existing_branch_name
```

上述命令只会删除分支的本地副本。分支可能存在于远程存储库中。如果要删除远程分支，请执行以下命令。

```bash
git push origin --delete existing_branch_name
```

**19.合并两个分支**

您可以使用`merge`参数和分支名称来合并两个分支。这会将指定的分支合并到主分支中。

```bash
git merge existing_branch_name
```

如果需要合并提交，可以使用`--no-ff`标志执行 git merge。

```bash
git merge --no-ff existing_branch_name
```

上面的命令会将指定的分支合并到主分支中，并生成一个合并提交。这对于记录存储库中发生的所有合并至关重要。

**20. 将提交日志显示为当前或所有分支的图表**

`log`您可以使用参数和`--graph --oneline --decorate`标志将提交日志作为当前分支的图表查看。该`--graph`选项将绘制一个 ASCII 图形，它表示提交历史的分支结构。当它与`--oneline`和`--decorate`标志一起使用时，它可以更容易地识别哪个提交属于哪个分支。

```bash
git log --graph --oneline --decorate
```

如果要查看所有分支的提交日志，可以使用该`--all`标志。

```bash
git log --all --graph --oneline --decorate
```

**21. 中止冲突的合并**

您可以使用`merge`参数和`--abort`标志中止冲突的合并。它允许您退出合并过程并返回到合并开始之后的状态。

```bash
git merge --abort
```

您还可以`reset`在合并冲突期间使用参数 to 将冲突文件重置为稳定状态。

```bash
git reset
```

**22.添加远程存储库**

您可以使用远程存储库的`remote add`参数`<shortname>`和远程存储库的 来添加远程存储`<url>`库。

```bash
git remote add awesomeapp https://github.com/someurl..
```

**23.查看远程 URL**

您可以使用`remote`参数和`-v`标志查看远程 URL。这将列出您与其他存储库的远程连接。

```bash
git remote -v
```

上面的命令是管理存储在存储库`.git/config`文件中的远程条目列表的接口。

**24.获取有关远程存储库的附加信息**

您可以通过使用`remote show`参数和远程名称来获取有关远程存储库的详细信息，例如`origin`.

```bash
git remote show origin
```

上面的命令将输出与远程关联的分支列表，以及连接到获取和推送文件的端点。

**25.将更改推送到远程存储库**

您可以使用`push`参数、存储库名称和分支名称将更改推送到远程存储库。

```bash
git push origin main
```

上述命令将帮助您将本地更改上传到中央存储库，以便其他团队成员可以查看您所做的更改。

**26.从远程存储库中提取更改**

您可以使用该`pull`参数从远程存储库中提取更改。这将获取当前分支的指定远程副本并立即将其合并到本地副本中。

```bash
git pull
```

参数`--verbose`您还可以使用该标志查看已下载文件的详细信息。

```bash
git pull --verbose
```

**27. 将远程存储库与本地存储库合并**

您可以使用`merge`参数和远程名称将远程存储库与本地存储库合并。

```bash
git merge origin
```

**28. 将新分支推送到远程存储库**

您可以使用`push`参数、`-u`标志、远程名称和分支名称将新分支推送到远程存储库。

```bash
git push -u origin new_branch
```

**29.删除远程分支**

您可以使用`push`参数、`--delete`标志、远程名称和分支名称来删除远程分支。

```bash
git push --delete origin existing_branch
```

**30. 使用变基**

您可以通过使用`rebase`参数和分支名称来使用此功能。变基是将一系列提交组合或移动到新的基本提交的过程。

```bash
git rebase branch_name
```

上面的命令会将您的分支的基础从一个提交更改为另一个，这将使您看起来好像您是从另一个提交创建的分支。Git 通过创建新的提交并将它们应用到指定的基础来实现这一点。非常有必要了解，即使分支看起来相同，它也是由全新的提交组成的。
