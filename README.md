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

