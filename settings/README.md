# Resource deployments


## AKS Clusterのデプロイ

- AKS Cluster
  - Vnet & subnet
  - connect to ACR
  - cluster monitoring


## Azure MonitorによるPrometheus scaping

「ポッドの注釈を指定して Prometheus メトリックの収集を構成する]  

see: [Container insightsでPrometheusメトリックスのスクレイピングを構成する](https://docs.microsoft.com/ja-jp/azure/azure-monitor/containers/container-insights-prometheus-integration)

```shell
# on azure-monitor folder
kubectl apply -f container-azm-ms-agentconfig.yaml
```

## Contourのデプロイ

ContourのInstall方法は

- [helmによるinstall](https://projectcontour.io/getting-started/#option-2-helm)
- [helm chart](https://github.com/bitnami/charts/tree/master/bitnami/contour)
  - [参考: ArtifactHub](https://artifacthub.io/packages/helm/bitnami/contour)

を参照

```shell
# on settings/contour folder
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm upgrade --install ingress-contour bitnami/contour \
  --namespace ingress-system --create-namespace \
  --values config.yaml
```

## Applicationのデプロイ


### アプリケーション

```shell
# on settings folder
kubectl apply -f namespace.yaml

# on settings/workloads folder
export APPINSIGHTS_JAVA_AGENT_VER=3.2.10
export BASE64_INSTRUMENTATION_STRING=`echo -n ${AI_INSTRUMENTATION_STRING} | base64`
cat app-insights-secret.yaml | envsubst | kubectl apply -f -
cat helloworld.yaml | envsubst | kubectl apply -f -
cat timefeed.yaml | envsubst | kubectl apply -f -
cat aggregator.yaml | envsubst | kubectl apply -f -
```

### Routing by ingress

```shell
# on settings/routings folder
cat ingress.yaml | envsubst | kubectl apply -f -

# delete ingress
cat ingress.yaml | envsubst | kubectl delete -f -
```

### Routing by HTTProxy

```shell
# on settings/routings folder
cat http-proxy.yaml | envsubst | kubectl apply -f -

# delete ingress
cat http-proxy.yaml | envsubst | kubectl delete -f -
```