## 修改容器运行时的 memlock 限制 ( es 配置启动检查为 false 可以不修改 )

如果无法在运行时上修改 memlock 限制，可以给 pod 特权后在 /usr/local/bin/docker-entrypoint.sh 启动脚本中加入临时修改命令 `ulimit -l unlimited`

```shell
:~# cat /lib/systemd/system/containerd.service
[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target local-fs.target

[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd
Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5
LimitNPROC=infinity
LimitCORE=infinity
LimitNOFILE=infinity
LimitMEMLOCK=infinity
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target
```

## 配置索引模板

```shell
ES_URL='http://ES_IP:ES_PORT'

# 配置默认模板 ( 换成 GET 方法即可获取模板详细信息 )
# 6 分片，1 副本
curl -s -XPUT -H 'Content-Type:application/json' "$ES_URL/_template/default-index-template" -d '
{
    "index_patterns": [
        "*"
    ],
    "settings": {
        "number_of_shards": 6,
        "number_of_replicas": 1
    }
}'

# 创建测试索引 ( 换成 GET 方法即可请求索引 )
curl -s -XPUT -H 'Content-Type: application/json' "$ES_URL/test-index/_doc/abc" -d'
{
    "a": 1,
    "b": 2,
    "c": 3
}'
```

> 返回值
>
> ```shell
> # 配置索引模板返回信息
> {
> 	"acknowledged": true
> }
> 
> # 创建测试索引返回信息
> {
> 	"_index": "test-index",
> 	"_type": "_doc",
> 	"_id": "abc",
> 	"_version": 1,
> 	"result": "created",
> 	"_shards": {
> 		"total": 1,
> 		"successful": 1,
> 		"failed": 0
> 	},
> 	"_seq_no": 0,
> 	"_primary_term": 1
> }
> ```
>
> 使用 `GET` 方法请求 `/_template/` 路径即可查看所有模板
