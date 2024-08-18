---
title: k8s-api-access-control
layout: default
parent: k8s
has_children: false
---

# 1. 访问控制流程

![access-control](/assets/images/k8s/access-control.png)

访问控制流程整体分三步走：**身份认证-->鉴权-->准入控制**

## 2. 身份认证

- 普通身份：由集群的证书机构签名的合法证书的用户。（下面介绍两种通过证书签发获取的方式）  
- 管理身份：服务账号与一组以 Secret 保存的凭据相关  
- 匿名身份：匿名访问默认情况下是被禁用的，可以通过--anonymous-auth=true来启用。  


### 2.1 私钥审批认证获取证书

{: .note }
key:通常指私钥 csr:证书签名申请，这不是证书 crt:证书  

通过openssl创建私钥

```shell
openssl genrsa -out user01.key 2048
openssl req -new -key user01.key -out user01.csr -subj "/CN=user01"
```

通过创建CertificateSigningRequest将私钥提交到集群
这里的request是证书的base64加密通过`cat user01.csr | base64 | tr -d "\n"`获得

```shell
cat <<EOF | kubectl apply -f -
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: user01
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0diWGwxYzJWeU1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQTdkRURyNThSZzlyanA5MEErdi83YjN1dWdmYTZRTldpTTJvMWw2ZXdJbnhJCkt0OEpQTkwwcERIcmhEam9TVUNHMUlrTS96M0JCTVBhVTQrbGhNTTRKQUFQUVZGek1XVVlLTSsrMCtZT00wNGwKY2dIaWlHVG9nUk5CckFoaUR1YUtKOUhDSzVyK3cvRUZKQ2o2MlpBOElRREZyUm41dzJKeVUxcVYwellXL05FYQpMeDlSbDZjOStUYXlHTG92VnVuWlVJekNDVmxWNUU4MnYyMFJJSGpJeFkvSjhvK2dEMG9UbUlIc0h6SEI2ZHQ1CkM3c0x1NVA0VG05OXNlKzREWmRtNTNRaTU4LzdlektiSXZteW41R2YxWTljRDlQdWloblZXcWtST1FleTZhZFYKMGhrZXV5QW1MYUNIcWhvMHNid2pMZmM4VjBkQnVyK1ozSzM3d1M3cDF3SURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBRW9GaElLRWdycDQrNjhyNWhDNjVJSStRdWs3UkErWGtaOEM5cVBoNGtPUGNiWGQ3SGdwCnM0RUU0ZURnZ3FtUTRSb0tVYi91M3NtdU80aTZHWC9qNDQ5V0c5Ni9TelpQRWJMVEFsTU5OVHVQZEUyRDRoei8KM0JDNzB1eGduMy9SQ0Z3SUdURTFSN04xaDIyUndNNWhRZWtNZEprNzBmYTZZSnNwV3dkR3FTNXVRaDVzQTJPTQpoZDJzSkFUU2dnQlJTWk5FWmxlQzdNOThHSFdSZlhSVE9SdmhZdTZMS0hGZVpjVGgvRFQzNWtKZmFUbWFOR01xCjJyK0NZMXAwQzJlbFZpT2hhUVdFMjVJNG40a2h3aDJrMjBCUldUUDVwaUwrenRLdXVKSjhMbW0ySS9xRXg3N1gKM29Kbys3UHpnU0Q4aGZYaFFrQ3pmYU1RamlVRTlheUN5dEk9Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  signerName: kubernetes.io/kube-apiserver-client
  expirationSeconds: 864000  # ten day
  usages:
  - client auth
EOF

# 查看集群的证书，这个时候的证书是Pending状态
kubectl  get csr
# 批准证书，批准以后证书变为Approved,Issued
kubectl certificate approve user01
# 获取证书，到这里我们就完成了身份的认证得到身份user01
kubectl get csr user01 -o jsonpath='{.status.certificate}'| base64 -d > user01.crt
```

### 2.2 通过集群证书签发私钥得到证书


整体流程是相似的只是在签发的时候有一点不一样

{: .important }
这里的/home/lee/kind/pki需要根据自己的具体目录指定

```shell
# 创建私钥 
openssl genrsa -out user02.key 2048
# 生成csr文件
openssl req -new -key user02.key -out user02.csr -subj "/CN=user02/"

# 使用创建集群的ca文件给csr签发证书文件得到crt文件
# 这里的/home/lee/k8s/kind/pki需要指定为自己的目录
openssl x509 -req -in user02.csr -CA /home/lee/k8s/kind/pki/ca.crt -CAkey /home/lee/k8s/kind/pki/ca.key -CAcreateserial -out user02.crt -days 365
# 这样就得到了用户user02
```


## 3. rbac (Role-Based Access Control)

使用上面创建的证书来访问集群资源

```shell
# 创建角色，制定可以访问的资源和可以执行的操作
kubectl create role role01 --verb=create --verb=get --verb=list --verb=update --verb=delete --resource=pods
# 绑定角色和权限
kubectl create rolebinding role01-binding-user01 --role=role01 --user=user01
# 设置客户端证书
kubectl config set-credentials user01 --client-key=k8s.key --client-certificate=user01.crt --embed-certs=true
#设置上下文（注意这里的--cluster 需要查看自己的集群是啥名字）
# 可以通过`kubectl config view`查看自己的集群名字
kubectl config set-context user01 --cluster=mycluster --user=user01
kubectl config use-context user01
```

