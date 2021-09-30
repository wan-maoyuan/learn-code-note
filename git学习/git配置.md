## 查看git配置清单
```shell
git config -l
```

## 查看git系统配置
```shell
git config --system --list
```

## 查看git本地的配置
```shell
git config --global --list
```

## git配置用户名和邮箱
```shell
git config --global user.name "wantcsoft"
git config --global user.email wanmaoyuan@outlook.com
```

## git工作流程
1. 创建本地仓库有两种方法：1、初始化全新的仓库。2、克隆远程仓库。
- 初始化全新的仓库
```shell
git init
```

- 克隆远程的仓库
```shell
git clone xxxxxxxxxxx
```

2. 文件4种状态
> 版本控制就是对文件的版本的版本控制，要对文件进行修改、提交等操作，首先要知道文件当前在什么状态，不然可能会提交了现在不想提交的文件，或者提交的文件没有提交上。

- Untracked
> 未跟踪，此文件在文件夹中，但没有加入git库，不参与版本控制。可通过```git add```状态变为```Staged```。

- Unmodify
> 文件已入库，未修改，及版本库中的文件快照内容与文件夹一致，如果被修改，而变为```Modified```，如果使用```git rm```移出版本库，则成为```Untracked```文件。

- Modified
> 文件已修改，仅仅是修改，并没有进行其他的操作，这个文件也有两个去处，通过```git add```可进入暂存```Staged```状态，使用```git checkout```则丢弃修改过，返回```Unmodify```状态，这个```git checkout```，即从库中取出文件，覆盖当前修改。

- Staged
> 暂存状态，执行```git commit```则将修改同步到库中，这时库中的文件和本地文件又变为一致，文件为```Unmodify```状态，执行```git reset HEAD filename```取消暂存，文件状态为```Modified```。

3. 查看文件的状态
- 查看指定文件状态
```shell
git status [filename]
```

- 查看所有文件状态
```shell
git status
```

4. 添加文件到暂存区
- 将制定文件添加到暂存区
```shell
git add [filename]
```

- 将所有文件添加到暂存区
```shell
git add .
```

5. 提交暂存区到本地仓库
```shell
git commit -m "提交信息"
```

6. 忽略文件
> 有时候我们不想把某些文件纳入版本控制中，比如数据库文件，临时文件，设计文件等。需要在根目录下建立“.gitignore”文件，文件有如下规则。

- 忽略文件中的空行或以 # 开始的行会被忽略。
- 可以试用Linux通配符。例如星号（*）代表任意多个字符，问号（？）代表一个字符，方括号（[abc]）代表可选字符范围，大括号（{string1, string2,……}）代表可选的字符串。
- 如果名称的最前面有一个感叹号（！），表示例外规则，将不被忽略
- 如果名称的最前面有一个路径分隔符（/），表示忽略的文件在此目录下，而子目录的文件不忽略。
- 如果名称的最后面有一个路径分隔符（/），表示忽略的是此目录下该名称的子目录，而非文件（默认文件或目录都忽略）。

```
*.txt           #忽略所有.txt 结尾的文件
!lib.txt        #但lib.txt文件除外
/temp           #仅忽略项目根目录下的TODO文件，不包括其他目录temp
build/          #忽略build/目录下的所有文件
doc/*.txt       #会忽略 doc/note.txt ，但不包括 doc/server/arch.txt
```

