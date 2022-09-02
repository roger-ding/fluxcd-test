# Fluxcd on kind cluster (using podinfo) - Deploying with Helm

## Fork podinfo repository to personal github account
Fork the following [podinfo repo](https://github.com/stefanprodan/podinfo)  

## Uninstall flux from kind cluster
```
$> flux uninstall
Are you sure you want to delete Flux and its custom resource definitions: y
► deleting components in flux-system namespace
...
✔ Namespace/flux-system deleted
✔ uninstall finished
```

## Bootstrap flux cd repository in personal github account
```
$> flux bootstrap github --owner=$GITHUB_USER --repository=fluxcd-test --branch=main --path=./clusters/podinfo-helm-flux-cluster --personal
► connecting to github.com
...
✔ all components are healthy
```

## Using flux cd to spin up podinfo resources
```
$> cd fluxcd-test
$> flux create source git podinfo --url=https://github.com/$GITHUB_USER/podinfo --branch=master --interval=15s --ignore-paths "# exclude all,/*,# include charts directory,\!/charts/" --export > ./clusters/podinfo-helm-flux-cluster/podinfo-helm-source.yaml
$> flux create helmrelease podinfo --chart ./charts/podinfo --source GitRepository/podinfo --namespace flux-system --target-namespace flux-system --chart-interval 15s --interval 15s --export > ./clusters/podinfo-helm-flux-cluster/podinfo-helm-release.yaml
$> git add . && git commit -m "helm podinfo added"
$> git push
```

### Verify flux cd state
```
$> watch flux get all 
Every 2.0s: flux get all 

NAME                            REVISION        SUSPENDED       READY   MESSAGE
gitrepository/flux-system       main/58d1dd7    False           True    stored artifact for revision 'main/58d1dd7375524edf2b02bbf7e6fb72ec18d10261'
gitrepository/podinfo           master/79f8138  False           True    stored artifact for revision 'master/79f81383288bf6542fcb5bdd8144b826b33b36e7'

NAME                            REVISION        SUSPENDED       READY   MESSAGE
helmchart/flux-system-podinfo   6.2.0           False           True    packaged 'podinfo' chart with version '6.2.0'

NAME                    REVISION        SUSPENDED       READY   MESSAGE
helmrelease/podinfo     6.2.0           False           True    Release reconciliation succeeded

NAME                            REVISION        SUSPENDED       READY   MESSAGE
kustomization/flux-system       main/58d1dd7    False           True    Applied revision: main/58d1dd7
```

### Verify podinfo resources are created and running
```
$> helm list -A
NAME               	NAMESPACE  	REVISION	UPDATED                                	STATUS  	CHART        	APP VERSION
flux-system-podinfo	flux-system	1       	2022-09-02 17:54:19.329777941 +0000 UTC	deployed	podinfo-6.2.0	6.2.0

$> kc get pod,deploy,service -A | grep podinfo
flux-system          pod/flux-system-podinfo-6c8bbc9c6b-t46k7           1/1     Running   0          38s
flux-system          deployment.apps/flux-system-podinfo                1/1     1            1           38s
flux-system          service/flux-system-podinfo                        ClusterIP   10.96.135.152   <none>        9898/TCP,9999/TCP        38s
```

