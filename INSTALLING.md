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
