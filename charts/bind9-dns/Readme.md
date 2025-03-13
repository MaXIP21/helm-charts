# Helm chart of Bind9 DNS server with Webmin

```
helm repo add peterb https://maxip21.github.io/helm-charts
```

## Deploy chart 

```
helm install -f values.yaml bind9-dns ./bind9-dns/
```

## Delete chart

```
helm delete bind9-dns
```