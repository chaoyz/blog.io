1. 检查git是否安装git --version | grep version | wc -l

2. 删除本机上所有vim安装包sudo yum remove -y vim*

3. 安装终端字符集处理库sudo yum install -y ncurses-devel

4. 编译安装vim
 - cd /usr/local/src/
 - git clone [https://github.com/vim/vim.git](https://github.com/vim/vim.git)
 - cd vim 

 - ./configure --prefix=/usr/local/vim

 - make && make install

5. 添加vim系统变量 

 - echo "export PATH=$PATH:/usr/local/vim8/bin" >> /etc/profile

 - source /etc/profile