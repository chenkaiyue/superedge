# 多地域部署无状态应用

## 操作步骤

以在边缘部署echo-service为例，我们希望在多个节点组内分别部署echo-service服务，需要做如下事情：

### 确定ServiceGroup唯一标识

这一步是逻辑规划，不需要做任何实际操作。我们将目前要创建的serviceGroup逻辑标记使用的UniqKey为：`zone`

### 将边缘节点分组

这一步需要使用kubectl对边缘节点打label

gridUniqKey字段设置为了zone，所以我们在将节点分组时采用label的key为zone，如果有三个节点(分别叫node0,node1,node2)，分别为他们添加zone: zone-0, zone: zone-1, zone: zone-2的label即可

注意：上一步中，label的key与ServiceGroup的UniqKey一致，value是NodeUnit的唯一key，value相同的节点表示属于同一个NodeUnit

如果同一个集群中有多个ServiceGroup请为每一个ServiceGroup分配不同的UniqKey

### 部署DeploymentGrid
参考本目录下的deploymentgrid.yaml
`kubectl apply -f deploymentgrid.yaml`

## 验证
这时，每组节点内都有了echo-service的Deployment和对应的pod
```
[~]# kubectl get dg
NAME                  AGE
deploymentgrid-demo   85s

[~]# kubectl get deploy
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deploymentgrid-demo-zone-0   2/2     2            2           85s
deploymentgrid-demo-zone-1   2/2     2            2           85s
deploymentgrid-demo-zone-2   2/2     2            2           85s
```

# 多地域部署无状态应用+流量区域自治能力

## 操作步骤
除了上面的操作步骤外，还需要部署本目录下的servicegrid.yaml
`kubectl apply -f servicegrid.yaml`

## 验证
这时，每组节点内都有了echo-service的Deployment和对应的pod，在节点内访问统一的service-name也只会将请求发向本组的节点

```
[~]# kubectl get dg
NAME                  AGE
deploymentgrid-demo   85s

[~]# kubectl get deploy
NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deploymentgrid-demo-zone-0   2/2     2            2           85s
deploymentgrid-demo-zone-1   2/2     2            2           85s
deploymentgrid-demo-zone-2   2/2     2            2           85s

[~]# kubectl get svc
NAME                   TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes             ClusterIP   172.19.0.1     <none>        443/TCP   87m
servicegrid-demo-svc   ClusterIP   172.19.0.177   <none>        80/TCP    80s

# execute on zone-0 nodeunit
[~]# curl 172.19.0.177|grep "node name"
        node name:      node0
...
# execute on zone-1 nodeunit
[~]# curl 172.19.0.177|grep "node name"
        node name:      node1
...
# execute on zone-2 nodeunit
[~]# curl 172.19.0.177|grep "node name"
        node name:      node2
...
```