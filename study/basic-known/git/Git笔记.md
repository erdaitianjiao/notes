- 配置名字和邮箱

~~~bash
git config --global user.name "Your Name"
git config --global user.email "email@example.com"
~~~

- 绑定远程仓库

~~~bash
git remote add origin git@xxx.git
git remote -v
~~~

- 生成密钥

~~~bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
cat ~/.ssh/id_rsa.pub
ssh -T git@github.com
~~~

