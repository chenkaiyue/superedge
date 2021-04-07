# 应用按NodeUnit灰度
DeploymentGrid和StatefulSetGrid均支持按照NodeUnit进行灰度

## 重要字段
和灰度功能相关的字段有这些：

autoDeleteUnusedTemplate，templatePool，templates，defaultTemplateName

templatePool：用于灰度的template集合

templates：NodeUnit和其使用的templatePool中的template的映射关系，如果没有指定，NodeUnit使用defaultTemplateName指定的template

defaultTemplateName：默认使用的template，如果不填写或者使用"default"就采用spec.template

autoDeleteUnusedTemplate：默认为false，如果设置为ture，会自动删除templatePool中既不在templates中也不在spec.template中的template模板


## 操作步骤

### 确定ServiceGroup唯一标识

这一步是逻辑规划，不需要做任何实际操作。我们将目前要创建的serviceGroup逻辑标记使用的UniqKey为：`zone`

### 将边缘节点分组

这一步需要使用kubectl对边缘节点打label

gridUniqKey字段设置为了zone，所以我们在将节点分组时采用label的key为zone，如果有三个节点(分别叫node0,node1,node2)，分别为他们添加zone: zone0, zone: zone1, zone: zone2的label即可

注意：上一步中，label的key与ServiceGroup的UniqKey一致，value是NodeUnit的唯一key，value相同的节点表示属于同一个NodeUnit

如果同一个集群中有多个ServiceGroup请为每一个ServiceGroup分配不同的UniqKey

### 部署具有灰度字段的wordload
使用不同的template创建workload，如创建当前目录下的canary_deploymentgrid.yaml
`kubectl apply -f canary_deploymentgrid.yaml`

## 验证
这个例子中，NodeUnit zone1将会使用test1 template，NodeUnit zone2将会使用test2 template，其余NodeUnit将会使用defaultTemplateName中指定的template，这里会使用test1
