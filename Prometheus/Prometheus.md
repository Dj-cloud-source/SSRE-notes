### 大纲
1. 基础概念
	1. Metrics
	2. Time Series
	3. Label
	4. Pull

2. 安装部署
	- Prometheus
	- Node Exporter
	- Grafana（可视化）

3. PromQL
	1. 查询指标
	2. 聚合
	3. rate( )
	4. histogram

4. 指标
	1. CPU
	2. Memory
	3. Disk
	4. Network

5. 告警
	- Alert rule
	- Alertmanager（告警发送）




Prometheus的数据模型：指标名 + 标签 + 数值 + 时间

Metrics
```
<metrics_name>{label_name="label_value",……,key="value"} value
```



Time Series
时间序列是一个指标随着时间变化形成的数据流
```markdown
#cpu
10:00 30%
10:01 35%
10:02 60% 
10:03 80%
```


Label
 prometheus靠标签区分数据
```markdown

cpu_usage{
instance="server01",
cpu="0"
}12345

```

以此区分
```
server01 CPU
server02 CPU
```



Pull

传统监控是服务器把数据推送到监控平台

而Prometheus是主动发送http请求，从Exporter里取metrics数据


那什么是Exporter？
在Prometheus中，实际监控样本数据的收集都是由Exporter完成，Prometheus服务器只需要定时从这些Exporter提供的http服务获取数据即可。Exporter负责把服务器系统数据换成Prometheus能解析的Metrics格式，并提供http接口









### 部署Prometheus
```bash
wget https://github.com/prometheus/prometheus/releases/download/v3.13.1/prometheus-3.13.1.linux-amd64.tar.gz


sha256sum prometheus*



mkdir /data
#-C 解压到指定目录
tar -zxvf prometheus-*.tar.gz  -C  /data

cd /data
chown -R root:root  /data/prometheus-3.13.1.linux-amd64/


ln -sv prometheus-3.13.1.linux-amd64/ prometheus


systemctl stop firewalld
setenforce 0

cd /data/prometheus


#去另一个窗口
ss -nlt

```


创建自启动脚本（写system的脚本才能用systemctl ）
```bash

vim /usr/lib/systemd/system/prometheus.service

[Unit]
Description=Prometheus Server
Documentation=https://prometheus.io/docs/introduction/overview/
After=network.target
[Service]
Type=simple
Restart=on-failure
ExecStart=/data/prometheus/prometheus \
    --config.file=/data/prometheus/prometheus.yml \
    --storage.tsdb.path=/data/prometheus/data \
    --web.listen-address=:9090 \
    --web.enable-lifecycle
ExecReload=/bin/kill -HUP $MAINPID
[Install]
WantedBy=multi-user.target

```

```bash
systemctl  daemon-reload
systemctl  start  prometheus
systemctl enable prometheus 

ss -nlt


#然后浏览器访问9090端口
```



### 部署Exporter

```bash

curl 192.168.175.10:9090/metrics

```

部署Exporter
```bash

wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz

mkdir  /data
tar -zxvf node_exporter-1.8.2.linux-amd64.tar.gz  -C  /data

cd /data
chown -R root:root  node_exporter-1.8.2.linux-amd64/
ln -sv  node_exporter-1.8.2.linux-amd64  node_exporter



vim /usr/lib/systemd/system/node_exporter.service

[Unit]
Description=Node Exporter

[Service]
ExecStart=/data/node_exporter/node_exporter
Restart=always

[Install]
WantedBy=multi-user.target





systemctl daemon-reload

systemctl start node_exporter

systemctl enable node_exporter


#注意监听端口
#Listening on" address=[::]:9100

ss -nlt
curl localhost:9100/metrics

#node_exporter 是9100

```

### 关联Prometheus

```bash
cd  /data/prometheus
vim  prometheus.yml   

scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "node_exporter"
    
    static_configs:
      - targets: ["192.168.88.20:9100"]
        
        
#注意缩进
```



> 
  配置文件解释
global:
  scrape_interval：每次数据采集的时间间隔，默认为1分钟
  scrape_timeout：采集请求超时时间，默认为10秒
  evaluation_interval：执行rules的频率，默认为1分钟
scrape_configs：主要用于配置被采集数据节点操作，每一个采集配置主要由以下几个参数
  job_name：全局唯一名称
    scrape_interval：默认等于global内设置的参数，设置后可以覆盖global中的值
    scrape_timeout：默认等于global内设置的参数
    metrics_path：从targets获取meitric的HTTP资源路径，默认是/metrics
    honor_labels：Prometheus如何处理标签之间的冲突。若设置为True，则通过保留变迁来解决冲突；若设置为false，则通过重命名；
    scheme：用于请求的协议方式，默认是http
    params：数据采集访问时HTTP URL设定的参数
    relabel_configs：采集数据重置标签配置
    metric_relabel_configs：重置标签配置
    sample_limit：对每个被已知样本数量的每次采集进行限制，如果超过限制，该数据将被视为失败。默认值为0，表示无限制



```bash
./promtool  check  config  prometheus.yml

systemctl  restart prometheus
ss -nlt

#然后浏览器就可以看见了
```



[MySQL监控](https://ncloud.eagleslab.com/%E7%9B%91%E6%8E%A7%E5%91%8A%E8%AD%A6/Prometheus.html#mysql%E7%9B%91%E6%8E%A7) 


### 部署 Grafana

```bash
yum install -y https://dl.grafana.com/grafana-enterprise/release/13.1.1/grafana-enterprise_13.1.1_29761037902_linux_amd64.rpm


rpm -q grafana-enterprise


systemctl daemon-reload
systemctl start grafana-server
systemctl status grafana-server
#Grafana是3000端口



#浏览器打开http://ip:3000
#用户名 admin 
#密码 admin
#首次登陆提示改密码

#添加数据源 add data source
# URL      http://ip:9090

```

[使用Grafana](https://ncloud.eagleslab.com/%E7%9B%91%E6%8E%A7%E5%91%8A%E8%AD%A6/Prometheus.html#%E4%BD%BF%E7%94%A8grafana) 




### Alertmanager

[课件](https://ncloud.eagleslab.com/%E7%9B%91%E6%8E%A7%E5%91%8A%E8%AD%A6/Prometheus.html#alertmanager%E6%9C%BA%E5%88%B6) 