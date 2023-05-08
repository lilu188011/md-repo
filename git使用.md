#### 1.配置git用户

```
	配置: 
	git config --global user.name yourname
	git config --global user.email youremail@example.com
```

#### 2.ssh配置

```
 ssh-keygen -t rsa -C "youremail@example.com" 
```

#### 3.提交git远程库步骤

```
git init (初始化仓库)
	
git add . (添加到本地暂存区)

git commit -m "first commit"(提交到本地分支)

git remote add origin (you git address) 
(将本地仓库与远程仓库关联（远程仓库默认名字为 origin ）)

git pull origin master (拉取远程仓库origin master分支)

git push -u origin master (提交远程仓库origin master分支)

```

#### 4.git其它命令

```
git checkout -- file  丢弃掉文件暂存区修改 恢复到上个版本状态

git rm --cached  filename  删除进入本地仓库文件

git checkout -b dev   创建并检出dev分支
	
git branch    		  查看当前分支

git merge dev 		  合并dev分支到当前分支

git branch -d <name>  删除分支

git log --graph --pretty=oneline --abbrev-commit  查看分支的合并情况

git stash             保存工作现场

git branch -D <name>   丢弃没有合并的分支

git tag <tagname>		用于新建一个标签，默认为HEAD

git tag -a <tagname> -m "blablabla..."  可以指定标签信息；

git tag可以查看所有标签。	

```

#### 5.gitignore

```
target/
!.mvn/wrapper/maven-wrapper.jar
!**/src/main/**
!**/src/test/**

### STS ###
.apt_generated
.classpath
.factorypath
.project
.settings
.springBeans
.sts4-cache
.log

### IntelliJ IDEA ###
.idea
*.iws
*.iml
*.ipr
.mvn
mvnw*

### NetBeans ###
/nbproject/private/
/nbbuild/
/dist/
/nbdist/
/.nb-gradle/
build/

### VS Code ###
.vscode/

### generated files ###
bin/
gen/

### MAC ###
.DS_Store

### Other ###
logs/
log
temp/



更新 重置gitignore
    git rm -r --cached .
    git add .
    git commit -m 'update .gitignore'
    
不起作用的原因是这个文件里的规则对已经追踪的文件是没有效果的.所以我们需要使用rm命令清除一下相关的缓存内容.这样文件将以未追踪的形式出现.然后再重新添加提交一下,.gitignore文件里的规则就可以起作用了.    
```

