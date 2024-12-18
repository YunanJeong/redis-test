# redis-test

redis-test

## Requirement

- K3s
- Helm

## install

```sh
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
helm install my-redis bitnami/redis --version 20.6.0 -f custom.yaml
helm upgrade my-redis bitnami/redis --version 20.6.0 -f custom.yaml
```
