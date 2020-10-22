# InfraNode-OKD4
```bash
#label node to infra
oc label node/worker03.openshift.ba.io node-role.kubernetes.io/infra=
oc label node/worker03.openshift.ba.io node-role.kubernetes.io/worker-

#move ingress controller
oc get ingresscontroller default -n openshift-ingress-operator -o yaml
oc edit ingresscontroller default -n openshift-ingress-operator -o yaml
...
  spec:
    nodePlacement:
      nodeSelector:
        matchLabels:
          node-role.kubernetes.io/infra: ""
...

#move imageregistry
oc get configs.imageregistry.operator.openshift.io cluster -o yaml
oc edit configs.imageregistry.operator.openshift.io cluster
...
spec:
  httpSecret: d4e852df9cff2a2aee186887e878b32193a1bfc99ee3a7496877cb4a5a8d1c4af86e4495135394dbfff1ad3b9ac8c9bf33733e5d7c66f16f58ed329b87eb23e9
  logging: 2
  managementState: Managed
  nodeSelector:
    node-role.kubernetes.io/infra: ""
...

#move cluster monitoring
vim cluster-monitoring-configmap.yaml
...
apiVersion: v1
kind: ConfigMap
metadata:
  name: cluster-monitoring-config
  namespace: openshift-monitoring
data:
  config.yaml: |+
    alertmanagerMain:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
    prometheusK8s:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
    prometheusOperator:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
    grafana:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
    k8sPrometheusAdapter:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
    kubeStateMetrics:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
    telemeterClient:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
    openshiftStateMetrics:
      nodeSelector:
        node-role.kubernetes.io/infra: ""
...
oc create -f cluster-monitoring-configmap.yaml
watch 'oc get pod -n openshift-monitoring -o wide'

```
