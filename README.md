# Fluxcd on kind cluster - Setup

## Export github user and token information
```
$> export GITHUB_USER=BLAHBLAH
$> export GITHUB_TOKEN=BLAHBLAH
```

## Check connection to kubernetes kind cluster
```
$> kc get nodes
NAME                   STATUS   ROLES                  AGE   VERSION
fluxcd-control-plane   Ready    control-plane,master   25h   v1.23.4
```

## Install flux and check that it is on latest version and prechecks pass (latest version at the time was 0.33.0)
```
$> curl -s https://fluxcd.io/install.sh
$> mv install.sh flux-install.sh
$> chmod +x flux-install.sh
$> ./flux-install.sh
$> flux --version
flux version 0.33.0
$> flux check --pre
► checking prerequisites
✔ Kubernetes 1.23.4 >=1.20.6-0
✔ prerequisites checks passed
```

## Proceed to clusters directory to view cluster files and configuration of different test scenarios using flux cd
1. [local-test-flux-cluster](https://github.com/roger-ding/fluxcd-test/tree/main/clusters/local-test-flux-cluster)
2. [podinfo-flux-cluster](https://github.com/roger-ding/fluxcd-test/tree/main/clusters/podinfo-flux-cluster)
3. [podinfo-helm-flux-cluster](https://github.com/roger-ding/fluxcd-test/tree/main/clusters/podinfo-helm-flux-cluster)

## Documentation and step by step links
1. [local-test-flux-cluster how-to](https://github.com/roger-ding/fluxcd-test/blob/main/clusters/local-test-flux-cluster/local-test-flux.md)
2. [podinfo-flux-cluster how-to](https://github.com/roger-ding/fluxcd-test/blob/main/clusters/podinfo-flux-cluster/podinfo-flux.md)
3. [podinfo-helm-flux-cluster how-to](https://github.com/roger-ding/fluxcd-test/blob/main/clusters/podinfo-helm-flux-cluster/podinfo-helm-flux.md)

