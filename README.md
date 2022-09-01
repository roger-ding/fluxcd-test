# Fluxcd on kind cluster (using podinfo)

## Fork podinfo repository to personal github account
Fork the following [podinfo repo](https://github.com/stefanprodan/podinfo)  
* $> export GITHUB_USER=<REDACTED>
* $> export GITHUB_TOKEN=<REDACTED>
* $> git clone https://github.com/$GITHUB_USER/podinfo.git
* $> cd podinfo/kustomize

### Edit the following file by adding "namespace: default" to metadata:
1. deployment.yaml
2. service.yaml
3. hpa.yaml

* $> git commit -am "add default namespace"
* $> git push

## Check connection to kubernetes kind cluster
* $> kc get nodes
```
NAME                   STATUS   ROLES                  AGE   VERSION
fluxcd-control-plane   Ready    control-plane,master   25h   v1.23.4
```

## Install flux and check that it is on latest version and prechecks pass
* $> curl -s https://fluxcd.io/install.sh
* $> mv install.sh flux-install.sh
* $> chmod +x flux-install.sh
* $> ./flux-install.sh
* $> flux --version
```
flux version 0.33.0
```

* $> flux check --pre
```
► checking prerequisites
✔ Kubernetes 1.23.4 >=1.20.6-0
✔ prerequisites checks passed
```

## Bootstrap flux cd repository in personal github account
* $> flux bootstrap github --owner=$GITHUB_USER --repository=fluxcd-test --branch=main --path=./clusters/podinfo-flux-cluster --personal
```
► connecting to github.com
✔ repository "https://github.com/roger-ding/fluxcd-test" created
► cloning branch "main" from Git repository "https://github.com/roger-ding/fluxcd-test.git"
...
✔ all components are healthy
```

## Using flux cd to spin up podinfo resources
* $> git clone https://github.com/$GITHUB_USER/fluxcd-test.git
* $> cd fluxcd-test
* $> flux create source git podinfo --url=https://github.com/$GITHUB_USER/podinfo --branch=master --interval=30s --export > ./clusters/podinfo-flux-cluster/podinfo-source.yaml
* $> flux create kustomization podinfo --source=podinfo --path="./kustomize" --prune=true --interval=5m --export > ./clusters/podinfo-flux-cluster/podinfo-kustomization.yaml
* $> git add . && git commit -m "podinfo added"
* $> git push

### Verify flux cd state
* $> flux get all
```
NAME                     	REVISION      	SUSPENDED	READY	MESSAGE
gitrepository/flux-system	main/86fcc24  	False    	True 	stored artifact for revision 'main/86fcc249a3b6b574d4757db0854b95409e3012d5'
gitrepository/podinfo    	master/fa5f988	False    	True 	stored artifact for revision 'master/fa5f9889da837da36f3f74dd5be52bd00567bd58'

NAME                     	REVISION      	SUSPENDED	READY	MESSAGE
kustomization/flux-system	main/86fcc24  	False    	True 	Applied revision: main/86fcc24
kustomization/podinfo    	master/fa5f988	False    	True 	Applied revision: master/fa5f988
```

*** OR ***

Preferred method as you are able to view the change in real time. To install watch, use google to find installation based on OS
* $> watch flux get kustomizations 
```
Every 2.0s: flux get kustomizations 

NAME            REVISION        SUSPENDED       READY   MESSAGE
flux-system     main/86fcc24    False           True    Applied revision: main/86fcc24
podinfo         master/fa5f988  False           True    Applied revision: master/fa5f988
```

### Verify podinfo resources are created and running
* $> echo "\n*** PODS ***"; kc get pod; echo "\n*** DEPLOYMENTS ***"; kc get deploy; echo "\n*** SERVICES ***"; kc get services
```
*** PODS ***
NAME                       READY   STATUS    RESTARTS   AGE
podinfo-66df4b59fb-gjmvr   1/1     Running   0          35m
podinfo-66df4b59fb-ppddw   1/1     Running   0          35m

*** DEPLOYMENTS ***
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
podinfo   2/2     2            2           35m

*** SERVICES ***
NAME         TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)             AGE
kubernetes   ClusterIP   10.96.0.1     <none>        443/TCP             26h
podinfo      ClusterIP   10.96.159.2   <none>        9898/TCP,9999/TCP   35m
```

# That is it! 
