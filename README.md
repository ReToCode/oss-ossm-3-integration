# OpenShift Serverless + OpenShift Service Mesh 3.0 integration

## Resources

* [Latest blogpost about OSSM 3.x](https://www.redhat.com/en/blog/red-hat-openshift-service-mesh-3-developer-preview-update)
* [3.0 installation readme](https://github.com/maistra/istio-operator/blob/maistra-3.0/bundle/README.md)
* [OSSM 3.x code base](https://github.com/openshift-service-mesh)
* [Using gateway injection](https://docs.openshift.com/container-platform/4.14/service_mesh/v2x/ossm-traffic-manage.html#ossm-automatic-gateway-injection_traffic-management)
* [sail-operator](https://github.com/openshift-service-mesh/sail-operator)
* [tech preview announcement](https://www.redhat.com/en/blog/red-hat-openshift-service-mesh-3-now-technology-preview)
* [OSSM 3.x docs](https://docs.openshift.com/service-mesh/3.0.0tp1/about/ossm-about-openshift-service-mesh.html)


## Future things to look at

* [x] Enable mTLS set-up
* [x] Check if we can move the gateways, OCP routes and so on to our own namespace `knative-serving-ingress` instead of `istio-system`
* [x] Make all Serverless-Operator tests pass (Serving only)
* [ ] For the future: Relying on Gateway API resources with net-gateway-api instead of net-istio
* [x] Update all usages of istio-inject annotation to the label, as per [docs](https://istio.io/latest/docs/reference/config/analysis/ist0135/) 

## Contents
* [Installing OSSM 3.x](./INSTALLING.md)
* [Analysis of injection](INJECTION.md)

## Findings

* Basically, it is upstream istio with a helm operator to install it. You can use all helm values to configure istio.
* The injection is different to OSSM 2.x. We will need to have the `istio-injection: enabled` label [on namespaces](https://github.com/openshift-knative/serverless-operator/pull/2928/files#diff-6f53748a4c1bbc532051365399170074ece61e7cb5832f7198756f506d065bb7R2) where we want proxies. This has the "downside" that all pods are injected with the `istio-proxy`. Where this is not applicable, we need to [opt-out](https://github.com/openshift-knative/serverless-operator/pull/2928/files#diff-b9d9613f9d1dfc50ee86295849a277240a6a425454367e619c75c575c51e7cd1R228) of it using the `sidecar.istio.io/inject: 'false'` label (or annotation on the StatefulSets of Eventing). More [here](https://redhat-internal.slack.com/archives/C019EPZ233P/p1727684613070569?thread_ts=1727347316.725149&cid=C019EPZ233P).
* The changes to make tests pass are here: https://github.com/openshift-knative/serverless-operator/pull/2928.
* We will probably need to CI jobs to test OSSM 2.x and OSSM 3.x as these are very different. To be decided by PM.
* To enforce mTLS on the mesh, we need a [PeerAuthentication](https://github.com/openshift-knative/serverless-operator/pull/2928/files#diff-1b15bf3976fab69ecbc170e056aa14195a7daff3327bb61a56182bd00f0d3f4aR1).
* We need a new [DestinationRule](https://github.com/openshift-knative/serverless-operator/pull/2928/files#diff-4afa9b5b369a5c0020ccc20f415d2b60c2e74ad49491b369c60488ec689068b7R13) to make mTLS (as there is no `mtls: true` anymore) work with DomainMappings.
* OSSM 3.x has no default `istio-ingressgateway`, we need to create our own [deployment and RBAC for it](https://github.com/openshift-knative/serverless-operator/pull/2928/files#diff-c6ba409ee99952aca11d97ec6dae9679ffa845d67ff7002b0ab0334e43d8ffe7R1). But with that, we can now host this in `knative-serving-ingress` namespace instead of `istio-system`. With that, this is aligned with Kourier.
* We need to deploy an [istio-cni](https://github.com/openshift-knative/serverless-operator/pull/2928/files#diff-0c1927f3034a63aee6874a0b346eba0f8cbed57f170ff41d197a5802a703e3e5R1) instance, otherwise OSSM 3.x does not work.
* The configuration is migrated from `SMCP` to [Istio CR](https://github.com/openshift-knative/serverless-operator/pull/2928/files#diff-4050713807eceecbb622535f0ec7ec7033fbccc189f9d4bcd23baf2092746105R1).
* As per discussion with the Service Mesh and RHOAI team, [we omit the creation](https://github.com/openshift-knative/serverless-operator/pull/2928/files#diff-b9d9613f9d1dfc50ee86295849a277240a6a425454367e619c75c575c51e7cd1R208) of `NetworkPolices` per default. If we add those, it becomes a "catch-all" policy, which denies everything else. As we don't know the setup up of a customer, we should rather document what communication paths we need and let the customer create the Policies as needed. 

More details in the linked PR.
