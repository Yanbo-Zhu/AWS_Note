
# 1 Route53里面的设置 

Es gibt eine Route53 - Zone "ivu.de" am EKS VPC, die wir selbst verwalten. Dadurch nutzt Route53 für ivu.de nicht die Auflösung per Internet, sondern per lokaler HostedZone. 


aktuell enthält die Route53 Zone "ivu.de" folgende Einträge

```
    "ivu.de" = {
      "nexus3"           = "192.168.82.60"
      "dockerhub.nexus3" = "192.168.82.60"
      "jfrog"            = "172.27.4.10"
    }
```


Das packen wir in die ConfigMap für coredns. So sieht die im Moment aus: 

```
Name:         coredns  
Namespace:    kube-system  
Labels:       app.kubernetes.io/instance=coredns  
              eks.amazonaws.com/component=coredns  
              k8s-app=kube-dns  
Annotations:  reloader.stakater.com/auto: true

Data  
====  
Corefile:  
----  
.:53 {  
    errors  
    health  
    ready  
    kubernetes cluster.local in-addr.arpa ip6.arpa {  
      pods insecure  
      fallthrough in-addr.arpa ip6.arpa  
    }  
    prometheus :9153  
    forward . /etc/resolv.conf  
    cache 30  
    loop  
    reload  
    loadbalance  
    hosts {  
      172.27.4.10 jfrog.ivu.de   # 加入这一句   
      10.100.7.249 git.ivu-ag.com  
      10.100.0.238 pad-dev1.ivu-ag.com  
      fallthrough  
    }  
}  


# 删除掉下面这些
# ivu-cloud.local resolution via AWS DNS
ivu-cloud.local:53 {  
    # dc07, dc08  
    forward . 172.100.0.50 172.100.1.80**  
    cache 30  
}
# 删除到这里结束, 下面的还要保留 

BinaryData  
====

Events:  <none>
```


要去对上面的 configMap 做什么操作
Kannst Du mal probieren, das rot markierte rauszunehmen und das grün markierte hinzuzufügen, und mir dann nochmal Bescheid geben?
Ich weiß nicht ob diese ConfigMap bereits per ArgoCD verwaltet wird. Falls ja, müsstest Du das dort ändern. 


通过上面的操作要达到的什么目的? 
Ich will die "ivu.de" hosted zone verschwinden lassen, damit sie per Default im Internet aufgelöst wird und softwarecenter.ivu.de gefunden wird. 
Aber dann brauchen wir die aktuellen Einträge dieser Zone, die nicht im Internet aufgelöst werden können, in CoreDNS. 
Wenn Du jfrog.ivu.de da hinzugefügt hast, entferne ich ivu.de als hosted zone (通过 route53  Web UI 就可以删除hostzone ). 



Die Route53 hosted zone "ivu.de" hängt direkt am EKS VPC aktuell. Die brauchen wir aber im Dev Cluster nicht mehr. 

----

Das sollte man m.E. mit verschiedenen Subdomains lösen. 导致了: 
- jfrog unter ivu.de kan nur lokal aufgelöst werden können. 
- softwarecenter unter ivu.de  kann sowahl im Internet als auch lokal  aufgelöst werden können. 

----

CoreDNS 的 configMap 中的

```
# ivu-cloud.local resolution via AWS DNS
ivu-cloud.local:53 {  
    # dc07, dc08  
    forward . 172.100.0.50 172.100.1.80**  
    cache 30  
}
```

上面这个的意义是  将 Record vom DNS Server der IVU-AG Domain 搬运到  IVU-Cloud-internen Eks Cluster 中 
aktuell holen sich die IVU-AG-internen Cluster den Record vom DNS Server der IVU-AG Domain. Wenn wir ihn hart in Core DNS eintragen, sollte weiter alles funktionieren - bis sich die IP mal ändert. 



