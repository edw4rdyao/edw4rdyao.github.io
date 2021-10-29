# Oracle数据库指令

## 启动监听

```shell
# 启动监听
$ su - oracle
$ lsnrctl start

# 查看默认监听端口1521的监听状态
$ netstat -an |grep 1521
```

## 启动数据库

```shell
$ sqlplus /nolog
```

## 登录数据库

```shell
# 使用sysdba(超级用户)账号
> connect / as sysdba

# 查看是否已启动数据库
> startup
```

## 用户管理

```shell
# 使用sysdba账号登陆后,可以修改其他账号密码
> alter user [username] account unlock;      # 解除锁定(必须带;号)
> alter user [username] identified by [pwd]; # 修改密码

# 登录任意的一个用户可以通过一下方法来知道当前有哪些用户
> select distinct owner from all_objects
```

## 查看服务名

```shell
> select global_name from global_name;
```

## 退出数据库

```shell
> exit
```

## 设置输出格式

```shell
# 设置sqlplus输出的最大行宽
> set linesize 200 pagesize 1000

# 设置页面之间的空行数
> set newpage

# 将列name（字符型）显示最大宽度调整为50个字符
col name format a50

# 将列num（num型）显示最大宽度调整为7个字符
col num format 9999999
```
