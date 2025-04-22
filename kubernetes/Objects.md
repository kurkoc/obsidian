```
kubectl api-resources
```

```
NAME                                SHORTNAMES   APIVERSION                          NAMESPACED   KIND
bindings                                         v1                                  true         Binding
componentstatuses                   cs           v1                                  false        ComponentStatus
configmaps                          cm           v1                                  true         ConfigMap
endpoints                           ep           v1                                  true         Endpoints
events                              ev           v1                                  true         Event
limitranges                         limits       v1                                  true         LimitRange
namespaces                          ns           v1                                  false        Namespace
nodes                               no           v1                                  false        Node
persistentvolumeclaims              pvc          v1                                  true         PersistentVolumeClaim
persistentvolumes                   pv           v1                                  false        PersistentVolume
pods                                po           v1                                  true         Pod
podtemplates                                     v1                                  true         PodTemplate
replicationcontrollers              rc           v1                                  true         ReplicationController
resourcequotas                      quota        v1                                  true         ResourceQuota
secrets                                          v1                                  true         Secret
serviceaccounts                     sa           v1                                  true         ServiceAccount
services                            svc          v1                                  true         Service
mutatingwebhookconfigurations                    admissionregistration.k8s.io/v1     false        MutatingWebhookConfiguration
validatingadmissionpolicies                      admissionregistration.k8s.io/v1     false        ValidatingAdmissionPolicy
validatingadmissionpolicybindings                admissionregistration.k8s.io/v1     false        ValidatingAdmissionPolicyBinding
validatingwebhookconfigurations                  admissionregistration.k8s.io/v1     false        ValidatingWebhookConfiguration
customresourcedefinitions           crd,crds     apiextensions.k8s.io/v1             false        CustomResourceDefinition
apiservices                                      apiregistration.k8s.io/v1           false        APIService
controllerrevisions                              apps/v1                             true         ControllerRevision
daemonsets                          ds           apps/v1                             true         DaemonSet
deployments                         deploy       apps/v1                             true         Deployment
replicasets                         rs           apps/v1                             true         ReplicaSet
statefulsets                        sts          apps/v1                             true         StatefulSet
selfsubjectreviews                               authentication.k8s.io/v1            false        SelfSubjectReview
tokenreviews                                     authentication.k8s.io/v1            false        TokenReview
localsubjectaccessreviews                        authorization.k8s.io/v1             true         LocalSubjectAccessReview
selfsubjectaccessreviews                         authorization.k8s.io/v1             false        SelfSubjectAccessReview
selfsubjectrulesreviews                          authorization.k8s.io/v1             false        SelfSubjectRulesReview
subjectaccessreviews                             authorization.k8s.io/v1             false        SubjectAccessReview
horizontalpodautoscalers            hpa          autoscaling/v2                      true         HorizontalPodAutoscaler
cronjobs                            cj           batch/v1                            true         CronJob
jobs                                             batch/v1                            true         Job
certificatesigningrequests          csr          certificates.k8s.io/v1              false        CertificateSigningRequest
leases                                           coordination.k8s.io/v1              true         Lease
bgpconfigurations                                crd.projectcalico.org/v1            false        BGPConfiguration
bgpfilters                                       crd.projectcalico.org/v1            false        BGPFilter
bgppeers                                         crd.projectcalico.org/v1            false        BGPPeer
blockaffinities                                  crd.projectcalico.org/v1            false        BlockAffinity
caliconodestatuses                               crd.projectcalico.org/v1            false        CalicoNodeStatus
clusterinformations                              crd.projectcalico.org/v1            false        ClusterInformation
felixconfigurations                              crd.projectcalico.org/v1            false        FelixConfiguration
globalnetworkpolicies                            crd.projectcalico.org/v1            false        GlobalNetworkPolicy
globalnetworksets                                crd.projectcalico.org/v1            false        GlobalNetworkSet
hostendpoints                                    crd.projectcalico.org/v1            false        HostEndpoint
ipamblocks                                       crd.projectcalico.org/v1            false        IPAMBlock
ipamconfigs                                      crd.projectcalico.org/v1            false        IPAMConfig
ipamhandles                                      crd.projectcalico.org/v1            false        IPAMHandle
ippools                                          crd.projectcalico.org/v1            false        IPPool
ipreservations                                   crd.projectcalico.org/v1            false        IPReservation
kubecontrollersconfigurations                    crd.projectcalico.org/v1            false        KubeControllersConfiguration
networkpolicies                                  crd.projectcalico.org/v1            true         NetworkPolicy
networksets                                      crd.projectcalico.org/v1            true         NetworkSet
tiers                                            crd.projectcalico.org/v1            false        Tier
endpointslices                                   discovery.k8s.io/v1                 true         EndpointSlice
events                              ev           events.k8s.io/v1                    true         Event
flowschemas                                      flowcontrol.apiserver.k8s.io/v1     false        FlowSchema
prioritylevelconfigurations                      flowcontrol.apiserver.k8s.io/v1     false        PriorityLevelConfiguration
ingressclasses                                   networking.k8s.io/v1                false        IngressClass
ingresses                           ing          networking.k8s.io/v1                true         Ingress
networkpolicies                     netpol       networking.k8s.io/v1                true         NetworkPolicy
runtimeclasses                                   node.k8s.io/v1                      false        RuntimeClass
poddisruptionbudgets                pdb          policy/v1                           true         PodDisruptionBudget
adminnetworkpolicies                anp          policy.networking.k8s.io/v1alpha1   false        AdminNetworkPolicy
clusterrolebindings                              rbac.authorization.k8s.io/v1        false        ClusterRoleBinding
clusterroles                                     rbac.authorization.k8s.io/v1        false        ClusterRole
rolebindings                                     rbac.authorization.k8s.io/v1        true         RoleBinding
roles                                            rbac.authorization.k8s.io/v1        true         Role
priorityclasses                     pc           scheduling.k8s.io/v1                false        PriorityClass
csidrivers                                       storage.k8s.io/v1                   false        CSIDriver
csinodes                                         storage.k8s.io/v1                   false        CSINode
csistoragecapacities                             storage.k8s.io/v1                   true         CSIStorageCapacity
storageclasses                      sc           storage.k8s.io/v1                   false        StorageClass
volumeattachments                                storage.k8s.io/v1                   false        VolumeAttachment
```

