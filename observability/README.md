# Observability

Local POC observability stack for Istio:

- Prometheus scrapes Istio metrics.
- Grafana uses Prometheus as the default datasource.
- Kiali reads Prometheus and links to Grafana.

## Install

```sh
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo add kiali https://kiali.org/helm-charts
helm repo update

helm upgrade --install prometheus prometheus-community/prometheus \
  --version 29.24.0 \
  -n istio-system \
  -f observability/helm-values/prometheus-values.yaml \
  --wait

helm upgrade --install grafana grafana/grafana \
  --version 10.5.15 \
  -n istio-system \
  -f observability/helm-values/grafana-values.yaml \
  --wait

helm upgrade --install kiali kiali/kiali-server \
  --version 2.30.0 \
  -n istio-system \
  -f observability/helm-values/kiali-values.yaml \
  --wait
```

## Access

```sh
kubectl -n istio-system port-forward svc/kiali 20001:20001
kubectl -n istio-system port-forward svc/grafana 3000:80
kubectl -n istio-system port-forward svc/prometheus-server 9090:80
```

- Kiali: http://localhost:20001
- Grafana: http://localhost:3000
- Prometheus: http://localhost:9090

Grafana login for this local POC is `admin` / `admin`.

## Validate

```sh
kubectl -n istio-system get deploy,svc | grep -E 'prometheus|grafana|kiali'
kubectl -n istio-system rollout status deploy/prometheus-server
kubectl -n istio-system rollout status deploy/grafana
kubectl -n istio-system rollout status deploy/kiali
kubectl -n istio-system port-forward svc/prometheus-server 9090:80
curl -s 'http://localhost:9090/api/v1/query?query=istio_requests_total'
```

For request-rate charts, use a window of at least `1m`. This POC config sets Prometheus `scrape_interval` to `15s`, so `rate(istio_requests_total[1m])` has enough samples after traffic is generated.

## Uninstall

```sh
helm uninstall kiali -n istio-system
helm uninstall grafana -n istio-system
helm uninstall prometheus -n istio-system
```
