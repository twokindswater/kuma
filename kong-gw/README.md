# kuma kong delegated gateway

https://kuma.io/docs/2.7.x/guides/gateway-delegated/


1. install kong gateway (https://docs.konghq.com/kubernetes-ingress-controller/latest/get-started/)
```shell
# Install the Gateway API CRDs before installing Kong Ingress Controller.
kubectl apply -f https://github.com/kubernetes-sigs/gateway-api/releases/download/v1.1.0/standard-install.yaml

customresourcedefinition.apiextensions.k8s.io/gatewayclasses.gateway.networking.k8s.io configured
customresourcedefinition.apiextensions.k8s.io/gateways.gateway.networking.k8s.io configured
customresourcedefinition.apiextensions.k8s.io/grpcroutes.gateway.networking.k8s.io created
customresourcedefinition.apiextensions.k8s.io/httproutes.gateway.networking.k8s.io configured
customresourcedefinition.apiextensions.k8s.io/referencegrants.gateway.networking.k8s.io configured
```

```shell
# Create a Gateway and GatewayClass instance to use.

echo "
---
apiVersion: gateway.networking.k8s.io/v1
kind: GatewayClass
metadata:
  name: kong
  annotations:
    konghq.com/gatewayclass-unmanaged: 'true'

spec:
  controllerName: konghq.com/kic-gateway-controller
---
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: kong
spec:
  gatewayClassName: kong
  listeners:
  - name: proxy
    port: 80
    protocol: HTTP
" | kubectl apply -f -
gatewayclass.gateway.networking.k8s.io/kong created
gateway.gateway.networking.k8s.io/kong created
```

2. install kong with helm 
```shell
# Add the Kong Helm charts:
helm repo add kong https://charts.konghq.com
helm repo update

helm pull kong/ingress --untar

# Install Kong Ingress Controller and Kong Gateway with Helm:
helm install kong ./ingress -n kong --create-namespace 
```

3. add kuma injection annotation in kong namespace 
```shell
kubectl label namespace kong kuma.io/sidecar-injection=enabled
```

4. Restart both the controller and the gateway to leverage sidecar injection
```shell
kubectl rollout restart -n kong deployment kong-gateway kong-controller
deployment.apps/kong-gateway restarted
deployment.apps/kong-controller restarted
```

5. get the kong-gateway service ip 
```shell
kubectl get svc --namespace kong kong-gateway-proxy -o jsonpath='{.status.loadBalancer.ingress[0].ip}'
34.64.211.53
```

6. test the kong-gateway service 
```shell
curl 34.64.211.53 -i             
HTTP/1.1 404 Not Found
Date: Tue, 04 Jun 2024 09:28:41 GMT
Content-Type: application/json; charset=utf-8
Connection: keep-alive
Content-Length: 103
X-Kong-Response-Latency: 0
Server: kong/3.6.1
X-Kong-Request-Id: d90ff4490a2e01755f25ef6923f27327

{
  "message":"no Route matched with those values",
  "request_id":"d90ff4490a2e01755f25ef6923f27327"
}%                                                      
```

7. patch our gateway to allow all routes
```shell 
kubectl patch --type=json gateways.gateway.networking.k8s.io kong -p='[{"op":"replace","path": "/spec/listeners/0/allowedRoutes/namespaces/from","value":"All"}]'
```

8. apply httpRoute 
```shell
kubectl apply -f httpRoute.yaml
```

9. if traffic not permitted 
```shell
k apply -f meshTrafficPolicy.yaml
```

--- 
tls 적용하기

1. create secret 
```shell
kubectl create secret tls kong-cluster-cert -n kong --key ~/.ssh/selfsign/private.key --cert ~/.ssh/selfsign/selfsigned.crt
```

2. gatway에 tls 적용하기 
```shell
kubectl apply -f gateway-tls.yaml
```

