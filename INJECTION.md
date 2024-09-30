# Istio Sidecar Injection

## Preface

Previously istio relied on an annotation to inject sidecars `sidecar.istio.io/inject: "true"`. 
This annotation is now deprecated and will be removed in the future. Instead, we should either use `sidecar.istio.io/inject: "true"` as a label OR
migrate to a revisioned istio installation, where the `istio.io/rev: <revision-name>` label is used to inject sidecars.
See documentation for more information: https://istio.io/latest/docs/setup/additional-setup/sidecar-injection/#controlling-the-injection-policy

## OSSM 3.x vs Upstream

Knative uses the non-revisioned istio installation and partially still uses the annotation approach. We cannot yet change this, 
because Knative Eventing uses `StatefulSets` with the annotation configured. Due to the Kubernetes [bug](https://github.com/kubernetes/kubernetes/issues/90519)
labels / annotations of `StatefulSets` cannot be updated without recreation. This would cause a downtime during upgrade for our customers
which is unfeasible.

So we need to keep the annotation approach for now, for upstream Knative + istio and for OpenShift + OSSM 3.x as well.

### Upstream (vanilla K8s)

```bash
# Install istio 1.21
/istio-1.21.0/bin
❯ ./istioctl install
This will install the Istio 1.21.0 "default" profile (with components: Istio core, Istiod, and Ingress gateways) into the cluster. Proceed? (y/N) y
✔ Istio core installed
✔ Istiod installed
✔ Ingress gateways installed
✔ Installation complete                                                                                                                                                                                                                     Made this installation the default for injection and validation.
```

```bash
# Create a namespace
kubectl create ns istio-test
```

```bash
# Create a deployment where the sidecar should be injected (using annotation)
oc apply -n istio-test -f injectionyaml/curl-annotation.yaml

oc -n istio-test get pod -l app=curl

NAME                    READY   STATUS    RESTARTS   AGE
curl-76f586c599-qsjrt   1/1     Running   0          28s
```

```bash
# Create a deployment where the sidecar should be injected (using label)
oc apply -n istio-test -f injectionyaml/curl-label.yaml

oc -n istio-test get pod -l app=curl

NAME                    READY   STATUS    RESTARTS   AGE
curl-56c6d99c54-qkwb2   2/2     Running   0          62s

oc -n istio-test get pod -l app=curl -o jsonpath="{range .items[*]} {.spec.containers[*].name}{'\n'}{end}"
 curl
 istio-proxy
```

### Downstream (OpenShift + Sail Operator)

```bash
# Install OSSM 3.x
oc new-project istio-system
oc new-project istio-cni
oc apply -f yaml/sail-operator.yaml
oc apply -f yaml/istio-cni.yaml
oc apply -f yaml/istio.yaml
```

```bash
# Create a namespace
oc create ns istio-test
```

```bash
# Create a deployment where the sidecar should be injected (using annotation)
oc apply -n istio-test -f injectionyaml/curl-annotation.yaml

oc -n istio-test get pod -l app=curl

NAME                    READY   STATUS    RESTARTS   AGE
curl-76f586c599-qsjrt   1/1     Running   0          28s
```

```bash
# Create a deployment where the sidecar should be injected (using label)
oc apply -n istio-test -f injectionyaml/curl-label.yaml

oc -n istio-test get pod -l app=curl

NAME                    READY   STATUS    RESTARTS   AGE
curl-56c6d99c54-qkwb2   2/2     Running   0          62s

oc -n istio-test get pod -l app=curl -o jsonpath="{range .items[*]} {.spec.containers[*].name}{'\n'}{end}"
 curl
 istio-proxy
```



