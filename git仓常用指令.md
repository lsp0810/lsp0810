仓库初始化:

```git
git init
```

添加文件到暂存区：

```git
git add ./src
```

查看仓库状态（暂存区状态）：

```git
git status
```

提交文件：

```git
git commit -m "message"
```

将更新同步到github上：

```
git push origin master
```

查看本地仓关联的远程仓信息：

```git
git remote -v
```

移除本地仓与远程仓的关联关系：

```git
git remote rm origin
```

将github代码保存到本地：

```git
git pull origin
```

### git能将本地代码同步到github的前置工作：

1）本地生成id_rsa以及id_rsa.pub文件

```git
ssh-keygen -t rsa -C "emailaddress"
//一路回车
```

2）生成新的ssh-key

具体流程：账户头像--->settings--->ssh and gpg keys--->new ssh keys

title自选，内容复制id_rsa.pub中的内容即可。

3）将本地仓库与github的一个仓库相关联

```git
git remote add origin 仓库地址
```

此后便可将本地仓同步到远程仓了