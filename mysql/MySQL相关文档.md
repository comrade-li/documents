# MySQL相关文档

## 一、MySQL安装(以8.x为例)

1. 添加mysql用户组和用户

    ```shell
    sudo groupadd mysql
    sudo useradd -r -g mysql -s /bin/false mysql
    ```

2. 解压并链接到/usr/local下,并添加相关目录并授权

    ```shell
    sudo tar -xvf mysql-8.4.1-linux-glibc2.28-x86_64.tar.xz -C /opt
    sudo ln -s /opt/mysql-8.4.1-linux-glibc2.28-x86_64/ /usr/local/mysql

    sudo mkdir /usr/local/mysql/mysql-files /usr/local/mysql/data 
    sudo chown mysql:mysql /usr/local/mysql/mysql-files /usr/local/mysql/data
    sudo chmod 750 /usr/local/mysql/mysql-files /usr/local/mysql/data
    ```

3. 添加配置文件

    ```shell
    echo '[mysqld]
    datadir=/usr/local/mysql/data
    socket=/tmp/mysql.sock
    port=3306
    log-error=/usr/local/mysql/data/localhost.localdomain.err
    user=mysql
    secure_file_priv=/usr/local/mysql/mysql-files
    local_infile=OFF' | sudo tee /etc/my.cnf
    ```

4. 初始化数据库

    ```shell
    cd /usr/local/mysql
    sudo ./bin/mysqld --defaults-file=/etc/my.cnf --initialize-insecure
    ```

5. 配置systemd和mysql客户端并连接msql服务

    ```shell
    # 添加systemd配置
    echo '[Unit]
    Description=MySQL Server
    Documentation=man:mysqld(8)
    Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
    After=network.target
    After=syslog.target

    [Install]
    WantedBy=multi-user.target

    [Service]
    User=mysql
    Group=mysql

    # Have mysqld write its state to the systemd notify socket
    Type=notify

    # Disable service start and stop timeout logic of systemd for mysqld service.
    TimeoutSec=0

    # Start main service
    # $MYSQL_OPTS暂时用不上
    # ExecStart=/usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf $MYSQLD_OPTS 
    ExecStart=/usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf

    # Use this to switch malloc implementation
    EnvironmentFile=-/etc/sysconfig/mysql

    # Sets open_files_limit
    LimitNOFILE = 10000

    Restart=on-failure

    RestartPreventExitStatus=1

    # Set environment variable MYSQLD_PARENT_PID. This is required for restart.
    Environment=MYSQLD_PARENT_PID=1

    PrivateTmp=false' | sudo tee /usr/lib/systemd/system/mysqld.service

    # 添加自启动
    sudo systemctl enable mysqld.service

    # 启动mysqld.service
    sudo systemctl start mysqld.service

    # 将mysql客户端链接到/usr/bin目录下
    sudo ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql

    # 连接mysql服务
    mysql -u root --skip-password
    ```

6. 修改root密码并新增用户

    ```sql
    -- 修改root密码
    ALTER USER 'root'@'localhost' IDENTIFIED BY 'root-password';
    -- 创建新用户
    CREATE USER 'admin'@'%' IDENTIFIED BY 'user-password';
    ```

7. 配置防火墙

    ```shell
    # 检查端口是否能连接
    telnet 192.168.0.15 3306

    # 将3306允许对外访问
    sudo firewall-cmd --zone=public --add-port=3306/tcp --permanent
    sudo firewall-cmd --reload
    ```
