# CRDA Operator

Operator for Code Ready Dependency Analytics Plateform Deployment

[![Go Report Card](https://goreportcard.com/badge/github.com/deepak1725/crda-operator)](https://goreportcard.com/report/github.com/deepak1725/crda-operator) 

This Operator will deploy necessary Services in [CRDA Plateform](https://github.com/fabric8-analytics) for Online Flow. 

## Prerequisites:
* [kubectl (v1.17+)](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
* [minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/).
* `kubectl` context should be set to minikube. 
    > You can download [kubectx](https://github.com/ahmetb/kubectx) for easy context, namespace switching. 
* [go](https://golang.org/dl/) version v1.13+.
* [docker](https://docs.docker.com/install/) version 17.03+.
* [kustomize](https://sigs.k8s.io/kustomize/docs/INSTALL.md) v3.1.0+
* [operator-sdk](https://sdk.operatorframework.io/docs/install-operator-sdk/)



## Local Installation:
* Export `AWS_KEY` and `AWS_SECRET` in System Environment. Verify by: `printenv | grep AWS`.  

I have identified 2 possible development scenarios:
1. CRDA Development
2. Operator Development


### One time setup

* You can see preset config values at [com_v1alpha1_codereadyanalytics_cr.yaml](deploy/crds/openshift.com_v1alpha1_codereadyanalytics_cr.yaml). 
Feel free to update any of configuration values as per requirements.

* Modify config values of Operator that caters to your needs and then generate image for same.

* Generate Image: `operator-sdk build {username}/{repo}` 
Ex: `operator-sdk build sharma1725/crda-operator`

* Push Image to registry: `docker push {username}/{repo}`



1. **CRDA Development**

Here i assume you have Operator Image, build in previous step.
* Update Image to your version in `operator.yaml`
* Deploy Operator: `kubectl apply -f deploy/operator.yaml`



2. **Operator Development**

PR's are Welcome. For that, you need to run operator locally:

* Create Namespace `kubectl create ns crda` 
* Create Role `kubectl apply -f deploy/role.yaml`
* Create Service Account `kubectl apply -f deploy/service_account.yaml`
* Create Role Binding `kubectl apply -f deply/service_binding.yaml`
* Deploy CRD: `kubectl apply -f deploy/crds/openshift.com_codereadyanalytics_crd.yaml`
* Deploy CR: `kubectl apply -f deploy/crds/openshift.com_v1alpha1_codereadyanalytics_cr.yaml`

* After changes are done, start operator to witness changes.

`operator-sdk run up --local`

This should deploy all the custom resources (CR) in local cluster in said namespace.


* Execute `kubectl get all -n crda`

You should see something similar to: 

```
NAME                                         READY   STATUS    RESTARTS   AGE
pod/api-server-6ccdcdcbd6-mh9c6              1/1     Running   0          4m16s
pod/bayesian-gremlin-http-6f454d4df7-4ltkf   1/1     Running   0          4m16s
pod/bayesian-pgbouncer-0                     1/1     Running   0          4m16s
pod/bayesian-worker-api-854bd5598f-8x4pj     1/1     Running   1          4m16s

NAME                            TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
service/api-server              NodePort       10.108.78.118    <none>        5000:32000/TCP   4m16s
service/bayesian-gremlin-http   NodePort       10.105.117.132   <none>        8182:32500/TCP   4m16s
service/bayesian-pgbouncer      LoadBalancer   10.104.65.12     <pending>     5432:31500/TCP   4m16s

NAME                                    READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/api-server              1/1     1            1           4m16s
deployment.apps/bayesian-gremlin-http   1/1     1            1           4m17s
deployment.apps/bayesian-worker-api     1/1     1            1           4m16s

NAME                                               DESIRED   CURRENT   READY   AGE
replicaset.apps/api-server-6ccdcdcbd6              1         1         1       4m16s
replicaset.apps/bayesian-gremlin-http-6f454d4df7   1         1         1       4m17s
replicaset.apps/bayesian-worker-api-854bd5598f     1         1         1       4m16s

NAME                                  READY   AGE
statefulset.apps/bayesian-pgbouncer   1/1     4m16s
```

## Get endpoints of services by 
> `minikube service list`

```
|-------------|-----------------------|--------------|-----------------------------|
|  NAMESPACE  |         NAME          | TARGET PORT  |             URL             |
|-------------|-----------------------|--------------|-----------------------------|
| crda        | api-server            |         5000 | http://192.168.xx.yyy:32000 |
| crda        | bayesian-gremlin-http |         8182 | http://192.168.xx.yyy:32500 |
| crda        | bayesian-pgbouncer    |         5432 | http://192.168.xx.yyy:31500 |
| default     | kubernetes            | No node port |
| kube-system | kube-dns              | No node port |
|-------------|-----------------------|--------------|-----------------------------|
```


## Production Installation via [OLM](https://sdk.operatorframework.io/docs/olm-integration/user-guide/) :

* Install OLM on Server `operator-sdk olm install`
* Generate CSV `operator-sdk generate csv --csv-version 0.1.0`
* Install Operator `operator-sdk run packagemanifests --operator-version 0.1.0 --olm`


PS: All resources are deployed in `crda` namespace.
