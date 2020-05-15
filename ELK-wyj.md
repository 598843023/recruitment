# ELK部署
任务名称：ELK日志收集系统搭建  

安装与配置文档：https://www.elastic.co/guide/index.html

支持平台：linux | windows | docker

任务提交者：wuyinjun

任务测试环境：阿里云服务器 http://139.224.135.141/

# 介绍
&emsp;&emsp;ELK是Elasticsearch、Logstash、Kibana三大开源框架首字母大写简称。市面上也被成为Elastic Stack。其中Elasticsearch是一个基于Lucene、分布式、通过Restful方式进行交互的近实时搜索平台框架。像类似百度、谷歌这种大数据全文搜索引擎的场景都可以使用Elasticsearch作为底层支持框架，可见Elasticsearch提供的搜索能力确实强大,市面上很多时候我们简称Elasticsearch为es。Logstash是ELK的中央数据流引擎，用于从不同目标（文件/数据存储/MQ）收集的不同格式数据，经过过滤后支持输出到不同目的地（文件/MQ/redis/elasticsearch/kafka等）。Kibana可以将elasticsearch的数据通过友好的页面展示出来，提供实时分析的功能。

# 环境要求
* 依赖组件：jdk8以上

* 服务器配置：vcpu 2，内存 4G 或更多

# 安装说明
&emsp;&emsp;这里本人使用yum安装，建议使用官方自身提供的elasticsearch、kibana、logstash的仓库，不建议使用操作系统自带的仓库或其他第三方仓库。

下面进行安装说明

## 创建elasticsearch、kibana、logstash官方仓库
```bash
cd /etc/yum.repos.d  
#切换到仓库目录

vim elasticsearch.repo  
#创建elasticsearch仓库

[elasticsearch]  
name=Elasticsearch repository for 7.x packages  
baseurl=https://artifacts.elastic.co/packages/7.x/yum  
gpgcheck=1  
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch  
enabled=0  
autorefresh=1  
type=rpm-md  
#保存退出

vim kibana.repo  
#创建kibana仓库

[kibana-7.x]  
name=Kibana repository for 7.x packages  
baseurl=https://artifacts.elastic.co/packages/7.x/yum  
gpgcheck=1  
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch  
enabled=1  
autorefresh=1  
type=rpm-md  
#保存退出

vim logstash.repo  
#创建logstash仓库

[logstash-7.x]  
name=Elastic repository for 7.x packages  
baseurl=https://artifacts.elastic.co/packages/7.x/yum  
gpgcheck=1  
gpgkey=https://artifacts.elastic.co/GPG-KEY-elasticsearch  
enabled=1  
autorefresh=1  
type=rpm-md

yum clean all  
#清理yum缓存

yum makecache  
#生成yum缓存
```
## 安装elasticsearch
```bash
yum install -y --enablerepo=elasticsearch elasticsearch  
#利用yum安装elasticsearch

vim /etc/elasticsearch/elasticsearch.yml  
#编辑elasticsearch配置文件

path.data: /var/lib/elasticsearch # ES数据保存目录  
path.logs: /var/log/elasticsearch # ES日志保存目录  
network.host: 172.19.61.146  # 监听ip  
http.port: 9200 # 监听端口  
discovery.seed_hosts: ["172.19.61.146","[::9200]"]  # 集群中node节点发现列表


systemctl enable elasticsearch.service  
#加入开机启动  

systemctl start elasticsearch.service  
#启动elasticsearch

curl localhost:9200  
#测试elasticsearch启动是否成功  
{  
    "name" : "Cp8oag6",  
     "cluster_name" : "elasticsearch",  
     "cluster_uuid" : "AT69_T_DTp-1qgIJlatQqA",  
     "version" : {  
       "number" : "7.7.0",  
       "build_flavor" : "default",  
       "build_type" : "tar",  
       "build_hash" : "f27399d",  
       "build_date" : "2016-03-30T09:51:41.449Z",  
       "build_snapshot" : false,   
       "lucene_version" : "8.5.1",   
       "minimum_wire_compatibility_version" : "1.2.3",   
       "minimum_index_compatibility_version" : "1.2.3"   
    },   
    "tagline" : "You Know, for Search"   
}
```

## 安装kibana

```bash
yum install -y kibana  
#yum安装kibana

vim /etc/kibana/kibana.yml  
#编辑kibana配置文件

server.port: 5601 #监听端口  
server.host: "172.19.61.146" #监听地址  
elasticsearch.hosts: ["http://172.19.61.146:9200"]   
#elasticsearch服务器地址  
i18n.locale: "zh-CN" #支持中文  
systemctl enable kibana.service  
#加入开机启动项

systemctl start kibana.service  
#启动kibana服务

使用浏览器访问 139.224.135.141/status查看是否安装成功
```

## 安装logstash

```bash
&emsp;&emsp;Logstash需要Java 8或Java11。请使用 正式的Oracle发行版或开源发行版，例如OpenJDK。

java -version  
#检查你的java版本

yum install -y java-11-openjdk  
#这里由于是新建的实例还未安装jdk，选择安装openjdk11

yum install -y logstash  
#yum安装logstash

systemctl start logstash  
#启动logstash

/usr/share/logstash/bin/logstash -e 'input { stdin{} } 
output { stdout{ codec => rubydebug }}'  #标准输入和输出  

[INFO ] 2020-05-14 23:24:10.634 [Api Webserver] agent - Successfully started Logstash API endpoint {:port=>9600}  
hello  
/usr/share/logstash/vendor/bundle/jruby/2.5.0/gems/awesome_print-1.7.0/lib/awesome_print/formatters/base_formatter.rb:31: warning: constant ::Fixnum is deprecated  
{  
   "@timestamp" => 2020-05-14T15:24:17.746Z, #当前事件的发生时间   
   "host" => "iZhdol39nnazbsZ", #标记事件发生在哪里  
     "@version" => "1", #事件版本号，一个事件就是一个 ruby 对象   
       "message" => "hello" #消息的具体内容   
}

vim ~/.bashrc  
#给logstash设置别名永久保存（非全局变量）

alias logstash='/usr/share/logstash/bin/logstash'  
#在最后添加并保存退出

source ~/.bashrc  
#使文件生效
```

# 使用说明

&emsp;&emsp;安装ELK时 必须在整个ELK系统中使用相同版本。例如，如果您使用的是Elasticsearch 7.7.0，则安装Kibana 7.7.0和Logstash 7.7.0。

## 路径
* 程序路径：  
Elasticsearch：/usr/share/elasticsearch/bin/  
Kibana：/usr/share/kibana/bin/  
Logstash：/usr/share/logstash/bin/

* 日志路径：  
logstash：/var/log/logstash  
elasticsearch：/var/log/elasticsearch  
Kibana：/var/log/Kibana

* 配置文件路径：
kibana：/etc/kibana/  
elasticsearch：/etc/elasticsearch/  
logstash： /etc/logstash/

## 版本号

使用如下命令查看版本号

```bash
elasticsearch：/usr/share/elasticsearch/bin/elasticsearch --version

logstash：logstash --version
```

## 端口号
本项目需要开启的端口号如下：
| item | port|  
 ---  | ---  | 
| elasticsearch|  9200|  
| Kibana | 5601|   
| logstash | 9300| 

# 日志
2020-05-14 完成CentOS安装研究