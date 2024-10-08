# OpenShift Serverless + OpenShift Service Mesh 3.0 integration

## Resources

* [Latest blogpost about OSSM 3.x](https://www.redhat.com/en/blog/red-hat-openshift-service-mesh-3-developer-preview-update)
* [3.0 installation readme](https://github.com/maistra/istio-operator/blob/maistra-3.0/bundle/README.md)
* [OSSM 3.x code base](https://github.com/openshift-service-mesh)
* [Using gateway injection](https://docs.openshift.com/container-platform/4.14/service_mesh/v2x/ossm-traffic-manage.html#ossm-automatic-gateway-injection_traffic-management)
* [sail-operator](https://github.com/openshift-service-mesh/sail-operator)


## Future things to look at

* [x] Enable mTLS set-up
* [x] Check if we can move the gateways, OCP routes and so on to our own namespace `knative-serving-ingress` instead of `istio-system`
* [ ] Make all Serverless-Operator tests pass (Serving only)
* [ ] Make all Serverless-Operator tests pass (Eventing only)
* [ ] Make all Serverless-Operator tests pass (Serving+Eventing)
* [ ] For the future: Relying on Gateway API resources with net-gateway-api instead of net-istio
* [x] Update all usages of istio-inject annotation to the label, as per [docs](https://istio.io/latest/docs/reference/config/analysis/ist0135/) 

## Installation on OpenShift

### Installing istio

```bash
oc new-project istio-system
oc new-project istio-cni

# Install the operator
oc apply -f yaml/sail-operator.yaml
# wait for the operator to be installed and ready

# Installing istio + CNI
oc apply -f yaml/istio.yaml
oc apply -f yaml/istio-cni.yaml
```

### Installing OpenShift Serverless and istio integration

```bash
# Installing OSS
oc apply -f yaml/serverless-operator.yaml
# wait for the operator to be installed and ready

# Creating a Knative istio gateway pod
# From the OSSM 3 docs:
# - Gateways are not part of the control plane. As a security best-practice, Ingress and Egress Gateways should be deployed in a different namespace than the namespace that contains the control plane.
oc new-project knative-serving-ingress
oc apply -f  yaml/gateway-deploy.yaml

# Creating Knative istio gateways
oc apply -f yaml/gateways-knative.yaml

# Enforce mTLS in the mesh
oc apply -f yaml/peer-authentication-mesh-mtls.yaml

# Installing Knative Serving
oc apply -f yaml/knativeserving.yaml
```

## Try it 

```bash
oc apply -f yaml/ksvc.yaml
```

```bash
curl -k https://svc-always-scaled-demo.apps.ci-ln-ggnggm2-76ef8.origin-ci-int-aws.dev.rhcloud.com
curl -k https://svc-always-scaled-demo.apps.sno.codemint.ch
```
```text
Hello Serverless!
```
