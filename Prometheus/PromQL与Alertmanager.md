### PromQL

1. 基础查询
	1. 查询指标
		- `node_cpu_seconds_total` 
	2. Label过滤
		- `node_cpu_seconds_total{cpu="7"}` 
	3. 区分时间序列

2. 数据类型
	1. Instant Vector 瞬时向量（最重要？）
		- 某一个时刻，所有符合条件的时间序列
	2. Range Vector 区间向量
		- 看一段时间 `node_cpu_seconds_total[5m]` 。为什么要看它？因为需要看变化速度——> rate
	3. Scalar 标量
	4. String 字符串

3. 运算
	- 算术运算
	- 比较运算
	- 逻辑运算

4. 聚合
	1. sum()
		- `sum(metric)` 
	2. avg()
		- `avg(node_cpu_seconds_total)` 
	3. max()
	4. min()
	5. by() 
		- 按标签分组
		- `sum by(instance)(node_cpu_seconds_total)`



5. 时间函数
	 - rate()
		 - 如果想知道每秒多少呢？ `rate(http_requests_total[5m])` 过去5分钟平均增长速度
		 - CPU使用率经典计算公式    `100 - ( avg by(instance)(rate(node_cpu_seconds_total{ mode="idle" }[5m])) *100 )`
	 - irate()
	 - increase()





### Alert 

1. Alert rule
	1. expr
	2. for
	3. labels
	4. annotations

2. Alert 状态
	1. pending观察中
	2. firing告警中
	3. resolved已恢复


3. Alertmanager
	1. 配置文件
	2. receivers
	3. route
	4. grouping
	5. silence
	6. inhibition

4. 实战
	1. CPU告警
	2. 内存告警
	3. 磁盘告警
	4. 服务down告警



Alert Rule

CPU告警
```yaml
groups:
- name: node_alert
  rules:
  - alert: HighCPU
    expr: 100 - (avg by(instance)(rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 90
    for: 5m
    labels:
      severity: warning
    annotations:
      summary: "CPU 使用率过高"
```



### 算了直接实战

```bash
wget https://github.com/prometheus/alertmanager/releases/download/v0.33.1/alertmanager-0.33.1.linux-amd64.tar.gz

tar -zxvf alertmanager-0.33.1.linux-amd64.tar.gz -C /data/


cd  /data/
ln -sv alertmanager-0.33.1.linux-amd64   alertmanager


chown -R root:root alertmanager*


```

```bash

cat alertmanager.yml 


global:                                # 全局配置模块
  resolve_timeout: 5m                # 用于设置处理超时时间，默认是5分钟
route:                                # 路由配置模块
  group_by: ['alertname']            # 告警分组
  group_wait: 10s                    # 10s内收到的同组告警在同一条告警通知中发送出去
  group_interval: 10s                # 同组之间发送告警通知的时间间隔
  repeat_interval: 1h                # 相同告警信息发送重复告警的周期
  receiver: 'web.hook'                # 使用的接收器名称
receivers:                            # 接收器
- name: 'web.hook'                    # 接收器名称
  webhook_configs:                    # 设置webhook地址
  - url: 'http://127.0.0.1:5001/'
inhibit_rules:                        # 告警抑制功能模块
  - source_match:
      severity: 'critical'            # 当存在源标签告警触发时抑制含有目标标签的告警
    target_match:
      severity: 'warning'            
    equal: ['alertname', 'dev', 'instance']     # 保证该配置下标签内容相同才会被抑制

```

编辑配置文件
```bash

vim /data/alertmanager/alertmanager.yml

global:
  resolve_timeout: 5m


# 告警路由
route:
  group_by: ['alertname']
  group_wait: 10s
  group_interval: 1m
  repeat_interval: 1h
  receiver: 'default-receiver'


# 接收器
receivers:
- name: 'default-receiver'


# 告警抑制
inhibit_rules:
- source_match:
    severity: 'critical'
  target_match:
    severity: 'warning'
  equal: ['alertname', 'instance']

```

```bash
vim /etc/systemd/system/alertmanager.service

[Unit]
Description=Alertmanager

[Service]
ExecStart=/data/alertmanager/alertmanager \
--config.file=/data/alertmanager/alertmanager.yml

Restart=always

[Install]
WantedBy=multi-user.target



cd /data/alertmanager

./amtool check-config alertmanager.yml



systemctl daemon-reload
systemctl start alertmanager
systemctl enable alertmanager

ss -nlt
#9093
```


关联prometheus

```bash
cd /data/prometheus
vim prometheus.yml

global:
  scrape_interval: 15s
  evaluation_interval: 15s


alerting:
  alertmanagers:
    - static_configs:
        - targets:
            - "localhost:9093"


rule_files:
  - "/data/prometheus/rules/*.yml"


scrape_configs:
  - job_name: "prometheus"
    static_configs:
      - targets:
          - "localhost:9090"


  - job_name: "node_exporter"
    static_configs:
      - targets:
          - "192.168.60.130:9100"
            
            
            

./promtool check config prometheus.yml
```




添加告警规则并在prometheus配置文件中导入
```bash

mkdir -p /data/prometheus/rules/

vim prometheus.yml

rule_files:
  - "/data/prometheus/rules/*.yml"

```

```bash
cd /data/prometheus/rules

vim rules.yml


groups:
- name: up
  rules:
  - alert: node
    expr: up{job="node_exporter"} == 0
    for: 1m
    labels:
      severity: critical
    annotations:
      description: "Node has been dwon for more than 1 minutes"
      summary: "Node down"


cd ..
./promtool check  rules rules/rules.yml

systemctl restart prometheus

```



### 检测脚本

```bash
vim check_monitor.sh

#!/bin/bash

echo "========== 1. 服务状态 =========="

systemctl is-active node_exporter 2>/dev/null || echo "node_exporter NOT RUNNING"
systemctl is-active prometheus 2>/dev/null || echo "prometheus NOT RUNNING"
systemctl is-active alertmanager 2>/dev/null || echo "alertmanager NOT RUNNING"
systemctl is-active grafana-server 2>/dev/null || echo "grafana NOT RUNNING"


echo
echo "========== 2. 端口监听 =========="

ss -nltp | egrep '3000|9090|9093|9100'


echo
echo "========== 3. Node Exporter =========="

curl -s localhost:9100/metrics | grep node_cpu_seconds_total | head -2


echo
echo "========== 4. Prometheus 配置 =========="

cd /data/prometheus || exit

./promtool check config prometheus.yml


echo
echo "========== 5. Prometheus Rules =========="

if ls /data/prometheus/rules/*.yml >/dev/null 2>&1
then
    ./promtool check rules /data/prometheus/rules/*.yml
else
    echo "No rule files found"
fi


echo
echo "========== 6. Prometheus连接Alertmanager =========="

curl -s localhost:9090/api/v1/alertmanagers


echo
echo
echo "========== 7. 当前告警 =========="

curl -s localhost:9090/api/v1/alerts


echo
echo
echo "========== 8. Alertmanager收到的告警 =========="

curl -s localhost:9093/api/v2/alerts


echo
echo
echo "========== END =========="
```

```bash

chmod +x check_monitor.sh

./check_monitor.sh
```