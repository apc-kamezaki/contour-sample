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

## Daprのインストール

### Dapr本体

```shell
# on settings/dapr folder
helm repo add dapr https://dapr.github.io/helm-charts/
helm repo update
helm upgrade --install dapr dapr/dapr \
  --namespace dapr-system --create-namespace \
  --values dapr-config.yaml
# check after install
kubectl get pods --namespace dapr-system
```


### Open Telemetry Collectorのインストール

[手順 Using OpenTelemetry Collector to collect traces to send to AppInsights](https://docs.dapr.io/operations/monitoring/tracing/open-telemetry-collector-appinsights/)


- 手順書にある open-telemetry-collector-appinsights.yaml と collector-config.yaml をダウンロード。
- ２つのYAMLのnamespaceを `dapr-system` に変更する。
- open-telemetry-collector-appinsights.yaml中の `INSTRUMENTATION-KEY` を `${AI_INSTRUMENTATION_KEY}` に変更。
- collector-config.yaml と同じ内容で `web`, `ingress-system` namespace 用のものを追加する。

インストール方法は以下の通り。

```shell
# on settings/dapr folder
cat open-telemetry-collector-appinsights.yaml | envsubst | kubectl apply -f -
kubectl apply -f collector-config.yaml
```

## Contourのインストール

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




## Cert-managerのインストール

cert-managerのインストール方法は[こちら](https://cert-manager.io/docs/installation/helm/)。

cert-manager本体のインストール

```shell
export CERT_MANAGER_VERSION="v1.8.0"
helm repo add jetstack https://charts.jetstack.io
helm repo update
helm upgrade --install cert-manager jetstack/cert-manager \
  --namespace cert-manager --create-namespace \
  --version ${CERT_MANAGER_VERSION} \
  --values config.yaml
```

cluster issuerのデプロイ

```
cat letsencrypt.yaml | envsubst | kubectl apply -f -
# wait for the ClusterIssuer to be ready
kubectl get clusterissuer -n cert-manager letsencrypt-prod
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
cat aggregator-v2.yaml | envsubst | kubectl apply -f -
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
kubectl apply -f contour-ingress-headless.yaml

# deploy http-proxy
cat http-proxy.yaml | envsubst | kubectl apply -f -

# delete ingress
cat http-proxy.yaml | envsubst | kubectl delete -f -
```
