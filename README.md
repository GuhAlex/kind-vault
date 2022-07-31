# Kind with Hashicorp Vault
---
## Requirements
- kind    >= v0.11.1
- helm    >= v3.2.1
- vault   >= v1.9.0
- kubectl >= v1.22.4
---
## About
Simple Kind Cluster with HashiCorp Vault to help in lab tests :grin:


### Setup :wrench:
create the cluster
```
kind create cluster --config kind-config.yaml
```
deploy ingress controller
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml
```
deploy vault with helm
```
helm repo add hashicorp https://helm.releases.hashicorp.com
helm install vault hashicorp/vault --version 0.19.0
```
set the vault ingress
```
kubectl apply -f vault-ingress.yaml
```
get the Vault tokens
```
kubectl exec -ti  vault-0 vault operator init > .tokens
```
add on hosts file
```
echo "127.0.0.1  vault.local" >> /etc/hosts
```
set VAULT_TOKEN variable
```
export VAULT_TOKEN=`grep "Initial Root Token:*" .tokens | grep -o "\bs.*"`
```
set VAULT_ADDR variable
```
export VAULT_ADDR=`sudo kubectl get ingress vault-ingress -n default  -o jsonpath='{"http://"}{.spec.rules[].host}{"\n"}'`
```

> #### Warning :warning:
> you will need to [unseal](https://www.vaultproject.io/docs/concepts/seal) the vault before using it with the tokens in .token file


---
### Optional

setup one kv secret on Vault
```
vault secrets enable kv-v2
vault kv put kv/my-secret  foo=bar
```
