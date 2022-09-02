# Fluxcd on kind cluster (using local-test files)

Test files are located in this repo ---> https://github.com/roger-ding/local-fluxcd-test-files

## Uninstall existing flux from kind cluster for new setup
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
$> flux bootstrap github --owner=$GITHUB_USER --repository=fluxcd-test --branch=main --path=./clusters/local-test-flux-cluster --personal
► connecting to github.com
...
✔ all components are healthy
```

## Using flux cd to spin up podinfo resources
```
$> cd fluxcd-test
$> flux create source git local-test-centos --url=https://github.com/$GITHUB_USER/local-fluxcd-test-files --branch=main --interval=15s --export > ./clusters/local-test-flux-cluster/pod-deploy-test/podinfo-source.yaml
$> flux create kustomization local-test-centos --source=local-test-centos --path="./test-files" --prune=true --interval=15s --target-namespace flux-system --export > ./clusters/local-test-flux-cluster/pod-deploy-test/podinfo-kustomization.yaml
$> git add . && git commit -m "local-test-centos pod deployment info added"
$> git push
```

## Verify flux cd state
```
$> watch flux get all 
Every 2.0s: flux get all  

NAME                            REVISION        SUSPENDED       READY   MESSAGE
gitrepository/flux-system       main/ec513c9    False           True    stored artifact for revision 'main/ec513c936a33a9c8c25600d9fc25c99bf8bf271f'
gitrepository/local-test        main/20241c9    False           True    stored artifact for revision 'main/20241c9e2020b9e4c5ca48cf929d2cac30b59869'
gitrepository/local-test-centos main/20241c9    False           True    stored artifact for revision 'main/20241c9e2020b9e4c5ca48cf929d2cac30b59869'

NAME                                    REVISION        SUSPENDED       READY   MESSAGE
helmchart/flux-system-local-test        0.1.0           False           True    packaged 'local-test' chart with version '0.1.0'

NAME                    REVISION        SUSPENDED       READY   MESSAGE
helmrelease/local-test  0.1.0           False           True    Release reconciliation succeeded

NAME                            REVISION        SUSPENDED       READY   MESSAGE
kustomization/flux-system       main/ec513c9    False           True    Applied revision: main/ec513c9
kustomization/local-test-centos main/20241c9    False           True    Applied revision: main/20241c9
```

## Verify local-test resources are created and running
```
$> kc get pod,deploy,service,serviceaccount -A | grep local-test
flux-system          pod/local-test-6f5b9d9c8d-2cftc                    1/1     Running   0          4m11s
flux-system          pod/local-test-centos-58b7879cbd-gpzgs             1/1     Running   0          4m12s
flux-system          deployment.apps/local-test                         1/1     1            1           4m11s
flux-system          deployment.apps/local-test-centos                  1/1     1            1           4m12s
flux-system          service/local-test                                 ClusterIP   10.96.201.45    <none>        80/TCP                   4m11s
flux-system          serviceaccount/local-test                          1         4m11s

$> helm list -A | grep local-test
local-test	flux-system	1       	2022-09-02 16:14:00.949682942 +0000 UTC	deployed	local-test-0.1.0	1.16.0
```

