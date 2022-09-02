# Fluxcd on kind cluster (using podinfo) - Deploying pods, services, and deployments

## Fork podinfo repository to personal github account
Fork the following [podinfo repo](https://github.com/stefanprodan/podinfo)  

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
$> flux bootstrap github --owner=$GITHUB_USER --repository=fluxcd-test --branch=main --path=./clusters/podinfo-flux-cluster --personal
► connecting to github.com
...
✔ all components are healthy
```

## Using flux cd to spin up podinfo resources
```
$> cd fluxcd-test
$> flux create source git podinfo --url=https://github.com/$GITHUB_USER/podinfo --branch=master --interval=15s --export > ./clusters/podinfo-flux-cluster/podinfo-source.yaml
$> flux create kustomization podinfo --source=podinfo --path="./kustomize" --prune=true --interval=15s --target-namespace flux-system --export > ./clusters/podinfo-flux-cluster/podinfo-kustomization.yaml
$> git add . && git commit -m "podinfo added"
$> git push
```

### Verify flux cd state
```
$> watch flux get all
Every 2.0s: flux get all 

NAME                            REVISION        SUSPENDED       READY   MESSAGE
gitrepository/flux-system       main/ec513c9    False           True    stored artifact for revision 'main/ec513c936a33a9c8c25600d9fc25c99bf8bf271f'
gitrepository/podinfo           master/79f8138  False           True    stored artifact for revision 'master/79f81383288bf6542fcb5bdd8144b826b33b36e7'

NAME                            REVISION        SUSPENDED       READY   MESSAGE
kustomization/flux-system       main/ec513c9    False           True    Applied revision: main/ec513c9
kustomization/podinfo           master/79f8138  False           True    Applied revision: master/79f8138
```

### Verify podinfo resources are created and running
```
$> kc get pod,services,deploy -A | grep podinfo
flux-system          pod/podinfo-66df4b59fb-2ckw2                       1/1     Running   0          23s
flux-system          pod/podinfo-66df4b59fb-p9zkc                       1/1     Running   0          38s
flux-system          service/podinfo                                    ClusterIP   10.96.65.98     <none>        9898/TCP,9999/TCP        38s
flux-system          deployment.apps/podinfo                            2/2     2            2           38s
```

