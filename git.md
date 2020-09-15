#### **1.创建分支**

**git  branch  [分支名]**

#### **2.切换分支**

**git checkout [分支名]**

#### **3.合并分支**

**第一步: 切换到接受修改的分支上.(被合并,增加新内容).**

##### **第二步: 执行命令 git merge [要和当前分支合并的分支名]**

#### 4.合并冲突

**vi 进入文件 i 编辑 删除冲突标识 修改文件至自己想要的程度 保存并退出.**

**git add [有冲突的文件名]**

**git commit -m "备注" 后不能跟文件名**

#### 5.本地配置推送远程仓库地址

**git remote add origin https://github.com/DyzYpp/huashan.git**

#### 6.推送到远程仓库

**git push [地址别名] [分支名]**

#### 7.关闭ssl安全验证

**git config --global http.sslVerify false**

#### 8.远程库克隆到本地

**git clone https://github.com/DyzYpp/huashan.git**

#### 9.给他人推送权限.

**远程仓库--Settings--Manage access 输入他人的GitHup账号!**

#### 10.更新远程库代码

**git pull = git fetch + git merge**