---
title: "Puxiang -- ELK Deployment Report"
author: [Leon]
date: "2019-05"
subject: "elastic"
keywords: [elastic, deployment, puxiang]
subtitle: "Best Practice deploy ELK in prod" 
lang: "en"
titlepage: true
titlepage-color: "06386e"
titlepage-text-color: "FFFFFF"
titlepage-rule-color: "FFFFFF"
titlepage-rule-height: 1
toc-own-page: true
CJKmainfont: "PingFangSC-Regular"
logo: "logo.png"
...

# 硬件資源

## macow

| Role          | Hostname          | IP         | CPU (Cores) | Memory (GB) | DIsk (GB) |
| ------------- | ----------------- | ---------- | ----------- | ----------- | --------- |
| kibana        | SRV-LOG-KBN01     | 10.20.0.60 | 4           | 8           | 50        |
| logstash      | SRV-LOG-LOGSTH01  | 10.20.0.61 | 8           | 16          | 200       |
| logstash      | SRV-LOG-LOGSTH02  | 10.20.0.62 | 8           | 16          | 200       |
| logstash      | SRV-LOG-LOGSTH03  | 10.20.0.63 | 8           | 16          | 200       |
| elasticsearch | SRV-LOG-ELASTIC01 | 10.20.0.64 | 16          | 64          | 2000      |
| elasticsearch | SRV-LOG-ELASTIC02 | 10.20.0.65 | 16          | 64          | 2000      |
| elasticsearch | SRV-LOG-ELASTIC03 | 10.20.0.66 | 16          | 64          | 2000      |

## manila

| Role | Hostname | IP   | CPU (Cores) | Memory (GB) | DIsk (GB) |
| ---- | -------- | ---- | ----------- | ----------- | --------- |
| kibana        | SRV-LOG-KBN02     | 10.20.0.70 | 4           | 8           | 50        |
| logstash      | SRV-LOG-LOGSTH04  | 10.20.0.71 | 8           | 16          | 200       |
| logstash      | SRV-LOG-LOGSTH05  | 10.20.0.72 | 8           | 16          | 200       |
| logstash      | SRV-LOG-LOGSTH06  | 10.20.0.73 | 8           | 16          | 200       |
| elasticsearch | SRV-LOG-ELASTIC04 | 10.20.0.74 | 16          | 64          | 2000      |
| elasticsearch | SRV-LOG-ELASTIC05 | 10.20.0.75 | 16          | 64          | 2000      |
| elasticsearch | SRV-LOG-ELASTIC06 | 10.20.0.76 | 16          | 64          | 2000      |

\pagebreak

# 軟件架構

