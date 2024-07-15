
https://www.bilibili.com/video/BV1Bv4y1E7i6/?spm_id_from=333.788.recommend_more_video.-1&vd_source=55e5cc2f534c16c73bbeb684e98c4195

![](image/Pasted%20image%2020240712124442.png)


# 1 create  eks cluster 

![](image/Pasted%20image%2020240712124549.png)



# 2 Depoly mircoserives appliucation 

![](image/Pasted%20image%2020240712124628.png)


![](image/Pasted%20image%2020240712124706.png)



# 3 Deploying Prometheus server 

using helm 
![](image/Pasted%20image%2020240712124850.png)


Check the things depolyed 
![](image/Pasted%20image%2020240712124949.png)


# 4 Understanding Prometheus Stack Components

Worker Nodes und k8s components are all monitored 

## 4.1 statefulSets
1 Two statefulSets
![](image/Pasted%20image%2020240712125231.png)


## 4.2 Depolyment

2 there Depolyments
![](image/Pasted%20image%2020240712125330.png)


## 4.3 ReplicaSets

3 there ReplicaSets
![](image/Pasted%20image%2020240712125414.png)

## 4.4 DaemonSet

4   One DaemonSet
这个 DaemonSet 就是 Node Exporter DaemonSet 
![](image/Pasted%20image%2020240712125552.png)
 
DaemonSet  runs on every Worker Node 
Node Exporter can connects to Server , translated Worker Node metrics to Prometheus metrics (like Cpu usage,  load on server 等等 )

## 4.5 Pods and Services 

5 Pods and Services 
![](image/Pasted%20image%2020240712145322.png) 

## 4.6 Configmap

kubectl get configmap -n monitoring 

![](image/Pasted%20image%2020240712150049.png)

You habe figurations for different parts and they are managed by operator 


## 4.7 Secrets 

kubectl get secret -n monitoring 

![](image/Pasted%20image%2020240712150254.png)



## 4.8 CRD

Costumized resource definition 
CRD is a extension of kubenetes API, 


Once Permeuthous monitoring stack was setup, servral Costumized resource definition  were generated automatically 

命令： kubectl get crd -n monitoring 


![](image/Pasted%20image%2020240712151127.png)




# 5 Components inside Prometheus, Altermanager, Operator 

使用 kubectl  describe 导出 一个 statefulset 和 depolyment 的描述信息 

![](image/Pasted%20image%2020240712151730.png)


---

statefulSet prometheus-monitoring-kube-prometheus-prometheus的描述文件 

![](image/Pasted%20image%2020240712152414.png)


包含两个 Container 

1 Container Prometheus 
![](image/Pasted%20image%2020240712152526.png)

Mounts 项： where Prometheus gets its configuration data 

--config.file项


2 
Config-reloader 



---

statefulSet alertmanager-monitoring-kube-prometheus-alertmanager的描述文件 



---


Depolyemnt monitoring-kube-prometheus-operator 的描述文件 
![](image/Pasted%20image%2020240712152140.png)



