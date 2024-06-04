# kuma

1. kuma install
```shell
helm repo add kuma https://kumahq.github.io/charts
helm repo update
helm install --create-namespace --namespace kuma-system kuma kuma/kuma

```
2. kuma demo install
```shell
k apply -f demo.yaml
```
3. Kuma에서 Mutual TLS(mTLS)는 서비스 간 통신을 암호화하고 인증을 보장하기 위해 사용되는 중요한 보안 기능입니다. mTLS는 양쪽(클라이언트와 서버) 모두에서 TLS 인증서를 사용하여 상호 인증을 수행합니다. 
4. If you enable mTLS without a MeshTrafficPermission policy, all traffic between your applications will be blocked.

```shell
k apply -f meshTrafficPermission.yaml
```
5. We can enable Mutual TLS with a builtin CA backend
```shell
k apply -f mesh.yaml
```
6. We can then restrict the traffic by default by executing the following command:
```shell
k apply -f meshTrafficPermission_deny.yaml
```
7. validate & test the demo app (doesn't work)
```shell
kubectl port-forward svc/demo-app -n kuma-demo 5000:5000
```
8. aloow traffic with meshTrafficPermission_meshService.yaml
```shell
k apply -f meshTrafficPermission_meshService.yaml
```

various strategy for traffic routing

