# kuma

1. kuma install
```shell
helm repo add kuma https://kumahq.github.io/charts
helm repo update
helm install --create-namespace --namespace kuma-system kuma kuma/kuma
```
or
```shell
helm install kuma ./kuma
```

2. kuma demo install
```shell
k apply -f demo.yaml
```
3. kuma는 다양한 활용성이 있음 (https://kuma.io/features/)

Security :
- mesh/multi mesh 
- mTLS : Kuma에서 Mutual TLS(mTLS)는 서비스 간 통신을 암호화하고 인증을 보장하기 위해 사용되는 중요한 보안 기능입니다. mTLS는 양쪽(클라이언트와 서버) 모두에서 TLS 인증서를 사용하여 상호 인증을 수행합니다. 

Ingress Traffic:
- delegated gw
- built-in gw
- kubernetes gateway api 

Traffic control:
- mesh http route
- mesh tcp route 

Observability:
- service map 


4. If you enable mTLS without a MeshTrafficPermission policy, all traffic between your applications will be blocked.
```shell
k apply -f ./mesh/meshTrafficPermission.yaml
```
5. We can enable Mutual TLS with a builtin CA backend
mTLS 적용하면 모든 트래픽이 차단됨
그래서 4번 과정을 통해 허용해줘야됨
```shell
k apply -f ./mesh/mesh.yaml
```

```yaml
apiVersion: kuma.io/v1alpha1
kind: Mesh
metadata:
  name: default
spec:
  mtls:
    enabledBackend: ca-1
    backends:
      - name: ca-1
        type: builtin
```

6. We can then restrict the traffic by default by executing the following command:
```shell
k apply -f ./mesh/meshTrafficPermission_deny.yaml
```
7. validate & test the demo app (doesn't work)
```shell
kubectl port-forward svc/demo-app -n kuma-demo 5000:5000
```
8. aloow traffic with meshTrafficPermission_meshService.yaml
```shell
k apply -f ./mesh/meshTrafficPermission_meshService.yaml
```

various strategy for traffic routing

---

# gateway-api


![img.png](../kuma/img.png)


1. Kubernetes doesn’t include Gateway API CRDs -> apigateway crd 생성 (https://gateway-api.sigs.k8s.io/guides/#install-standard-channel)
```shell
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.1.0/standard-install.yaml
```

2. Gateway API CRDs 설치 확인
   istio, kuma 등 controller 설치하면 gatewayclass가 자동으로 생성됨
```shell
k get GatewayClasses  -A       
NAME    CONTROLLER                    ACCEPTED   AGE
istio   istio.io/gateway-controller   True       50m
kuma    gateways.kuma.io/controller   True       50m
```
3. kuma-gw install
```shell
helm install kuma-gw ./kuma-gw 
```

4. kuma-gw가 설치되면 자동으로 pod, service 생성됨
```shell
k get pods -n kuma-demo                
NAME                                    READY   STATUS             RESTARTS        AGE
kuma-6c45b667b4-w6ttw                   1/1     Running            0               93m
```

```shell
k get service -n kuma-demo             
NAME                   TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)        AGE
kuma                   LoadBalancer   10.102.106.203   34.64.63.247   80:30267/TCP   92m
```

5. gateway, httproute 랑 target service의 namespace가 다르면 referenceGrant 적용해야됨(./kuma-gw/templates/referenceGrant.yaml)

6. kuma에 curl로 확인
```shell
curl 34.64.63.247   
{"Hello":"World"}%                                                            
```

