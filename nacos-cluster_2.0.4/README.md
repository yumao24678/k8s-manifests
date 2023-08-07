[TOC]

***

# \[ k8s 部署 nacos 集群 \] 

# 基础环境准备

## 1. 配置 nacos 数据库

### 1.1. 创建数据库及用户

```sql
CREATE DATABASE IF NOT EXISTS `nacos_config` DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;
CREATE USER IF NOT EXISTS 'nacos'@'172.%' IDENTIFIED WITH mysql_native_password BY 'tk4IIb6E4CFAECn';
GRANT ALL ON `nacos_config`.* TO 'nacos'@'172.%';
FLUSH PRIVILEGES;
```

### 1.2. 导入数据

下载 sql 数据文件

数据文件地址: `https://raw.githubusercontent.com/alibaba/nacos/2.0.4/distribution/conf/nacos-mysql.sql` ( 不同版本的文件名可能会不同 )

```shell
curl -SL -o "/tmp/nacos-mysql.sql" "https://raw.githubusercontent.com/alibaba/nacos/2.0.4/distribution/conf/nacos-mysql.sql"
```

导入数据

```shell
mysql \
  -h"database-1.cluster-cqiqr9iaq5so.ap-southeast-1.rds.amazonaws.com" \
  -uadmin \
  -pouEqffpU5U_ycFw \
  nacos_config </tmp/nacos-mysql.sql

# 返回值为 0 表示导入成功
echo $?
0
```

> 注意需要指定数据库

检查数据库

```sql
show tables from nacos_config;
/*
+------------------------+
| Tables_in_nacos_config |
+------------------------+
| config_info            |
| config_info_aggr       |
| config_info_beta       |
| config_info_tag        |
| config_tags_relation   |
| group_capacity         |
| his_config_info        |
| permissions            |
| roles                  |
| tenant_capacity        |
| tenant_info            |
| users                  |
+------------------------+
12 rows in set (0.00 sec)
*/
```

## 2. 将用户名和密码加密并替换到 secret 资源清单

```shell
echo -n "nacos" |base64
bmFjb3M=

echo -n "tk4IIb6E4CFAECn" |base64
dGs0SUliNkU0Q0ZBRUNu
```

secret 资源文件

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: nacos
  namespace: nacos
type: Opaque
data:
  # 将加密后的用户名和密码替换到此处两个 key 中
  MYSQL_SERVICE_USER: bmFjb3M=
  MYSQL_SERVICE_PASSWORD: dGs0SUliNkU0Q0ZBRUNu
```



访问地址: `http://IP:PORT/nacos/` 

默认用户密码: `nacos/nacos` 



