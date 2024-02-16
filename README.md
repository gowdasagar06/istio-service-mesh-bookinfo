# istio-service-mesh-bookinfo

#To download Istio version
curl -L https://istio.io/downloadIstio | sh -
cd istio-1.20.3
export PATH=$PWD/bin:$PATH
cd ..
kubectl create namespace istio-system
#add repo to helm
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
helm install istio-base istio/base -n istio-system --set defaultRevision=default
helm ls -n istio-system
#based on the version you installed, replace the version below

helm install -n istio-system istiod \
istio-1.20.3/manifests/charts/istio-control/istio-discovery \
--set global.hub="docker.io/istio" --set global.tag="1.20.3"

helm install -n istio-system istio-ingress \
istio-1.20.3/manifests/charts/gateways/istio-ingress \
--set global.hub="docker.io/istio" \
--set global.tag="1.20.3" \
--set gateways.istio-ingressgateway.serviceAnnotations."service\.beta\.kubernetes\.io/aws-load-balancer-proxy-protocol"="*" \
--set gateways.istio-ingressgateway.serviceAnnotations."service\.beta\.kubernetes\.io/aws-load-balancer-connection-idle-timeout"="60" \
--set gateways.istio-ingressgateway.serviceAnnotations."service\.beta\.kubernetes\.io/aws-load-balancer-cross-zone-load-balancing-enabled"="true" \
--set gateways.istio-ingressgateway.serviceAnnotations."service\.beta\.kubernetes\.io/aws-load-balancer-type"="nlb"

kubectl get all -n istio-system

kubectl apply -f istio-1.20.3/samples/addons/

kubectl label namespace default istio-injection=enabled

git clone https://github.com/gowdasagar06/istio-service-mesh-bookinfo.git
cd istio-service-mesh-bookinfo

kubectl apply -f bookinfo.yaml

kubectl exec "$(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}')" -c ratings -- curl -sS productpage:9080/productpage | grep -o "<title>.*</title>"

kubectl patch svc kiali -n istio-system -p '{"spec": {"type": "LoadBalancer"}}'

http://loadbalancer_url:20001

#get ingress url and access bookinfo application

http://loadbalancer_url/productpage