```
kubectl api-resources -o wide
```

```
kubectl api-resources --namespaced=true
```

explain komutu yardımıyla herhangi bir objenin kullanım detaylarına bakabiliriz. 

```
kubectl explain deployment

GROUP:      apps
KIND:       Deployment
VERSION:    v1

DESCRIPTION:
    Deployment enables declarative updates for Pods and ReplicaSets.

FIELDS:
  apiVersion    <string>
    APIVersion defines the versioned schema of this representation of an object.
    Servers should convert recognized schemas to the latest internal value, and
    may reject unrecognized values. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources

  kind  <string>
    Kind is a string value representing the REST resource this object
    represents. Servers may infer this from the endpoint the client submits
    requests to. Cannot be updated. In CamelCase. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds

  metadata      <ObjectMeta>
    Standard object's metadata. More info:
    https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#metadata

  spec  <DeploymentSpec>
    Specification of the desired behavior of the Deployment.

  status        <DeploymentStatus>
    Most recently observed status of the Deployment.

```

Üstteki komut ile sadece ana doküman field'ları listelenir. Örneğin `metadata` alanının detaylarını tam olarak göremeyiz. 

```
kubectl explain deployment.metadata
```

diyerek iç objelerden herhangi birisinin detayına bakabiliriz.


```
kubectl explain deployment.spec.strategy

GROUP:      apps
KIND:       Deployment
VERSION:    v1

FIELD: strategy <DeploymentStrategy>


DESCRIPTION:
    The deployment strategy to use to replace existing pods with new ones.
    DeploymentStrategy describes how to replace existing pods with new ones.

FIELDS:
  rollingUpdate <RollingUpdateDeployment>
    Rolling update config params. Present only if DeploymentStrategyType =
    RollingUpdate.

  type  <string>
  enum: Recreate, RollingUpdate
    Type of deployment. Can be "Recreate" or "RollingUpdate". Default is
    RollingUpdate.

    Possible enum values:
     - `"Recreate"` Kill all existing pods before creating new ones.
     - `"RollingUpdate"` Replace the old ReplicaSets by new one using rolling
    update i.e gradually scale down the old ReplicaSets and scale up the new
    one.
```

Eğer istenirse;

```
kubectl explain deployment --recursive
```

diyerek de içiçe bütün alanları detaylarıyla beraber getirebiliriz.