![架构图.001](http://ww2.sinaimg.cn/large/006tNc79gy1g347nuhbs0j31hc0u0gpb.jpg)

\pagebreak

# 安裝部署

## 準備工作

### 安裝 ansible 

```shell
[root@SRV-LOG-KBN01 ansible-elk]$ yum install -y ansible   .
```

### 编辑 ansible inventory 文件

```
[elasticsearch]
es01 ansible_host=10.20.0.64
es02 ansible_host=10.20.0.65
es03 ansible_host=10.20.0.66

[kibana:vars]
es_hosts=10.20.0.064

[kibana]
kibana01 ansible_host=10.20.0.60

[logstash]
logstash01 ansible_host=10.20.0.61
logstash02 ansible_host=10.20.0.62
logstash03 ansible_host=10.20.0.63
```

### 關閉防火墻

```shell
[sysadmin@SRV-LOG-KBN01 ansible-elk]$ ansible -i inventory all -m shell -a 'systemctl stop firewalld && systemctl disable firewalld'
```

### 關閉 selinux 

```shell
[sysadmin@SRV-LOG-KBN01 ansible-elk]$ ansible -i inventory all -m shell -a "sed -ri s/enforcing/disabled/ /etc/selinux/config && setenforce 0"
```

### 檢查磁盤掛載

\pagebreak

## 安裝 elasticsearch

```shell
[sysadmin@SRV-LOG-KBN01 ansible-elk]$ ansible-playbook -i inventory playbook/elasticsearch.yml 
```
執行結果：
```shell
PLAY [elasticsearch] *************************************

TASK [Gathering Facts] ***********************************
ok: [es01]
ok: [es02]
ok: [es03]

TASK [prepare : Creates directory] ***********************
changed: [es01]
changed: [es02]
changed: [es03]

TASK [system : pam_limits] *******************************
changed: [es01]
changed: [es03]
changed: [es02]

TASK [elasticsearch : download jdk rpm file] *************
changed: [es03]
changed: [es02]
changed: [es01]

TASK [elasticsearch : Install jdk] ***********************
changed: [es01]
changed: [es02]
changed: [es03]

TASK [elasticsearch : download elasticsearch rpm file] ***
changed: [es01]
changed: [es03]
changed: [es02]

TASK [elasticsearch : Install elasticsearch] *************
changed: [es03]
changed: [es01]
changed: [es02]

TASK [elasticsearch : Copy templated elasticsearch.yml] **
changed: [es01]
changed: [es02]
changed: [es03]

TASK [elasticsearch : Copy templated jvm.options] ********
changed: [es01]
changed: [es03]
changed: [es02]

TASK [elasticsearch : Start Elasticsearch Service] *******
changed: [es01]
changed: [es03]
changed: [es02]

PLAY RECAP ***********************************************
es01      : ok=16   changed=14   unreachable=0    failed=0 
es02      : ok=16   changed=14   unreachable=0    failed=0 
es03      : ok=16   changed=14   unreachable=0    failed=0 
```

\pagebreak

## 安裝 kibana

```shell
[sysadmin@SRV-LOG-KBN01 ansible-elk]$ ansible-playbook -i inventory playbook/kibana.yml
```

執行結果：

```shell
PLAY [kibana] *********************************************

TASK [Gathering Facts] ************************************
ok: [kibana01]

TASK [prepare : Creates directory] ************************
changed: [kibana01]

TASK [kibana : download kibana rpm file] ******************
changed: [kibana01]

TASK [kibana : Install kibana] ****************************
changed: [kibana01]

TASK [kibana : Copy templated kibana.yml] *****************
changed: [kibana01]

TASK [kibana : systemd] ***********************************
changed: [kibana01]

PLAY RECAP ************************************************
kibana01   : ok=6    changed=5    unreachable=0    failed=0
```

\pagebreak

## 安裝 Logstash

```shell
[sysadmin@SRV-LOG-KBN01 ansible-elk]$ ansible-playbook -i inventory playbook/logstash.yml
```

執行結果：

```
PLAY [logstash] *****************************************************

TASK [Gathering Facts] **********************************************
ok: [logstash01]
ok: [logstash02]
ok: [logstash03]

TASK [prepare : Creates directory] **********************************
changed: [logstash01]
changed: [logstash02]
changed: [logstash03]

TASK [logstash : download jdk rpm file] *****************************
changed: [logstash01]
changed: [logstash03]
changed: [logstash02]

TASK [logstash : Install jdk] ***************************************
changed: [logstash02]
changed: [logstash01]
changed: [logstash03]

TASK [logstash : download logstash rpm file] ************************
changed: [logstash03]
changed: [logstash01]
changed: [logstash02]

TASK [logstash : Install logstash] **********************************
changed: [logstash03]
changed: [logstash02]
changed: [logstash01]

TASK [logstash : Copy templated logstash.yml] ***********************
changed: [logstash01]
changed: [logstash02]
changed: [logstash03]

TASK [logstash : Copy templated jvm.options] ************************
changed: [logstash01]
changed: [logstash03]
changed: [logstash02]

TASK [logstash : systemd] *******************************************
changed: [logstash03]
changed: [logstash01]
changed: [logstash02]

PLAY RECAP **********************************************************
logstash01     : ok=9    changed=8    unreachable=0    failed=0   
logstash02     : ok=9    changed=8    unreachable=0    failed=0   
logstash03     : ok=9    changed=8    unreachable=0    failed=0 
```

\pagebreak

## 開啟 x-pack

### 準備工作

在 elasticsearch 機器上安裝 python 的 pyyaml 模塊，用於 yaml 文件編輯 

```shell
[sysadmin@SRV-LOG-KBN01 ansible-elk]$ ansible -i inventory elasticsearch -m shell -a 'curl https://bootstrap.pypa.io/get-pip.py | python && pip install pyyaml'
```

### 開啟 x-pack 安全功能，并生成 ssl 證書

```shell
[sysadmin@SRV-LOG-KBN01 ansible-elk]$ ansible-playbook -i inventory playbook/xpack.yml 
```

執行結果

```shell
PLAY [elasticsearch] ****************************************

TASK [Gathering Facts] **************************************
ok: [es02]
ok: [es01]
ok: [es03]

TASK [xpack-elasticsearch : Copy ca file] *******************
changed: [es02]
changed: [es01]
changed: [es03]

TASK [xpack-elasticsearch : Generate certificate] ***********
changed: [es02]
changed: [es01]
changed: [es03]

TASK [xpack-elasticsearch : add xpack security configs] *****
changed: [es01]
changed: [es02]
changed: [es03]

TASK [xpack-elasticsearch : Start Elasticsearch Service] ****
changed: [es01]
changed: [es03]
changed: [es02]

PLAY RECAP **************************************************
es01       : ok=5    changed=4    unreachable=0    failed=0
es02       : ok=5    changed=4    unreachable=0    failed=0
es03       : ok=5    changed=4    unreachable=0    failed=0
```

### 激活 License

訪問 kibana -> management -> license，并上傳提前準備好的 license json file

![image-20190517125817057](http://ww3.sinaimg.cn/large/006tNc79gy1g34855t6s9j31l30u0n6j.jpg)

### 生成賬號密碼

登錄一台 elasticsearch，執行以下命令生成 credentials

```shell
[root@SRV-LOG-ELASTIC01 sysadmin]$ /usr/share/elasticsearch/bin/elasticsearch-setup-passwords auto
```

得到執行結果并保存

```shell
Initiating the setup of passwords for reserved users elastic,apm_system,kibana,logstash_system,beats_system,remote_monitoring_user.
The passwords will be randomly generated and printed to the console.
Please confirm that you would like to continue [y/N]y


Changed password for user apm_system
PASSWORD apm_system = ****************

Changed password for user kibana
PASSWORD kibana = ****************

Changed password for user logstash_system
PASSWORD logstash_system = ****************

Changed password for user beats_system
PASSWORD beats_system = ****************

Changed password for user remote_monitoring_user
PASSWORD remote_monitoring_user = ****************

Changed password for user elastic
PASSWORD elastic = ****************
```

### kibana 與有訪問控制的 es 通信

修改 kibana.yml 配置

```shell
[root@SRV-LOG-KBN01 ansible-elk]$ vim /etc/kibana/kibana.yml
```

去掉以下兩行內容的注釋并修改用戶名密碼

```shell
elasticsearch.username: "kibana"
elasticsearch.password: "******"
```

重啟 kibana

```shell
systemctl restart kibana
```

### 為 logstash 開啟集中式管理

創建 logstash_admin 用戶，賦予 logstash_system, logstash_admin 角配置色，配置集中式配置管理：

![image-20190517130631944](http://ww1.sinaimg.cn/large/006tNc79gy1g348dqwuznj31h30u07g6.jpg)

登錄 logstash 機器，編輯 /etc/logstash/logstash.yml，添加以下內容

```shell
xpack.monitoring.elasticsearch.username: logstash_system
xpack.monitoring.elasticsearch.password: **********
xpack.management.enabled: true
xpack.management.pipeline.id: ["beats"， "syslog"]
xpack.management.elasticsearch.username: logstash_admin
xpack.management.elasticsearch.password: **********
xpack.management.elasticsearch.hosts: ["http://10.20.0.64:9200","http://10.20.0.65:9200","http://10.20.0.66:9200"]
```

重啟 logstash

```shell
[sysadmin@SRV-LOG-KBN01 ansible-elk]$ ansible -i inventory logstash -m shell -a 'systemctl restart logstash'
```

\pagebreak

## 參數配置

### 集群動態配置

```shell
PUT _cluster/settings
{
  "persistent": {
    "cluster.routing.allocation.node_concurrent_recoveries": "10",
    "cluster.routing.allocation.node_initial_primaries_recoveries": "10",
    "cluster.routing.allocation.cluster_concurrent_rebalance": "10",
    "cluster.routing.allocation.enable": "all",
    "indices.recovery.max_bytes_per_sec": "100mb",
    "action.destructive_requires_name": "true"
  }
}
```

### 基礎模板配置

```shell
PUT _template/suncity
{
    "order": 0,
    "index_patterns": [
      "suncity_*"
    ],
    "settings": {
      "index": {
        "routing": {
          "allocation": {
            "total_shards_per_node": 1
          }
        },
        "refresh_interval": "30s",
        "number_of_shards": 1,
        "translog": {
          "sync_interval": "10s",
          "durability": "async"
        },
        "query": {
          "default_field": "message"
        },
        "number_of_replicas": 1
      }
    },
    "mappings": {
      "_default_": {
        "dynamic_templates": [
          {
            "strings": {
              "mapping": {
                "ignore_above": 256,
                "type": "keyword"
              },
              "match_mapping_type": "string"
            }
          }
        ],
        "properties": {
          "message": {
            "norms": false,
            "type": "text"
          }
        }
      }
    },
    "aliases": {}
}
```

### 創建 logstash_writer 角色 和 用戶

使用 UI 創建

![image-20190518164530497](http://ww1.sinaimg.cn/large/006tNc79gy1g35kbu2a53j31nm0u0anj.jpg)

![image-20190518005001838](http://ww4.sinaimg.cn/large/006tNc79gy1g34spsl1foj31iy0u0n8c.jpg)

使用 API 创建

```shell
POST /_security/role/logstash_writer
{
    "cluster" : [
      "manage_index_templates",
      "monitor"
    ],
    "indices" : [
      {
        "names" : [
          "*"
        ],
        "privileges" : [
          "write",
          "delete",
          "create_index"
        ],
        "field_security" : {
          "grant" : [
            "*"
          ]
        },
        "allow_restricted_indices" : false
      }
    ]
    "transient_metadata" : {
      "enabled" : true
    }
}

POST /_security/user/logstash_writer
{
    "username" : "logstash_writer",
    "roles" : [
      "logstash_writer"
    ],
    "full_name" : "logstash_writer",
    "email" : "logstash_writer@example.com",
    "enabled" : true
}
```

### 部署效果

![image-20190518180028418](http://ww4.sinaimg.cn/large/006tNc79gy1g35mhuhdhnj31ai0u0thh.jpg)



\pagebreak

# 主機日誌採集

## 數據建模

### 配置 ILM

使用 UI 配置

![image-20190518165435766](http://ww1.sinaimg.cn/large/006tNc79gy1g35klbg8clj31kw0u07jv.jpg)

使用 API 配置

```shell
PUT _ilm/policy/suncity
{
    "policy" : {
      "phases" : {
        "hot" : {
          "min_age" : "0ms",
          "actions" : {
            "rollover" : {
              "max_size" : "30gb"
            }
          }
        },
        "delete" : {
          "min_age" : "90d",
          "actions" : {
            "delete" : { }
          }
        }
      }
    }
  }
```



### 導入 beats module 的 template 和 viz

找一台機器安裝 filebeat，metricbeat, winlogbeat(optional)

```shell
filebeat setup --modules auditd,system,elasticsearch,logstash,kibana -M "system.syslog.var.convert_timezone=true" -M "system.auth.var.convert_timezone=true" -E "setup.template.pattern=suncity_filebeat_6.7.2-*" -E "setup.template.settings.index.nmber_of_shards=1" -E "setup.template.settings.index.lifecycle.rollover_alias=filebeat-6.7.2" -E "setup.template.settings.index.lifecycle.namee=suncity" -E "setup.kibana.host=10.20.0.60:5601" -E "output.elasticsearch.hosts=10.20.0.64:9200" -E "output.elasticsearch.username=elastic" -E "output.elasticsearch.password=******"

metricbeat setup -E "setup.template.pattern=suncity_metricbeat_6.7.2-*" -E "setup.template.settings.index.nmber_of_shards=1" -E "setup.template.settings.index.lifecycle.rollover_alias=metricbeat-6.7.2" -E "setup.template.settings.index.lifecycle.namee=suncity" -E "setup.kibana.host=10.20.0.60:5601" -E "output.elasticsearch.hosts=10.20.0.64:9200" -E "output.elasticsearch.username=elastic" -E "output.elasticsearch.password=******"

winlogbeat.exe setup -E "setup.template.pattern=suncity_winlogbeat_6.7.2-*" -E "setup.template.settings.index.number_of_shards=1" -E "setup.template.settings.index.lifecycle.rollover_alias=winlogbeat-6.7.2" -E "setup.template.settings.index.lifecycle.name=suncity" -E "setup.kibana.host=10.20.0.60:5601" -E "output.elasticsearch.hosts=10.20.0.64:9200" -E "output.elasticsearch.username=elastic" -E "output.elasticsearch.password==******"
```

### 創建 rollover index

```shell
PUT suncity_filebeat_6.7.2-000001
{
  "aliases": {
    "filebeat-6.7.2": {
      "is_write_index":true
    }
  }
}


PUT suncity_metricbeat_6.7.2-000001
{
  "aliases": {
    "metricbeat-6.7.2": {
      "is_write_index":true
    }
  }
}
```



### 創建 beats_admin 用戶

用于集中式管理时 enroll

![image-20190518005041646](http://ww3.sinaimg.cn/large/006tNc79gy1g34sqfqqhij31je0u0tkl.jpg)

使用 API 创建

```shell
POST /_security/user/beats_admin
{
    "username" : "beats_admin",
    "roles" : [
      "beats_admin"
    ],
    "full_name" : "beats_admin",
    "email" : "beats_admin@example.com",
    "enabled" : true
}
```



### 添加 logstash 配置

pipeline-id : beats

```shell
input {
    beats {
        port => 5044
    }
}

filter {
}

output {
  if [@metadata][pipeline] {
    elasticsearch {
      hosts => ["http://10.20.0.74:9200", "http://10.20.0.75:9200", "http://10.20.0.76:9200"]
      manage_template => false
      index => "%{[@metadata][beat]}-%{[@metadata][version]}"
      pipeline => "%{[@metadata][pipeline]}" 
      user => "logstash_writer"
      password => "**********"
    }
  } else {
    elasticsearch {
      hosts => ["http://10.20.0.74:9200", "http://10.20.0.75:9200", "http://10.20.0.76:9200"]
      manage_template => false
      index => "%{[@metadata][beat]}-%{[@metadata][version]}"
      user => "logstash_writer"
      password => "***********"
    }
  }
}
```

\pagebreak

## 部署 beats agents

### 使用 ansible 部署

1. 编辑 inventory 文件

   ```
   [beats]
   10.50.0.91 ansible_user=admin ansible_ssh_pass="*******" ansible_become_pass="******"
   10.50.0.92 ansible_user=admin ansible_ssh_pass="*******" ansible_become_pass="******"
   10.50.0.81 ansible_user=admin ansible_ssh_pass="*******" ansible_become_pass="******"
   10.50.0.83 ansible_user=admin ansible_ssh_pass="*******" ansible_become_pass="******"
   10.50.0.72 ansible_user=admin ansible_ssh_pass="*******" ansible_become_pass="******"
   .......
   ```

2. 对未安装 python 的 beats 安装 python

   ```shell
   [sysadmin@SRV-LOG-KBN01 ansible-elk]$ ansible -i inventory beats -m raw -a 'test -e /usr/bin/python || sudo apt -y install python'
   ```

3. 执行部署命令

   ```shell
   [sysadmin@SRV-LOG-KBN01 ansible-elk]$ ansible-playbook -i inventory playbook/beats.yml
   ```

   

### 使用 shell script 部署

Centos:

```shell
#!/bin/bash
KIBANA_URL=http://10.20.0.60:5601 && \
rpm -ivh https://mirrors.tuna.tsinghua.edu.cn/elasticstack/6.x/yum/6.7.2/filebeat-6.7.2-x86_64.rpm && \
PASS=beats_admin filebeat enroll $KIBANA_URL --username beats_admin --password env:PASS --force && \
systemctl enable filebeat && \
systemctl start filebeat && \
rpm -ivh https://mirrors.tuna.tsinghua.edu.cn/elasticstack/6.x/yum/6.7.2/metricbeat-6.7.2-x86_64.rpm && \
PASS=beats_admin filebeat enroll $KIBANA_URL --username beats_admin --password env:PASS --force && \
systemctl enable filebeat && \
systemctl start filebeat
```

Ubuntu:

```shell
#!/bin/bash
KIBANA_URL=http://10.20.0.60:5601 && \
curl -sLO https://mirrors.tuna.tsinghua.edu.cn/elasticstack/apt/6.x/pool/main/f/filebeat/filebeat-6.7.2-amd64.deb && \
dpkg -i filebeat-6.7.2-amd64.deb && \
PASS=beats_admin filebeat enroll $KIBANA_URL --username beats_admin --password env:PASS --force && \
systemctl enable filebeat && \
systemctl start filebeat && \
curl -sLO https://mirrors.tuna.tsinghua.edu.cn/elasticstack/apt/6.x/pool/main/m/metricbeat/metricbeat-6.7.2-amd64.deb && \
dpkg -i metricbeat-6.7.2-amd64.deb && \
PASS=beats_admin filebeat enroll $KIBANA_URL --username beats_admin --password env:PASS --force && \
systemctl enable filebeat && \
systemctl start filebeat
```

Windows:

```
1.解壓提前準備好的 zip 安裝包
2.執行壓縮包中的 deploy-beats.bat
```



\pagebreak

## 下發 beats 配置

添加 beats 配置，採集系統日誌、指標信息

![image-20190518170148859](http://ww3.sinaimg.cn/large/006tNc79gy1g35kssopabj31nd0u0tks.jpg)

下發配置到主機

![image-20190517102951803](http://ww4.sinaimg.cn/large/006tNc79gy1g343uq5w8zj31iw0u0wqy.jpg)



\pagebreak

# Firewall Switcher Router 日誌採集

## 準備工作

### 安裝 nginx syslog-ng 

工作目錄 : /home/sysadmin/ansible-elk

```shell
[sysadmin@SRV-LOG-KBN01 ansible-elk]$ ansible -i inventory logstash -m shell -a "systemctl stop rsyslog && systemctl disable rsyslog" (optional)
[sysadmin@SRV-LOG-KBN01 ansible-elk]$ ansible -i inventory logstash -m shell -a "yum install -y epel-release"
[sysadmin@SRV-LOG-KBN01 ansible-elk]$ ansible -i inventory logstash -m shell -a "yum install -y nginx syslog-ng"
[sysadmin@SRV-LOG-KBN01 ansible-elk]$ ansible -i inventory logstash -m shell -a "systemctl enable nginx syslog-ng && systemctl start nginx syslog-ng"
```

### 修改 logstash 使用 root 用戶運行

```shell
[sysadmin@SRV-LOG-KBN01 ansible-elk]$ ansible -i inventory logstash -m shell -a "sed -ri s/User=logstash/User=root/ /etc/systemd/system/logstash.service && sed -ri s/Group=logstash/Group=root/ /etc/systemd/system/logstash.service"
ansible -i inventory logstash -m shell -a "systemctl restart logstash"
```

### 編寫、上傳并測試 nginx syslog-ng logrotate 配置

files/config/logrotate-remote

```
/data/syslog/*.log {
    dateext
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 600 root root
    postrotate
        systemctl reload syslog-ng
    endscript
}
```

files/config/nginx.conf

```
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user root;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

stream {
    upstream syslog_upstreams {
        server 10.20.0.61:513;
        server 10.20.0.62:513;
        server 10.20.0.63:513;
    }

    server {
        listen 514 udp; #表明是udp
        proxy_pass syslog_upstreams;
        proxy_bind $remote_addr transparent;
        proxy_responses 0;
        proxy_buffer_size 4096k;
    }
}
```

files/config/syslog-ng-remote.conf

```
/data/syslog/*.log {
    dateext
    daily
    rotate 7
    compress
    delaycompress
    missingok
    notifempty
    create 600 root root
    postrotate
        systemctl reload syslog-ng
    endscript
}
[sysadmin@SRV-LOG-KBN01 config]$ cat syslog-ng-remote.conf 
options {
    flush_lines (0);
    time_reopen (10);
    log_fifo_size (1000);
    use_dns (no);
    use_fqdn (no);
    create_dirs (yes);
    keep_hostname (yes);
};

#Log source
source network_logsource { udp(ip(0.0.0.0) port(513) flags(no-parse)); };

#Log destination path
destination network_data {
  file(
    "/data/syslog/${HOST}_${SOURCEIP}.log"
    template("${DATE} ${MSG}\n")
  );
};

log { source(network_logsource); destination(network_data); };
```

```shell
[sysadmin@SRV-LOG-KBN01 ansible-elk]$ ansible -i inventory logstash -m copy -a 'src=/home/sysadmin/ansible-elk/files/config/nginx.conf dest=/etc/nginx/nginx.conf'
[sysadmin@SRV-LOG-KBN01 ansible-elk]$ ansible -i inventory logstash -m copy -a 'src=/home/sysadmin/ansible-elk/files/config/syslog-ng-remote.conf dest=/etc/syslog-ng/conf.d/remote.conf'
[sysadmin@SRV-LOG-KBN01 ansible-elk]$ ansible -i inventory logstash -m copy -a 'src=/home/sysadmin/ansible-elk/files/config/logrotate-remote dest=/etc/logrotate.d/remote'
[sysadmin@SRV-LOG-KBN01 ansible-elk]$ ansible -i inventory logstash -m shell -a "nginx -t && syslog-ng -s"
[sysadmin@SRV-LOG-KBN01 ansible-elk]$ ansible -i inventory logstash -m shell -a "systemctl restart nginx syslog-ng"
```

\pagebreak

## 數據建模

### 創建 模板 和 rollover index

```shell
PUT _template/suncity_syslog
{
    "order" : 1,
    "index_patterns" : [
      "suncity_syslog-*"
    ],
    "settings" : {
      "index" : {
        "lifecycle" : {
          "name" : "suncity",
          "rollover_alias" : "syslog"
        }
      }
    }
}

PUT suncity_syslog-000001
{
  "aliases": {
    "syslog": {
      "is_write_index":true
    }
  }
}
```

### 添加 logstash 配置

pipeline-id: syslog

```shell
input {
    file {
        path => "/data/syslog/*.log"
    }
}
filter {
    mutate {
        copy => { "message" => "[@metadata][message]" }
        remove_field => [ "message" ]
    }
    grok {
        match => ["path", "%{SYSLOGHOST:observer.host}_%{SYSLOGHOST:observer.ip}\.log"]
        remove_field => [ "path" ]
    }
    grok {
        match => ["[@metadata][message]", "%{SYSLOGTIMESTAMP:timestamp} <%{NONNEGINT:syslog_pri}>(?:%{SYSLOGTIMESTAMP} )?(?:%{SYSLOGHOST} )?%{GREEDYDATA:message}"]
    }
    date {
        match => [ "timestamp", "MMM dd HH:mm:ss", "MMM  d HH:mm:ss"]
        timezone => "Asia/Shanghai"
        remove_field => [ "timestamp" ]
    }
    syslog_pri {
        remove_field => [ "syslog_pri" ]
    }
    mutate {
        rename => {
            "syslog_facility" => "facility_label"
            "syslog_facility_code" => "facility"
            "syslog_severity" => "severity_label"
            "syslog_severity_code" => "severity"
        }
    }

    grok {
        match => ["message", "^%{PROG:program}\[%{POSINT:pid}?\]"]
    }
}
output {
    elasticsearch {
        hosts => ["http://10.20.0.74:9200", "http://10.20.0.75:9200", "http://10.20.0.76:9200"]
        manage_template => false
        index => "syslog"
        user => "logstash_writer"
        password => "*******"
    }
}
```



\pagebreak

## 日誌採集

配置發送 syslog 到 :

- macow: 10.20.0.61:514，10.20.0.62:514，10.20.0.63:514 (UDP)
- manila: 10.20.0.71:514，10.20.0.72:514，10.20.0.73:514 (UDP)

\pagebreak

# 採集效果

![採集的機器概覽](http://ww2.sinaimg.cn/large/006tNc79gy1g35lwnfaqlj31nv0u04bp.jpg)

![採集的的機器指標](/Users/leon/Library/Application Support/typora-user-images/image-20190518174138064.png)

![ssh登錄情況](http://ww2.sinaimg.cn/large/006tNc79gy1g35lzd8nhaj31gp0u0dzq.jpg)

![windows-events](/Users/leon/Library/Application Support/typora-user-images/image-20190518174342374.png)

![syslogs](http://ww1.sinaimg.cn/large/006tNc79gy1g35m16qkv0j31n70u04gu.jpg)

\pagebreak

# 郵件告警

## 添加郵件配置

```shell
PUT _cluster/settings
{
  "persistent": {
    "xpack": {
      "notification": {
        "email": {
          "account": {
            "suncity": {
              "profile": "outlook",
              "email_defaults": {
                "from": "elk@suncity-group.com"
              },
              "smtp": {
                "auth": true,
                "host": "exchange.suncity-group.com",
                "port": 587,
                "user": "elk",
                "password": "******"
              }
            }
          },
          "default_account": "suncity"
        }
      }
    }
  }
}
```

\pagebreak

## 編寫告警規則

### 防火墻、交換機、路由器、VPN 出現錯誤關鍵字

```shell
PUT _xpack/watcher/watch/syslog-errors-1
{
    "trigger" : {
      "schedule" : {
        "interval" : "1h"
      }
    },
    "input" : {
      "search" : {
        "request" : {
          "search_type" : "query_then_fetch",
          "indices" : [
            "syslog"
          ],
          "types" : [ ],
          "body" : {
            "query" : {
              "bool" : {
                "must" : [
                  {
                    "range" : {
                      "@timestamp" : {
                        "gte" : "now-1h"
                      }
                    }
                  },
                  {
                    "terms" : {
                      "message" : [
                        "emergency",
                        "alert",
                        "critical"
                      ]
                    }
                  }
                ]
              }
            },
            "size" : 500
          }
        }
      }
    },
    "condition" : {
      "compare" : {
        "ctx.payload.hits.total" : {
          "gt" : 0
        }
      }
    },
    "transform":{
      "script":"ctx.payload.hits.hits=ctx.payload.hits.hits.stream().map(i->{i.show_time=Instant.from(OffsetDateTime.parse(i._source['@timestamp'],DateTimeFormatter.ISO_DATE_TIME)).atZone(ZoneId.of('Asia/Shanghai')).format(DateTimeFormatter.ofPattern('YYYY-MM-dd HH:mm:ss'));return i;}).collect(Collectors.toList());return ctx.payload;"
    },
    "actions" : {
      "send_email" : {
        "email" : {
          "to" : [
            "nick.ng@suncity-group.com",
            "system_network_team@suncity-group.com"
          ],
          "subject" : "[ELK Alert] syslog critical alert",
          "body" : {
            "html" : """<table border="1"> <thead> <tr> <th>Time</th> <th>Message</th> </tr> </thead> <tbody> {{#ctx.payload.hits.hits}} <tr> <td>{{show_time}}</td> <td>{{_source.message}}</td> </tr> {{/ctx.payload.hits.hits}} </tbody> </table>"""
          }
        }
      }
    },
    "metadata" : {
      "name" : "firewall, switches, routers errors",
      "xpack" : {
        "type" : "json"
      }
    }
}
```



### Linux 主機出現錯誤關鍵字

```shell
PUT _xpack/watcher/watch/filebeat-system-errors-1
{
    "trigger" : {
      "schedule" : {
        "interval" : "1h"
      }
    },
    "input" : {
      "search" : {
        "request" : {
          "search_type" : "query_then_fetch",
          "indices" : [
            "filebeat-*"
          ],
          "types" : [ ],
          "body" : {
            "query" : {
              "bool" : {
                "must" : [
                  {
                    "range" : {
                      "@timestamp" : {
                        "gte" : "now-1h"
                      }
                    }
                  },
                  {
                    "term" : {
                      "fileset.name" : "syslog"
                    }
                  },
                  {
                    "terms" : {
                      "system.syslog.message" : [
                        "emergency",
                        "alert",
                        "critical"
                      ]
                    }
                  }
                ]
              }
            },
            "size" : 500
          }
        }
      }
    },
    "condition" : {
      "compare" : {
        "ctx.payload.hits.total" : {
          "gt" : 0
        }
      }
    },
    "transform":{
      "script":"ctx.payload.hits.hits=ctx.payload.hits.hits.stream().map(i->{i.show_time=Instant.from(OffsetDateTime.parse(i._source['@timestamp'],DateTimeFormatter.ISO_DATE_TIME)).atZone(ZoneId.of('Asia/Shanghai')).format(DateTimeFormatter.ofPattern('YYYY-MM-dd HH:mm:ss'));return i;}).collect(Collectors.toList());return ctx.payload;"
    },
    "actions" : {
      "send_email" : {
        "email" : {
          "to" : [
            "nick.ng@suncity-group.com",
            "system_network_team@suncity-group.com"
          ],
          "subject" : "[ELK Alert] linux critical alert",
          "body" : {
            "html" : """<table border="1"> <thead> <tr> <th>Host</th> <th>Time</th> <th>Message</th> </tr> </thead> <tbody> {{#ctx.payload.hits.hits}} <tr> <td>{{_source.beat.hostname}}</td> <td>{{show_time}}</td> <td>{{_source.system.syslog.message}}</td> </tr> {{/ctx.payload.hits.hits}} </tbody> </table>"""
          }
        }
      }
    },
    "metadata" : {
      "name" : "linux servers with critical events",
      "xpack" : {
        "type" : "json"
      }
    }
}
```



### Windows 主機出現錯誤關鍵字

```shell
PUT _xpack/watcher/watch/winlogbeat-system-errors-1
{
    "trigger" : {
      "schedule" : {
        "interval" : "1h"
      }
    },
    "input" : {
      "search" : {
        "request" : {
          "search_type" : "query_then_fetch",
          "indices" : [
            "winlogbeat-*"
          ],
          "types" : [ ],
          "body" : {
            "query" : {
              "bool" : {
                "must" : [
                  {
                    "range" : {
                      "@timestamp" : {
                        "gte" : "now-1h"
                      }
                    }
                  },
                  {
                    "term" : {
                      "level" : "Error"
                    }
                  }
                ]
              }
            },
            "size" : 500
          }
        }
      }
    },
    "condition" : {
      "compare" : {
        "ctx.payload.hits.total" : {
          "gt" : 0
        }
      }
    },
    "transform":{
      "script":"ctx.payload.hits.hits=ctx.payload.hits.hits.stream().map(i->{i.show_time=Instant.from(OffsetDateTime.parse(i._source['@timestamp'],DateTimeFormatter.ISO_DATE_TIME)).atZone(ZoneId.of('Asia/Shanghai')).format(DateTimeFormatter.ofPattern('YYYY-MM-dd HH:mm:ss'));return i;}).collect(Collectors.toList());return ctx.payload;"
    },
    "actions" : {
      "send_email" : {
        "email" : {
          "to" : [
            "nick.ng@suncity-group.com",
            "system_network_team@suncity-group.com"
          ],
          "subject" : "[ELK Alert] windows critical alert",
          "body" : {
            "html" : """<table border="1"> <thead> <tr> <th>Host</th> <th>Time</th> <th>Item</th> <th>Message</th> </tr> </thead> <tbody> {{#ctx.payload.hits.hits}} <tr> <td>{{_source.beat.hostname}}</td> <td>{{show_time}}</td> <td>{{_source.log_name}}</td> <td>{{_source.message}}</td> </tr> {{/ctx.payload.hits.hits}} </tbody> </table>"""
          }
        }
      }
    },
    "metadata" : {
      "name" : "winlog with critical events",
      "xpack" : {
        "type" : "json"
      }
    }
}
```



### 單個主機 5 分鐘內登錄失敗超過 50 次

```shell
PUT _xpack/watcher/watch/filebeat-system-login_fail_alert-2
{
    "trigger" : {
      "schedule" : {
        "interval" : "5m"
      }
    },
    "input" : {
      "search" : {
        "request" : {
          "search_type" : "query_then_fetch",
          "indices" : [
            "filebeat-*"
          ],
          "types" : [ ],
          "body" : {
            "query" : {
              "bool" : {
                "must" : [
                  {
                    "range" : {
                      "@timestamp" : {
                        "gte" : "now-{{ctx.metadata.time_period}}"
                      }
                    }
                  },
                  {
                    "term" : {
                      "system.auth.ssh.event" : "Failed"
                    }
                  }
                ]
              }
            },
            "size" : 0,
            "aggs" : {
              "terms_hosts" : {
                "terms" : {
                  "field" : "host.name",
                  "size" : 1000,
                  "min_doc_count" : "{{ctx.metadata.threshold}}"
                },
                "aggs" : {
                  "terms_ssh_ip" : {
                    "terms" : {
                      "field" : "system.auth.ssh.ip",
                      "size" : 10000,
                      "min_doc_count" : "1"
                    }
                  }
                }
              },
              "sum_count" : {
                "sum_bucket" : {
                  "buckets_path" : "terms_hosts>_count"
                }
              }
            }
          }
        }
      }
    },
    "condition" : {
      "compare" : {
        "ctx.payload.aggregations.sum_count.value" : {
          "gt" : 0
        }
      }
    },
    "actions" : {
      "send_email" : {
        "email" : {
          "to" : [
            "nick.ng@suncity-group.com",
            "system_network_team@suncity-group.com"
          ],
          "subject" : "[ELK Alert] host login fails more than {{ctx.metadata.threshold}} in {{ctx.metadata.time_period}}",
          "body" : {
            "html" : """<table border="1"> <thead> <tr> <th>host</th><th>login failed times</th> <th>ip</th></tr> </thead> <tbody> {{#ctx.payload.aggregations.terms_hosts.buckets}} <tr> <td>{{key}}</td><td>{{doc_count}}</td> <td><table><tbody>{{#terms_ssh_ip.buckets}}<tr><td>{{key}}</td> <td>{{doc_count}}</td></tr>{{/terms_ssh_ip.buckets}}</tbody></table></td> </tr> {{/ctx.payload.aggregations.terms_hosts.buckets}} </tbody> </table>"""
          }
        }
      }
    },
    "metadata" : {
      "name" : "host login fails 50 times in 5 minutes",
      "threshold" : 50,
      "xpack" : {
        "type" : "json"
      },
      "time_period" : "5m"
    }
}
```



### 單個 IP  5分鐘內登錄失敗超過 10 次

```shell
PUT _xpack/watcher/watch/filebeat-system-login_fail_alert-1
{
    "trigger" : {
      "schedule" : {
        "interval" : "5m"
      }
    },
    "input" : {
      "search" : {
        "request" : {
          "search_type" : "query_then_fetch",
          "indices" : [
            "filebeat-*"
          ],
          "types" : [ ],
          "body" : {
            "query" : {
              "bool" : {
                "must" : [
                  {
                    "range" : {
                      "@timestamp" : {
                        "gte" : "now-{{ctx.metadata.time_period}}"
                      }
                    }
                  },
                  {
                    "term" : {
                      "system.auth.ssh.event" : "Failed"
                    }
                  }
                ]
              }
            },
            "size" : 0,
            "aggs" : {
              "terms_ssh_ip" : {
                "terms" : {
                  "field" : "system.auth.ssh.ip",
                  "size" : 10000,
                  "min_doc_count" : "{{ctx.metadata.threshold}}"
                },
                "aggs" : {
                  "terms_hosts" : {
                    "terms" : {
                      "field" : "host.name",
                      "size" : 1000,
                      "min_doc_count" : "1"
                    }
                  }
                }
              },
              "sum_count" : {
                "sum_bucket" : {
                  "buckets_path" : "terms_ssh_ip>_count"
                }
              }
            }
          }
        }
      }
    },
    "condition" : {
      "compare" : {
        "ctx.payload.aggregations.sum_count.value" : {
          "gt" : 0
        }
      }
    },
    "actions" : {
      "send_email" : {
        "email" : {
          "to" : [
            "nick.ng@suncity-group.com",
            "system_network_team@suncity-group.com"
          ],
          "subject" : "[ELK Alert] login fails more than {{ctx.metadata.threshold}} in {{ctx.metadata.time_period}}",
          "body" : {
            "html" : """<table border="1"> <thead> <tr> <th>ip</th><th>login failed times</th> <th>host</th></tr> </thead> <tbody> {{#ctx.payload.aggregations.terms_ssh_ip.buckets}} <tr> <td>{{key}}</td><td>{{doc_count}}</td> <td><table><tbody>{{#terms_hosts.buckets}}<tr><td>{{key}}</td> <td>{{doc_count}}</td></tr>{{/terms_hosts.buckets}}</tbody></table></td> </tr> {{/ctx.payload.aggregations.terms_ssh_ip.buckets}} </tbody> </table>"""
          }
        }
      }
    },
    "metadata" : {
      "name" : "ip login fails 10 times in 5 minutes",
      "threshold" : 10,
      "xpack" : {
        "type" : "json"
      },
      "time_period" : "5m"
    }
}
```



### 最近 5 小時內沒有發送日誌的主機

```shell
PUT _xpack/watcher/watch/metricbeat-system-no_metrics-1
{
    "trigger" : {
      "schedule" : {
        "interval" : "5m"
      }
    },
    "input" : {
      "search" : {
        "request" : {
          "search_type" : "query_then_fetch",
          "indices" : [
            "filebeat-*"
          ],
          "types" : [ ],
          "body" : {
            "query" : {
              "range" : {
                "@timestamp" : {
                  "gte" : "now-6h"
                }
              }
            },
            "aggs" : {
              "terms_hosts" : {
                "terms" : {
                  "field" : "beat.hostname",
                  "size" : 1000
                },
                "aggs" : {
                  "max_timestamp" : {
                    "max" : {
                      "field" : "@timestamp"
                    }
                  }
                }
              }
            },
            "size" : 0
          }
        }
      }
    },
    "condition" : {
      "script" : {
        "source" : "for ( i in ctx.payload.aggregations.terms_hosts.buckets ){ if (Instant.ofEpochMilli(ctx.trigger.triggered_time.getMillis()).minusSeconds(ctx.metadata.time_in_seconds).compareTo(Instant.parse(i.max_timestamp.value_as_string)) > 0){ return true }}",
        "lang" : "painless"
      }
    },
    "transform" : {
      "script" : {
        "source" : "ctx.payload.aggregations.terms_hosts.buckets=ctx.payload.aggregations.terms_hosts.buckets.stream().filter(i -> { return Instant.ofEpochMilli(ctx.trigger.triggered_time.getMillis()).minusSeconds(ctx.metadata.time_in_seconds).compareTo(Instant.parse(i.max_timestamp.value_as_string)) > 0 }).map(i -> { i.max_timestamp.show_value=Instant.parse(i.max_timestamp.value_as_string).atZone(ZoneId.of('Asia/Shanghai')).format(DateTimeFormatter.ofPattern('YYYY-MM-dd HH:m:ss')); return i; }).collect(Collectors.toList()); return ctx.payload;",
        "lang" : "painless"
      }
    },
    "actions" : {
      "send_email" : {
        "email" : {
          "to" : [
            "nick.ng@suncity-group.com",
            "system_network_team@suncity-group.com"
          ],
          "subject" : "[ELK Alert] no metric hosts in 5 hours",
          "body" : {
            "html" : """<table border="1"> <thead> <tr> <th>host</th><th>last metric time</th></tr> </thead> <tbody> {{#ctx.payload.aggregations.terms_hosts.buckets}} <tr> <td> {{key}} </td> <td> {{max_timestamp.show_value}} </td> </tr> {{/ctx.payload.aggregations.terms_hosts.buckets}} </tbody> </table>"""
          }
        }
      }
    },
    "metadata" : {
      "time_in_seconds" : 18000,
      "name" : "no metric hosts in 5 hours",
      "xpack" : {
        "type" : "json"
      }
    }
}
```



### 告警效果

![image-20190518084413739](http://ww2.sinaimg.cn/large/006tNc79gy1g356f483i4j30ky0vidjh.jpg)

![image-20190518084436889](http://ww3.sinaimg.cn/large/006tNc79gy1g356fgsmcaj30kc0zadjk.jpg)

![image-20190518171827347](http://ww3.sinaimg.cn/large/006tNc79gy1g35la3npryj31ac0ew787.jpg)

\pagebreak
