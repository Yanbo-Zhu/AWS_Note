
https://www.bilibili.com/video/BV1Bv4y1E7i6/?spm_id_from=333.788.recommend_more_video.-1&vd_source=55e5cc2f534c16c73bbeb684e98c4195

![](../image/Pasted%20image%2020240712124442.png)


# 1 create  eks cluster 

![](../image/Pasted%20image%2020240712124549.png)



# 2 Depoly mircoserives appliucation 

![](../image/Pasted%20image%2020240712124628.png)


![](../image/Pasted%20image%2020240712124706.png)



# 3 Deploying Prometheus server through Helm 

using helm 
![](../image/Pasted%20image%2020240712124850.png)


Check the things depolyed 
![](../image/Pasted%20image%2020240712124949.png)


# 4 Understanding Prometheus Stack Components

Worker Nodes und k8s components are all monitored 

## 4.1 statefulSets
1 Two statefulSets
![](../image/Pasted%20image%2020240712125231.png)


## 4.2 Depolyment

2 there Depolyments
![](../image/Pasted%20image%2020240712125330.png)


## 4.3 ReplicaSets

3 there ReplicaSets
![](../image/Pasted%20image%2020240712125414.png)

## 4.4 DaemonSet

4   One DaemonSet
这个 DaemonSet 就是 Node Exporter DaemonSet 
![](../image/Pasted%20image%2020240712125552.png)
 
DaemonSet  runs on every Worker Node 
Node Exporter can connects to Server , translated Worker Node metrics to Prometheus metrics (like Cpu usage,  load on server 等等 )

## 4.5 Pods and Services 

5 Pods and Services 
![](../image/Pasted%20image%2020240712145322.png) 

## 4.6 Configmap

kubectl get configmap -n monitoring 

![](../image/Pasted%20image%2020240712150049.png)

You habe figurations for different parts and they are managed by operator 


## 4.7 Secrets 

kubectl get secret -n monitoring 

![](../image/Pasted%20image%2020240712150254.png)



## 4.8 CRD

Costumized resource definition 
CRD is a extension of kubenetes API, 


Once Permeuthous monitoring stack was setup, servral Costumized resource definition  were generated automatically 

命令： kubectl get crd -n monitoring 


![](../image/Pasted%20image%2020240712151127.png)




# 5 Components inside Prometheus, Altermanager, Operator 

使用 kubectl  describe 导出 一个 statefulset 和 depolyment 的描述信息 


## 5.1 statefulSet prometheus 的描述文件

![](../image/Pasted%20image%2020240712151730.png)


statefulSet prometheus-monitoring-kube-prometheus-prometheus的描述文件 

![](../image/Pasted%20image%2020240712152414.png)


包含两个 Container 

1 Container Prometheus 
![](../image/Pasted%20image%2020240712152526.png)

### 5.1.1 Mounts
where Prometheus gets its configuration data

1 
`/etc/prometheus/config_out from config-out` 对应 --config.file项 指明 what endpoint to scrape the metrics
prometheus.env.yaml 中 含有 addresses of applications ,  这些 address 中 expose the address of metrics 


2 
rulesfiles 
![](../image/Pasted%20image%2020240713124312.png)

Rules configuration file: alerting the rules, etc. 


### 5.1.2 Config-reloader 
含有 sidecar/ helper container 

![](../image/Pasted%20image%2020240713124819.png)


Config-reloader  is responsible for reloading, when configuration files changes . 
It tells prometheus, hi, your configuration change, please pick up the latest changes  and reload it

When we add new target to monitor , new changes,  config-reloader will add this new target without restart the= prometheus 

> Config-Reloader will also reload the changes to roots file 


![](../image/Pasted%20image%2020240713125231.png)

Image 被使用 


通过 args  中的参量 --config-file, Config-reloader was told which config-file to watch 

![](../image/Pasted%20image%2020240713125203.png)



查看secret 
![](../image/Pasted%20image%2020240713125628.png)



查看 configmap 
![](../image/Pasted%20image%2020240713125704.png)

![](../image/Pasted%20image%2020240713125730.png)


![](../image/Pasted%20image%2020240713130326.png)




## 5.2 statefulSet alertmanager的描述文件

statefulSet alertmanager-monitoring-kube-prometheus-alertmanager的描述文件 

![](../image/Pasted%20image%2020240713125905.png)

![](../image/Pasted%20image%2020240713125921.png)


![](../image/Pasted%20image%2020240713125939.png)



## 5.3 Depolyemnt monitoring的描述文件


Depolyemnt monitoring-kube-prometheus-operator 的描述文件 
![](../image/Pasted%20image%2020240712152140.png)

![](../image/Pasted%20image%2020240713125950.png)


The Contianer "kube-prometheus-stack" is the orchestrator of whole monitoring stack  所以这个 image 很重要 

![](../image/Pasted%20image%2020240713130121.png)

Args 中的参量 是 interative connected 的 







