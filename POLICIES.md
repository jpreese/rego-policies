# Policies

## Violations

* [Namespace has a NetworkPolicy](#namespace-has-a-networkpolicy)
* [Namespace has a ResourceQuota](#namespace-has-a-resourcequota)
* [Common k8s labels are set](#common-k8s-labels-are-set)
* [Container env has CONTAINER_MAX_MEMORY set](#container-env-has-container_max_memory-set)
* [Container image is not set as latest](#container-image-is-not-set-as-latest)
* [Container image is not from a known registry](#container-image-is-not-from-a-known-registry)
* [Container does not set Java Xmx option](#container-does-not-set-java-xmx-option)
* [Label key is consistent](#label-key-is-consistent)
* [Container liveness and readiness probes are equal](#container-liveness-and-readiness-probes-are-equal)
* [Container liveness prob is not set](#container-liveness-prob-is-not-set)
* [Container readiness prob is not set](#container-readiness-prob-is-not-set)
* [Container resource limits CPU not set](#container-resource-limits-cpu-not-set)
* [Container resource limits memory not greater than](#container-resource-limits-memory-not-greater-than)
* [Container resource limits memory not set](#container-resource-limits-memory-not-set)
* [Container resources limit memory has incorrect unit](#container-resources-limit-memory-has-incorrect-unit)
* [Container resources requests cpu has incorrect unit](#container-resources-requests-cpu-has-incorrect-unit)
* [Container resource requests memory not greater than](#container-resource-requests-memory-not-greater-than)
* [Container secret not mounted as envs](#container-secret-not-mounted-as-envs)
* [Container volume mount path is consistent](#container-volume-mount-path-is-consistent)
* [Container volume mount not set](#container-volume-mount-not-set)
* [DeploymentConfig triggers not set](#deploymentconfig-triggers-not-set)
* [Pod hostnetwork not set](#pod-hostnetwork-not-set)
* [Pod replica below 1](#pod-replica-below-1)
* [Pod replica is not odd](#pod-replica-is-not-odd)
* [RoleBinding has apiGroup set](#rolebinding-has-apigroup-set)
* [RoleBinding has kind set](#rolebinding-has-kind-set)
* [BuildConfig no longer served by v1](#buildconfig-no-longer-served-by-v1)
* [DeploymentConfig no longer served by v1](#deploymentconfig-no-longer-served-by-v1)
* [ImageStream no longer served by v1](#imagestream-no-longer-served-by-v1)
* [ProjectRequest no longer served by v1](#projectrequest-no-longer-served-by-v1)
* [RoleBinding no longer served by v1](#rolebinding-no-longer-served-by-v1)
* [Route no longer served by v1](#route-no-longer-served-by-v1)
* [SecurityContextConstraints no longer served by v1](#securitycontextconstraints-no-longer-served-by-v1)
* [Template no longer served by v1](#template-no-longer-served-by-v1)
* [BuildConfig exposeDockerSocket deprecated](#buildconfig-exposedockersocket-deprecated)
* [authorization openshift io is deprecated](#authorization-openshift-io-is-deprecated)
* [automationbroker io v1alpha1 is deprecated](#automationbroker-io-v1alpha1-is-deprecated)
* [operators coreos com v1 CatalogSourceConfigs is deprecated](#operators-coreos-com-v1-catalogsourceconfigs-is-deprecated)
* [operators coreos com v2 CatalogSourceConfigs is deprecated](#operators-coreos-com-v2-catalogsourceconfigs-is-deprecated)
* [operators coreos com v1 OperatorSource is deprecated](#operators-coreos-com-v1-operatorsource-is-deprecated)
* [osb openshift io v1 is deprecated](#osb-openshift-io-v1-is-deprecated)
* [servicecatalog k8s io v1beta1 is deprecated](#servicecatalog-k8s-io-v1beta1-is-deprecated)
* [BuildConfig jenkinsPipelineStrategy is deprecated](#buildconfig-jenkinspipelinestrategy-is-deprecated)
* [Deployment has a matching PodDisruptionBudget](#deployment-has-a-matching-poddisruptionbudget)
* [Deployment has matching PersistentVolumeClaim](#deployment-has-matching-persistentvolumeclaim)
* [Deployment has a matching Service](#deployment-has-a-matching-service)
* [Deployment has matching ServiceAccount](#deployment-has-matching-serviceaccount)
* [Service has matching ServiceMonitor](#service-has-matching-servicemonitor)
* [Image contains expected SHA in history.](#image-contains-expected-sha-in-history.)
* [Image size is not greater than an expected value](#image-size-is-not-greater-than-an-expected-value)

## Namespace has a NetworkPolicy

**Severity:** violation

**Resources:** core/Namespace networking.k8s.io/NetworkPolicy

Kubernetes network policies specify the access permissions for groups of pods,
much like security groups in the cloud are used to control access to VM instances.
In other words, it creates firewalls between pods running on a Kubernetes cluster.
See: Network policies -> https://learnk8s.io/production-best-practices#governance

### Rego

```rego
package combine.namespace_has_networkpolicy

import data.lib.konstraint

violation[msg] {
  manifests := input[_]
  some i

  lower(manifests[i].apiVersion) == "v1"
  lower(manifests[i].kind) == "namespace"
  namespace := manifests[i]

  not namespace_has_networkpolicy(manifests)

  msg := konstraint.format(sprintf("%s/%s does not have a networking.k8s.io/v1:NetworkPolicy. See: https://docs.openshift.com/container-platform/4.4/networking/configuring-networkpolicy.html", [namespace.kind, namespace.metadata.name]))
}

namespace_has_networkpolicy(manifests) {
  current := manifests[_]

  lower(current.apiVersion) == "networking.k8s.io/v1"
  lower(current.kind) == "networkpolicy"
}
```

_source: [policy/combine/namespace-has-networkpolicy](policy/combine/namespace-has-networkpolicy)_

## Namespace has a ResourceQuota

**Severity:** violation

**Resources:** core/Namespace core/ResourceQuota

With ResourceQuotas, you can limit the total resource consumption of all containers inside a Namespace.
Defining a resource quota for a namespace limits the total amount of CPU, memory or storage resources
that can be consumed by all containers belonging to that namespace. You can also set quotas for other
Kubernetes objects such as the number of Pods in the current namespace.
See: Namespace limits -> https://learnk8s.io/production-best-practices#governance

### Rego

```rego
package combine.namespace_has_resourcequota

import data.lib.konstraint

violation[msg] {
  manifests := input[_]
  some i

  lower(manifests[i].apiVersion) == "v1"
  lower(manifests[i].kind) == "namespace"
  namespace := manifests[i]

  not namespace_has_resourcequota(manifests)

  msg := konstraint.format(sprintf("%s/%s does not have a core/v1:ResourceQuota. See: https://docs.openshift.com/container-platform/4.5/applications/quotas/quotas-setting-per-project.html", [namespace.kind, namespace.metadata.name]))
}

namespace_has_resourcequota(manifests) {
  current := manifests[_]

  lower(current.apiVersion) == "v1"
  lower(current.kind) == "resourcequota"
}
```

_source: [policy/combine/namespace-has-resourcequota](policy/combine/namespace-has-resourcequota)_

## Common k8s labels are set

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/DaemonSet apps/Deployment apps/StatefulSet core/Service route.openshift.io/Route

Check if all workload related kinds contain labels as suggested by k8s.
See: https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels

### Rego

```rego
package ocp.bestpractices.common_k8s_labels_notset

import data.lib.konstraint
import data.lib.openshift

violation[msg] {
  openshift.is_all_kind

  obj := konstraint.object
  not is_common_labels_set(obj.metadata)

  msg := konstraint.format(sprintf("%s/%s: does not contain all the expected k8s labels in 'metadata.labels'. See: https://kubernetes.io/docs/concepts/overview/working-with-objects/common-labels", [obj.kind, obj.metadata.name]))
}

is_common_labels_set(metadata) {
  metadata.labels["app.kubernetes.io/name"]
  metadata.labels["app.kubernetes.io/instance"]
  metadata.labels["app.kubernetes.io/version"]
  metadata.labels["app.kubernetes.io/component"]
  metadata.labels["app.kubernetes.io/part-of"]
  metadata.labels["app.kubernetes.io/managed-by"]
}
```

_source: [policy/ocp/bestpractices/common-k8s-labels-notset](policy/ocp/bestpractices/common-k8s-labels-notset)_

## Container env has CONTAINER_MAX_MEMORY set

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/DaemonSet apps/Deployment apps/StatefulSet

Red Hat OpenJDK image uses CONTAINER_MAX_MEMORY env via the downward API to set Java memory settings.
Instead of manually setting -Xmx, let the image automatically set it for you.
See: https://github.com/jboss-openshift/cct_module/blob/master/jboss/container/java/jvm/bash/artifacts/opt/jboss/container/java/jvm/java-default-options

### Rego

```rego
package ocp.bestpractices.container_env_maxmemory_notset

import data.lib.konstraint
import data.lib.openshift

violation[msg] {
  openshift.is_workload_kind

  container := openshift.containers[_]

  not is_env_max_memory_set(container)
  obj := konstraint.object

  msg := konstraint.format(sprintf("%s/%s: container '%s' does not have an env named 'CONTAINER_MAX_MEMORY' which is used by the Red Hat base images to calculate memory. See: https://docs.openshift.com/container-platform/4.4/nodes/clusters/nodes-cluster-resource-configure.html and https://github.com/jboss-openshift/cct_module/blob/master/jboss/container/java/jvm/bash/artifacts/opt/jboss/container/java/jvm/java-default-options", [obj.kind, obj.metadata.name, container.name]))
}

is_env_max_memory_set(container) {
  env := container.env[_]
  env.name == "CONTAINER_MAX_MEMORY"
  env.valueFrom.resourceFieldRef.resource == "limits.memory"
}
```

_source: [policy/ocp/bestpractices/container-env-maxmemory-notset](policy/ocp/bestpractices/container-env-maxmemory-notset)_

## Container image is not set as latest

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/DaemonSet apps/Deployment apps/StatefulSet

Images should use immutable tags. Today's latest is not tomorrows latest.

### Rego

```rego
package ocp.bestpractices.container_image_latest

import data.lib.konstraint
import data.lib.openshift

violation[msg] {
  openshift.is_workload_kind

  container := openshift.containers[_]

  endswith(container.image, ":latest")
  obj := konstraint.object

  msg := konstraint.format(sprintf("%s/%s: container '%s' is using the latest tag for its image (%s), which is an anti-pattern.", [obj.kind, obj.metadata.name, container.name, container.image]))
}
```

_source: [policy/ocp/bestpractices/container-image-latest](policy/ocp/bestpractices/container-image-latest)_

## Container image is not from a known registry

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/DaemonSet apps/Deployment apps/StatefulSet

Only images from trusted and known registries should be used

### Rego

```rego
package ocp.bestpractices.container_image_unknownregistries

import data.lib.konstraint
import data.lib.openshift

violation[msg] {
  openshift.is_workload_kind

  container := openshift.containers[_]
  registry_list := ["image-registry.openshift-image-registry.svc", "registry.redhat.io/", "quay.io/"]

  not known_registry(container.image, registry_list)

  obj := konstraint.object
  msg := konstraint.format(sprintf("%s/%s: container '%s' is from (%s), which is an unknown registry.", [obj.kind, obj.metadata.name, container.name, container.image]))
}

known_registry(image, knownregistry){
  registry := knownregistry[_]
  startswith(image, registry)
}
```

_source: [policy/ocp/bestpractices/container-image-unknownregistries](policy/ocp/bestpractices/container-image-unknownregistries)_

## Container does not set Java Xmx option

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/DaemonSet apps/Deployment apps/StatefulSet

Red Hat OpenJDK image uses CONTAINER_MAX_MEMORY env via the downward API to set Java memory settings.
Instead of manually setting -Xmx, let the image automatically set it for you.

### Rego

```rego
package ocp.bestpractices.container_java_xmx_set

import data.lib.konstraint
import data.lib.openshift

violation[msg] {
  openshift.is_workload_kind

  container := openshift.containers[_]

  container_opts_contains_xmx(container)
  obj := konstraint.object

  msg := konstraint.format(sprintf("%s/%s: container '%s' contains -Xmx in either, command, args or env. Instead, it is suggested you use the downward API to set the env 'CONTAINER_MAX_MEMORY'", [obj.kind, obj.metadata.name, container.name]))
}

container_opts_contains_xmx(container) {
  value := container.command[_]
  contains(value, "-Xmx")
}

container_opts_contains_xmx(container) {
  value := container.args[_]
  contains(value, "-Xmx")
}

container_opts_contains_xmx(container) {
  value := container.env[_]
  contains(value.value, "-Xmx")
}
```

_source: [policy/ocp/bestpractices/container-java-xmx-set](policy/ocp/bestpractices/container-java-xmx-set)_

## Label key is consistent

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/DaemonSet apps/Deployment apps/StatefulSet

Label keys should be qualified by 'app.kubernetes.io' or 'company.com' to allow a consistent understanding.

### Rego

```rego
package ocp.bestpractices.container_labelkey_inconsistent

import data.lib.konstraint
import data.lib.openshift

violation[msg] {
  openshift.is_workload_kind

  some key
  obj := konstraint.object
  value := obj.metadata.labels[key]

  not label_key_starts_with_expected(key)

  msg := konstraint.format(sprintf("%s/%s: has a label key which did not start with 'app.kubernetes.io/' or 'redhat-cop.github.com/'. Found '%s'", [obj.kind, obj.metadata.name, key]))
}

label_key_starts_with_expected(key) {
  startswith(key, "app.kubernetes.io/")
}

label_key_starts_with_expected(key) {
  startswith(key, "redhat-cop.github.com/")
}
```

_source: [policy/ocp/bestpractices/container-labelkey-inconsistent](policy/ocp/bestpractices/container-labelkey-inconsistent)_

## Container liveness and readiness probes are equal

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/DaemonSet apps/Deployment apps/StatefulSet

When Liveness and Readiness probes are pointing to the same endpoint, the effects of the probes are combined.
When the app signals that it's not ready or live, the kubelet detaches the container from the Service and delete it at the same time.
You might notice dropping connections because the container does not have enough time to drain the current connections or process the incoming ones.
See: Health checks -> https://learnk8s.io/production-best-practices#application-development

### Rego

```rego
package ocp.bestpractices.container_liveness_readinessprobe_equal

import data.lib.konstraint
import data.lib.openshift

violation[msg] {
  openshift.is_workload_kind

  container := openshift.containers[_]

  container.livenessProbe
  container.readinessProbe
  container.livenessProbe == container.readinessProbe
  obj := konstraint.object

  msg := konstraint.format(sprintf("%s/%s: container '%s' livenessProbe and readinessProbe are equal, which is an anti-pattern.", [obj.kind, obj.metadata.name, container.name]))
}
```

_source: [policy/ocp/bestpractices/container-liveness-readinessprobe-equal](policy/ocp/bestpractices/container-liveness-readinessprobe-equal)_

## Container liveness prob is not set

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/DaemonSet apps/Deployment apps/StatefulSet

A Liveness checks determines if the container in which it is scheduled is still running.
If the liveness probe fails due to a condition such as a deadlock, the kubelet kills the container.
See: https://docs.openshift.com/container-platform/4.4/applications/application-health.html

### Rego

```rego
package ocp.bestpractices.container_livenessprobe_notset

import data.lib.konstraint
import data.lib.openshift

violation[msg] {
  openshift.is_workload_kind

  container := openshift.containers[_]

  konstraint.missing_field(container, "livenessProbe")
  obj := konstraint.object

  msg := konstraint.format(sprintf("%s/%s: container '%s' has no livenessProbe. See: https://docs.openshift.com/container-platform/4.4/applications/application-health.html", [obj.kind, obj.metadata.name, container.name]))
}
```

_source: [policy/ocp/bestpractices/container-livenessprobe-notset](policy/ocp/bestpractices/container-livenessprobe-notset)_

## Container readiness prob is not set

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/DaemonSet apps/Deployment apps/StatefulSet

A Readiness check determines if the container in which it is scheduled is ready to service requests.
If the readiness probe fails a container, the endpoints controller ensures the container has its IP address removed from the endpoints of all services.
See: https://docs.openshift.com/container-platform/4.4/applications/application-health.html

### Rego

```rego
package ocp.bestpractices.container_readinessprobe_notset

import data.lib.konstraint
import data.lib.openshift

violation[msg] {
  openshift.is_workload_kind

  container := openshift.containers[_]

  konstraint.missing_field(container, "readinessProbe")
  obj := konstraint.object

  msg := konstraint.format(sprintf("%s/%s: container '%s' has no readinessProbe. See: https://docs.openshift.com/container-platform/4.4/applications/application-health.html", [obj.kind, obj.metadata.name, container.name]))
}
```

_source: [policy/ocp/bestpractices/container-readinessprobe-notset](policy/ocp/bestpractices/container-readinessprobe-notset)_

## Container resource limits CPU not set

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/DaemonSet apps/Deployment apps/StatefulSet

If you're not sure about what's the best settings for your app, it's better not to set the CPU limits.
See: Resources utilisation -> https://learnk8s.io/production-best-practices#application-development
See: https://www.reddit.com/r/kubernetes/comments/all1vg/on_kubernetes_cpu_limits

### Rego

```rego
package ocp.bestpractices.container_resources_limits_cpu_set

import data.lib.konstraint
import data.lib.openshift

violation[msg] {
  openshift.is_workload_kind

  container := openshift.containers[_]

  container.resources.limits.cpu
  obj := konstraint.object

  msg := konstraint.format(sprintf("%s/%s: container '%s' has cpu limits (%d). It is not recommended to limit cpu. See: https://www.reddit.com/r/kubernetes/comments/all1vg/on_kubernetes_cpu_limits", [obj.kind, obj.metadata.name, container.name, container.resources.limits.cpu]))
}
```

_source: [policy/ocp/bestpractices/container-resources-limits-cpu-set](policy/ocp/bestpractices/container-resources-limits-cpu-set)_

## Container resource limits memory not greater than

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/DaemonSet apps/Deployment apps/StatefulSet

Setting a too high memory limit can cause under utilisation on a node.
It is better to run multiple pods which use smaller limits.
See: Resources utilisation -> https://learnk8s.io/production-best-practices#application-development

### Rego

```rego
package ocp.bestpractices.container_resources_limits_memory_greater_than

import data.lib.konstraint
import data.lib.memory
import data.lib.openshift

violation[msg] {
  openshift.is_workload_kind

  #NOTE: upperBound is an arbitrary number and it should be changed to what your company believes is the correct policy
  upperBound := 6 * memory.gb

  container := openshift.containers[_]

  not startswith(container.resources.limits.memory, "$")
  memoryBytes := units.parse_bytes(container.resources.limits.memory)
  memoryBytes > upperBound
  obj := konstraint.object

  msg := konstraint.format(sprintf("%s/%s: container '%s' has a memory limit of '%s' which is larger than the upper '%dGi' limit.", [obj.kind, obj.metadata.name, container.name, container.resources.limits.memory, (upperBound / memory.gb)]))
}
```

_source: [policy/ocp/bestpractices/container-resources-limits-memory-greater-than](policy/ocp/bestpractices/container-resources-limits-memory-greater-than)_

## Container resource limits memory not set

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/DaemonSet apps/Deployment apps/StatefulSet

A container without a memory limit has memory utilisation of zero — according to the scheduler.
An unlimited number of Pods if schedulable on any nodes leading to resource overcommitment and potential node (and kubelet) crashes.
See: Resources utilisation -> https://learnk8s.io/production-best-practices#application-development

### Rego

```rego
package ocp.bestpractices.container_resources_limits_memory_notset

import data.lib.konstraint
import data.lib.openshift

violation[msg] {
  openshift.is_workload_kind

  container := openshift.containers[_]

  # TODO: Maybe should use below factored out?
  #konstraint.missing_field(container.resources.limits, "memory")
  not container.resources.limits.memory
  obj := konstraint.object

  msg := konstraint.format(sprintf("%s/%s: container '%s' has no memory limits. It is recommended to limit memory, as memory always has a maximum. See: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers", [obj.kind, obj.metadata.name, container.name]))
}
```

_source: [policy/ocp/bestpractices/container-resources-limits-memory-notset](policy/ocp/bestpractices/container-resources-limits-memory-notset)_

## Container resources limit memory has incorrect unit

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/DaemonSet apps/Deployment apps/StatefulSet

Begininers can easily confuse the allowed memory unit, this policy enforces what is valid.
k8s also allows for millibyte as a unit for memory, which causes unintended consequences for the scheduler.
See: https://github.com/kubernetes/kubernetes/issues/28741
See: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-units-in-kubernetes

### Rego

```rego
package ocp.bestpractices.container_resources_memoryunit_incorrect

import data.lib.konstraint
import data.lib.openshift

violation[msg] {
  openshift.is_workload_kind

  container := openshift.containers[_]

  not startswith(container.resources.requests.memory, "$")
  not startswith(container.resources.limits.memory, "$")
  not is_resource_memory_units_valid(container)
  obj := konstraint.object

  msg := konstraint.format(sprintf("%s/%s: container '%s' memory resources for limits or requests (%s / %s) has an incorrect unit. See: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-units-in-kubernetes", [obj.kind, obj.metadata.name, container.name, container.resources.limits.memory, container.resources.requests.memory]))
}

is_resource_memory_units_valid(container) {
  memoryLimitsUnit := regex.find_n("[A-Za-z]+", container.resources.limits.memory, 1)[0]
  memoryRequestsUnit := regex.find_n("[A-Za-z]+", container.resources.requests.memory, 1)[0]

  units := ["Ei", "Pi", "Ti", "Gi", "Mi", "Ki", "E", "P", "T", "G", "M", "K"]
  memoryLimitsUnit == units[_]
  memoryRequestsUnit == units[_]
}
```

_source: [policy/ocp/bestpractices/container-resources-memoryunit-incorrect](policy/ocp/bestpractices/container-resources-memoryunit-incorrect)_

## Container resources requests cpu has incorrect unit

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/DaemonSet apps/Deployment apps/StatefulSet

Beginners can easily confuse the allowed cpu unit, this policy enforces what is valid.
See: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-units-in-kubernetes

### Rego

```rego
package ocp.bestpractices.container_resources_requests_cpuunit_incorrect

import data.lib.konstraint
import data.lib.openshift

violation[msg] {
  openshift.is_workload_kind

  container := openshift.containers[_]

  not is_resource_requests_cpu_contains_dollar(container)
  not is_resource_requests_cpu_units_valid(container)
  obj := konstraint.object

  msg := konstraint.format(sprintf("%s/%s container '%s' cpu resources for requests (%s) has an incorrect unit. See: https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/#resource-units-in-kubernetes", [obj.kind, obj.metadata.name, container.name, container.resources.requests.cpu]))
}

is_resource_requests_cpu_contains_dollar(container) {
  not is_resource_requests_cpu_a_core(container)
  startswith(container.resources.requests.cpu, "$")
}

is_resource_requests_cpu_a_core(container)  {
  is_number(input.resources.requests.cpu)
  to_number(input.resources.requests.cpu)
}

is_resource_requests_cpu_units_valid(container)  {
  is_resource_requests_cpu_a_core(container)
}

is_resource_requests_cpu_units_valid(container)  {
  not is_resource_requests_cpu_a_core(container)

  # 'cpu' can be a quoted number, which is why we concat an empty string[] to match whole cpu cores
  cpuRequestsUnit := array.concat(regex.find_n("[A-Za-z]+", container.resources.requests.cpu, 1), [""])[0]

  units := ["m", ""]
  cpuRequestsUnit == units[_]
}
```

_source: [policy/ocp/bestpractices/container-resources-requests-cpuunit-incorrect](policy/ocp/bestpractices/container-resources-requests-cpuunit-incorrect)_

## Container resource requests memory not greater than

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/DaemonSet apps/Deployment apps/StatefulSet

Setting a too high memory request can cause under utilisation on a node.
It is better to run multiple pods which use smaller requests.
See: Resources utilisation -> https://learnk8s.io/production-best-practices#application-development

### Rego

```rego
package ocp.bestpractices.container_resources_requests_memory_greater_than

import data.lib.konstraint
import data.lib.memory
import data.lib.openshift

violation[msg] {
  openshift.is_workload_kind

  #NOTE: upperBound is an arbitrary number and it should be changed to what your company believes is the correct policy
  upperBound := 2 * memory.gb

  container := openshift.containers[_]

  not startswith(container.resources.requests.memory, "$")
  memoryBytes := units.parse_bytes(container.resources.requests.memory)
  memoryBytes > upperBound
  obj := konstraint.object

  msg := konstraint.format(sprintf("%s/%s: container '%s' has a memory request of '%s' which is larger than the upper '%dGi' limit.", [obj.kind, obj.metadata.name, container.name, container.resources.requests.memory, (upperBound / memory.gb)]))
}
```

_source: [policy/ocp/bestpractices/container-resources-requests-memory-greater-than](policy/ocp/bestpractices/container-resources-requests-memory-greater-than)_

## Container secret not mounted as envs

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/DaemonSet apps/Deployment apps/StatefulSet

The content of Secret resources should be mounted into containers as volumes rather than passed in as environment variables.
This is to prevent that the secret values appear in the command that was used to start the container, which may be inspected
by individuals that shouldn't have access to the secret values.
See: Configuration and secrets -> https://learnk8s.io/production-best-practices#application-development

### Rego

```rego
package ocp.bestpractices.container_secret_mounted_envs

import data.lib.konstraint
import data.lib.openshift

violation[msg] {
  openshift.is_workload_kind

  container := openshift.containers[_]

  env := container.env[_]
  env.valueFrom.secretKeyRef
  obj := konstraint.object

  msg := konstraint.format(sprintf("%s/%s: container '%s' has a secret '%s' mounted as an environment variable. As secrets are not secret, its not good practice to mount as env vars.", [obj.kind, obj.metadata.name, container.name, env.valueFrom.secretKeyRef.name]))
}
```

_source: [policy/ocp/bestpractices/container-secret-mounted-envs](policy/ocp/bestpractices/container-secret-mounted-envs)_

## Container volume mount path is consistent

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/DaemonSet apps/Deployment apps/StatefulSet

Mount paths should be mounted at '/var/run/company.com' to allow a consistent understanding.

### Rego

```rego
package ocp.bestpractices.container_volumemount_inconsistent_path

import data.lib.konstraint
import data.lib.openshift

violation[msg] {
  openshift.is_workload_kind

  container := openshift.containers[_]

  volumeMount := container.volumeMounts[_]
  not startswith(volumeMount.mountPath, "/var/run")
  obj := konstraint.object

  msg := konstraint.format(sprintf("%s/%s: container '%s' has a volumeMount '%s' mountPath at '%s'. A good practice is to use consistent mount paths, such as: /var/run/{organization}/{mount} - i.e.: /var/run/io.redhat-cop/my-secret", [obj.kind, obj.metadata.name, container.name, volumeMount.name, volumeMount.mountPath]))
}
```

_source: [policy/ocp/bestpractices/container-volumemount-inconsistent-path](policy/ocp/bestpractices/container-volumemount-inconsistent-path)_

## Container volume mount not set

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/DaemonSet apps/Deployment apps/StatefulSet

A volume does not have a corresponding volume mount. There is probably a mistake in your definition.

### Rego

```rego
package ocp.bestpractices.container_volumemount_missing

import data.lib.konstraint
import data.lib.openshift

violation[msg] {
  openshift.is_workload_kind

  volume := openshift.pods[_].spec.volumes[_]

  not containers_volumemounts_contains_volume(openshift.containers, volume)
  obj := konstraint.object

  msg := konstraint.format(sprintf("%s/%s: volume '%s' does not have a volumeMount in any of the containers.", [obj.kind, obj.metadata.name, volume.name]))
}

containers_volumemounts_contains_volume(containers, volume) {
  containers[_].volumeMounts[_].name == volume.name
}
```

_source: [policy/ocp/bestpractices/container-volumemount-missing](policy/ocp/bestpractices/container-volumemount-missing)_

## DeploymentConfig triggers not set

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig

If you are using a DeploymentConfig without 'spec.triggers' set, you could probably just use the k8s Deployment.

### Rego

```rego
package ocp.bestpractices.deploymentconfig_triggers_notset

import data.lib.konstraint
import data.lib.openshift

violation[msg] {
  openshift.is_deploymentconfig

  obj := konstraint.object
  konstraint.missing_field(obj.spec, "triggers")

  msg := konstraint.format(sprintf("%s/%s: has no triggers set. Could you use a k8s native Deployment? See: https://kubernetes.io/docs/concepts/workloads/controllers/deployment", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/bestpractices/deploymentconfig-triggers-notset](policy/ocp/bestpractices/deploymentconfig-triggers-notset)_

## Pod hostnetwork not set

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/DaemonSet apps/Deployment apps/StatefulSet

Pods which require 'spec.hostNetwork' should be limited due to security concerns.

### Rego

```rego
package ocp.bestpractices.pod_hostnetwork

import data.lib.konstraint
import data.lib.openshift

violation[msg] {
  openshift.is_workload_kind

  pod := openshift.pods[_]
  pod.spec.hostNetwork
  obj := konstraint.object

  msg := konstraint.format(sprintf("%s/%s: hostNetwork is present which gives the pod access to the loopback device, services listening on localhost, and could be used to snoop on network activity of other pods on the same node.", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/bestpractices/pod-hostnetwork](policy/ocp/bestpractices/pod-hostnetwork)_

## Pod replica below 1

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/Deployment

Never run a single Pod individually.
See: Fault tolerance -> https://learnk8s.io/production-best-practices#application-development

### Rego

```rego
package ocp.bestpractices.pod_replicas_below_one

import data.lib.konstraint
import data.lib.openshift

violation[msg] {
  openshift.is_workload_kind

  obj := konstraint.object
  obj.spec.replicas <= 1

  msg := konstraint.format(sprintf("%s/%s: replicas is %d - expected replicas to be greater than 1 for HA guarantees.", [obj.kind, obj.metadata.name, obj.spec.replicas]))
}
```

_source: [policy/ocp/bestpractices/pod-replicas-below-one](policy/ocp/bestpractices/pod-replicas-below-one)_

## Pod replica is not odd

**Severity:** violation

**Resources:** apps.openshift.io/DeploymentConfig apps/Deployment

Pods should be run with a replica which is odd, i.e.: 3, 5, 7, etc, for HA guarantees.
See: Fault tolerance -> https://learnk8s.io/production-best-practices#application-development

### Rego

```rego
package ocp.bestpractices.pod_replicas_not_odd

import data.lib.konstraint
import data.lib.openshift

violation[msg] {
  openshift.is_workload_kind

  obj := konstraint.object
  obj.spec.replicas % 2 == 0

  msg := konstraint.format(sprintf("%s/%s: replicas is %d - expected an odd number for HA guarantees.", [obj.kind, obj.metadata.name, obj.spec.replicas]))
}
```

_source: [policy/ocp/bestpractices/pod-replicas-not-odd](policy/ocp/bestpractices/pod-replicas-not-odd)_

## RoleBinding has apiGroup set

**Severity:** violation

**Resources:** rbac.authorization.k8s.io/RoleBinding

Migrating from 3.11 to 4.x requires the 'roleRef.apiGroup' to be set.

### Rego

```rego
package ocp.bestpractices.rolebinding_roleref_apigroup_notset

import data.lib.konstraint
import data.lib.kubernetes

violation[msg] {
  kubernetes.is_rolebinding

  obj := konstraint.object
  konstraint.missing_field(obj.roleRef, "apiGroup")

  msg := konstraint.format(sprintf("%s/%s: RoleBinding roleRef.apiGroup key is null, use rbac.authorization.k8s.io instead.", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/bestpractices/rolebinding-roleref-apigroup-notset](policy/ocp/bestpractices/rolebinding-roleref-apigroup-notset)_

## RoleBinding has kind set

**Severity:** violation

**Resources:** rbac.authorization.k8s.io/RoleBinding

Migrating from 3.11 to 4.x requires the 'roleRef.kind' to be set.

### Rego

```rego
package ocp.bestpractices.rolebinding_roleref_kind_notset

import data.lib.konstraint
import data.lib.kubernetes

violation[msg] {
  kubernetes.is_rolebinding

  obj := konstraint.object
  konstraint.missing_field(obj.roleRef, "kind")

  msg := konstraint.format(sprintf("%s/%s: RoleBinding roleRef.kind key is null, use ClusterRole or Role instead.", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/bestpractices/rolebinding-roleref-kind-notset](policy/ocp/bestpractices/rolebinding-roleref-kind-notset)_

## BuildConfig no longer served by v1

**Severity:** violation

**Resources:** v1/BuildConfig

OCP4.x expects build.openshift.io/v1.

### Rego

```rego
package ocp.deprecated.ocp3_11.buildconfig_v1

import data.lib.konstraint

violation[msg] {
  obj := konstraint.object
  lower(obj.apiVersion) == "v1"
  lower(obj.kind) == "buildconfig"

  msg := konstraint.format(sprintf("%s/%s: API v1 for BuildConfig is no longer served by default, use build.openshift.io/v1 instead.", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/deprecated/3_11/buildconfig-v1](policy/ocp/deprecated/3_11/buildconfig-v1)_

## DeploymentConfig no longer served by v1

**Severity:** violation

**Resources:** v1/DeploymentConfig

OCP4.x expects apps.openshift.io/v1.

### Rego

```rego
package ocp.deprecated.ocp3_11.deploymentconfig_v1

import data.lib.konstraint

violation[msg] {
  obj := konstraint.object
  lower(obj.apiVersion) == "v1"
  lower(obj.kind) == "deploymentconfig"

  msg := konstraint.format(sprintf("%s/%s: API v1 for DeploymentConfig is no longer served by default, use apps.openshift.io/v1 instead.", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/deprecated/3_11/deploymentconfig-v1](policy/ocp/deprecated/3_11/deploymentconfig-v1)_

## ImageStream no longer served by v1

**Severity:** violation

**Resources:** v1/ImageStream

OCP4.x expects image.openshift.io/v1.

### Rego

```rego
package ocp.deprecated.ocp3_11.imagestream_v1

import data.lib.konstraint

violation[msg] {
  obj := konstraint.object
  lower(obj.apiVersion) == "v1"
  lower(obj.kind) == "imagestream"

  msg := konstraint.format(sprintf("%s/%s: API v1 for ImageStream is no longer served by default, use image.openshift.io/v1 instead.", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/deprecated/3_11/imagestream-v1](policy/ocp/deprecated/3_11/imagestream-v1)_

## ProjectRequest no longer served by v1

**Severity:** violation

**Resources:** v1/ProjectRequest

OCP4.x expects project.openshift.io/v1.

### Rego

```rego
package ocp.deprecated.ocp3_11.projectrequest_v1

import data.lib.konstraint

violation[msg] {
  obj := konstraint.object
  lower(obj.apiVersion) == "v1"
  lower(obj.kind) == "projectrequest"

  msg := konstraint.format(sprintf("%s/%s: API v1 for ProjectRequest is no longer served by default, use project.openshift.io/v1 instead.", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/deprecated/3_11/projectrequest-v1](policy/ocp/deprecated/3_11/projectrequest-v1)_

## RoleBinding no longer served by v1

**Severity:** violation

**Resources:** v1/RoleBinding

OCP4.x expects rbac.authorization.k8s.io/v1

### Rego

```rego
package ocp.deprecated.ocp3_11.rolebinding_v1

import data.lib.konstraint

violation[msg] {
  obj := konstraint.object
  lower(obj.apiVersion) == "v1"
  lower(obj.kind) == "rolebinding"

  msg := konstraint.format(sprintf("%s/%s: API v1 for RoleBinding is no longer served by default, use rbac.authorization.k8s.io/v1 instead.", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/deprecated/3_11/rolebinding-v1](policy/ocp/deprecated/3_11/rolebinding-v1)_

## Route no longer served by v1

**Severity:** violation

**Resources:** v1/Route

OCP4.x expects route.openshift.io/v1.

### Rego

```rego
package ocp.deprecated.ocp3_11.route_v1

import data.lib.konstraint

violation[msg] {
  obj := konstraint.object
  lower(obj.apiVersion) == "v1"
  lower(obj.kind) == "route"

  msg := konstraint.format(sprintf("%s/%s: API v1 for Route is no longer served by default, use route.openshift.io/v1 instead.", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/deprecated/3_11/route-v1](policy/ocp/deprecated/3_11/route-v1)_

## SecurityContextConstraints no longer served by v1

**Severity:** violation

**Resources:** v1/SecurityContextConstraints

OCP4.x expects security.openshift.io/v1.

### Rego

```rego
package ocp.deprecated.ocp3_11.securitycontextconstraints_v1

import data.lib.konstraint

violation[msg] {
  obj := konstraint.object
  lower(obj.apiVersion) == "v1"
  lower(obj.kind) == "securitycontextconstraints"

  msg := konstraint.format(sprintf("%s/%s: API v1 for SecurityContextConstraints is no longer served by default, use security.openshift.io/v1 instead.", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/deprecated/3_11/securitycontextconstraints-v1](policy/ocp/deprecated/3_11/securitycontextconstraints-v1)_

## Template no longer served by v1

**Severity:** violation

**Resources:** v1/Template

OCP4.x expects template.openshift.io/v1.

### Rego

```rego
package ocp.deprecated.ocp3_11.template_v1

import data.lib.konstraint

violation[msg] {
  obj := konstraint.object
  lower(obj.apiVersion) == "v1"
  lower(obj.kind) == "template"

  msg := konstraint.format(sprintf("%s/%s: API v1 for Template is no longer served by default, use template.openshift.io/v1 instead.", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/deprecated/3_11/template-v1](policy/ocp/deprecated/3_11/template-v1)_

## BuildConfig exposeDockerSocket deprecated

**Severity:** violation

**Resources:** build.openshift.io/BuildConfig

'spec.strategy.customStrategy.exposeDockerSocket' is no longer supported by BuildConfig.
See: https://docs.openshift.com/container-platform/4.1/release_notes/ocp-4-1-release-notes.html#ocp-41-deprecated-features

### Rego

```rego
package ocp.deprecated.ocp4_1.buildconfig_custom_strategy

import data.lib.konstraint

violation[msg] {
  obj := konstraint.object
  lower(obj.apiVersion) == "build.openshift.io/v1"
  lower(obj.kind) == "buildconfig"

  obj.spec.strategy.customStrategy.exposeDockerSocket

  msg := konstraint.format(sprintf("%s/%s: 'spec.strategy.customStrategy.exposeDockerSocket' is deprecated. If you want to continue using custom builds, you should replace your Docker invocations with Podman or Buildah.", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/deprecated/4_1/buildconfig-custom-strategy](policy/ocp/deprecated/4_1/buildconfig-custom-strategy)_

## authorization openshift io is deprecated

**Severity:** violation

**Resources:** authorization.openshift.io/ClusterRole authorization.openshift.io/ClusterRoleBinding authorization.openshift.io/Role authorization.openshift.io/RoleBinding

From OCP4.2 onwards, you should migrate from 'authorization.openshift.io' to rbac.authorization.k8s.io/v1.
See: https://docs.openshift.com/container-platform/4.2/release_notes/ocp-4-2-release-notes.html#ocp-4-2-deprecated-features

### Rego

```rego
package ocp.deprecated.ocp4_2.authorization_openshift

import data.lib.konstraint

violation[msg] {
  obj := konstraint.object
  contains(lower(obj.apiVersion), "authorization.openshift.io")

  msg := konstraint.format(sprintf("%s/%s: API authorization.openshift.io for ClusterRole, ClusterRoleBinding, Role and RoleBinding is deprecated, use rbac.authorization.k8s.io/v1 instead.", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/deprecated/4_2/authorization-openshift](policy/ocp/deprecated/4_2/authorization-openshift)_

## automationbroker io v1alpha1 is deprecated

**Severity:** violation

**Resources:** automationbroker.io/Bundle automationbroker.io/BundleBinding automationbroker.io/BundleInstance

'automationbroker.io/v1alpha1' is deprecated in OCP 4.2 and removed in 4.4.
See: https://docs.openshift.com/container-platform/4.2/release_notes/ocp-4-2-release-notes.html#ocp-4-2-deprecated-features
See: https://docs.openshift.com/container-platform/4.4/release_notes/ocp-4-4-release-notes.html#ocp-4-4-deprecated-removed-features

### Rego

```rego
package ocp.deprecated.ocp4_2.automationbroker_v1alpha1

import data.lib.konstraint

violation[msg] {
  obj := konstraint.object
  contains(lower(obj.apiVersion), "automationbroker.io/v1alpha1")

  msg := konstraint.format(sprintf("%s/%s: automationbroker.io/v1alpha1 is deprecated.", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/deprecated/4_2/automationbroker-v1alpha1](policy/ocp/deprecated/4_2/automationbroker-v1alpha1)_

## operators coreos com v1 CatalogSourceConfigs is deprecated

**Severity:** violation

**Resources:** operators.coreos.com/CatalogSourceConfigs

'operators.coreos.com/v1:CatalogSourceConfigs' is deprecated in OCP 4.2 and removed in 4.5.
See: https://docs.openshift.com/container-platform/4.2/release_notes/ocp-4-2-release-notes.html#ocp-4-2-deprecated-features
See: https://docs.openshift.com/container-platform/4.5/release_notes/ocp-4-5-release-notes.html#ocp-4-5-deprecated-removed-features

### Rego

```rego
package ocp.deprecated.ocp4_2.catalogsourceconfigs_v1

import data.lib.konstraint

violation[msg] {
  obj := konstraint.object
  contains(lower(obj.apiVersion), "operators.coreos.com/v1")
  lower(obj.kind) == "catalogsourceconfigs"

  msg := konstraint.format(sprintf("%s/%s: operators.coreos.com/v1 is deprecated.", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/deprecated/4_2/catalogsourceconfigs-v1](policy/ocp/deprecated/4_2/catalogsourceconfigs-v1)_

## operators coreos com v2 CatalogSourceConfigs is deprecated

**Severity:** violation

**Resources:** operators.coreos.com/CatalogSourceConfigs

'operators.coreos.com/v2:CatalogSourceConfigs' is deprecated in OCP 4.2 and removed in 4.5.
See: https://docs.openshift.com/container-platform/4.2/release_notes/ocp-4-2-release-notes.html#ocp-4-2-deprecated-features
See: https://docs.openshift.com/container-platform/4.5/release_notes/ocp-4-5-release-notes.html#ocp-4-5-deprecated-removed-features

### Rego

```rego
package ocp.deprecated.ocp4_2.catalogsourceconfigs_v2

import data.lib.konstraint

violation[msg] {
  obj := konstraint.object
  contains(lower(obj.apiVersion), "operators.coreos.com/v2")
  lower(obj.kind) == "catalogsourceconfigs"

  msg := konstraint.format(sprintf("%s/%s: operators.coreos.com/v2 is deprecated.", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/deprecated/4_2/catalogsourceconfigs-v2](policy/ocp/deprecated/4_2/catalogsourceconfigs-v2)_

## operators coreos com v1 OperatorSource is deprecated

**Severity:** violation

**Resources:** operators.coreos.com/OperatorSource

'operators.coreos.com/v1:OperatorSource' is deprecated in OCP 4.2 and will be removed in a future version.
See: https://docs.openshift.com/container-platform/4.2/release_notes/ocp-4-2-release-notes.html#ocp-4-2-deprecated-features

### Rego

```rego
package ocp.deprecated.ocp4_2.operatorsources_v1

import data.lib.konstraint

violation[msg] {
  obj := konstraint.object
  contains(lower(obj.apiVersion), "operators.coreos.com/v1")
  lower(obj.kind) == "operatorsource"

  msg := konstraint.format(sprintf("%s/%s: operators.coreos.com/v1 is deprecated.", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/deprecated/4_2/operatorsources-v1](policy/ocp/deprecated/4_2/operatorsources-v1)_

## osb openshift io v1 is deprecated

**Severity:** violation

**Resources:** osb.openshift.io/TemplateServiceBroker osb.openshift.io/AutomationBroker

'osb.openshift.io/v1' is deprecated in OCP 4.2 and removed in 4.5.
See: https://docs.openshift.com/container-platform/4.2/release_notes/ocp-4-2-release-notes.html#ocp-4-2-deprecated-features
See: https://docs.openshift.com/container-platform/4.5/release_notes/ocp-4-5-release-notes.html#ocp-4-5-deprecated-removed-features

### Rego

```rego
package ocp.deprecated.ocp4_2.osb_v1

import data.lib.konstraint

violation[msg] {
  obj := konstraint.object
  contains(lower(obj.apiVersion), "osb.openshift.io/v1")

  msg := konstraint.format(sprintf("%s/%s: osb.openshift.io/v1 is deprecated.", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/deprecated/4_2/osb-v1](policy/ocp/deprecated/4_2/osb-v1)_

## servicecatalog k8s io v1beta1 is deprecated

**Severity:** violation

**Resources:** servicecatalog.k8s.io/ClusterServiceBroker servicecatalog.k8s.io/ClusterServiceClass servicecatalog.k8s.io/ClusterServicePlan servicecatalog.k8s.io/ServiceInstance servicecatalog.k8s.io/ServiceBinding

'servicecatalog.k8s.io/v1beta1' is deprecated in OCP 4.2 and removed in 4.5.
See: https://docs.openshift.com/container-platform/4.2/release_notes/ocp-4-2-release-notes.html#ocp-4-2-deprecated-features
See: https://docs.openshift.com/container-platform/4.5/release_notes/ocp-4-5-release-notes.html#ocp-4-5-deprecated-removed-features

### Rego

```rego
package ocp.deprecated.ocp4_2.servicecatalog_v1beta1

import data.lib.konstraint

violation[msg] {
  obj := konstraint.object
  contains(lower(obj.apiVersion), "servicecatalog.k8s.io/v1beta1")

  msg := konstraint.format(sprintf("%s/%s: servicecatalog.k8s.io/v1beta1 is deprecated.", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/deprecated/4_2/servicecatalog-v1beta1](policy/ocp/deprecated/4_2/servicecatalog-v1beta1)_

## BuildConfig jenkinsPipelineStrategy is deprecated

**Severity:** violation

**Resources:** build.openshift.io/BuildConfig

'spec.strategy.jenkinsPipelineStrategy' is no longer supported by BuildConfig.
See: https://docs.openshift.com/container-platform/4.3/release_notes/ocp-4-3-release-notes.html#ocp-4-3-deprecated-features

### Rego

```rego
package ocp.deprecated.ocp4_3.buildconfig_jenkinspipeline_strategy

import data.lib.konstraint

violation[msg] {
  obj := konstraint.object
  lower(obj.apiVersion) == "build.openshift.io/v1"
  lower(obj.kind) == "buildconfig"

  obj.spec.strategy.jenkinsPipelineStrategy

  msg := konstraint.format(sprintf("%s/%s: 'spec.strategy.jenkinsPipelineStrategy' is deprecated. Use Jenkinsfiles directly on Jenkins or OpenShift Pipelines instead.", [obj.kind, obj.metadata.name]))
}
```

_source: [policy/ocp/deprecated/4_3/buildconfig-jenkinspipeline-strategy](policy/ocp/deprecated/4_3/buildconfig-jenkinspipeline-strategy)_

## Deployment has a matching PodDisruptionBudget

**Severity:** violation

**Resources:** apps/Deployment

All Deployments should have matching PodDisruptionBudget, via 'spec.template.metadata.labels', to provide HA guarantees.
See: Fault tolerance -> https://learnk8s.io/production-best-practices#application-development
See: https://kubernetes.io/docs/tasks/run-application/configure-pdb/

### Rego

```rego
package ocp.requiresinventory.deployment_has_matching_poddisruptionbudget

import data.lib.konstraint

violation[msg] {
  konstraint.is_deployment

  deployment := konstraint.object

  not deployment_has_matching_poddisruptionbudget(deployment, data.inventory.namespace[deployment.metadata.namespace])

  msg := konstraint.format(sprintf("%s/%s does not have a policy/v1beta1:PodDisruptionBudget or its selector labels dont match. See: https://kubernetes.io/docs/tasks/run-application/configure-pdb/#specifying-a-poddisruptionbudget", [deployment.kind, deployment.metadata.name]))
}

deployment_has_matching_poddisruptionbudget(deployment, manifests) {
  cached := manifests["policy/v1beta1"]["PodDisruptionBudget"]
  current := cached[_]

  deployment.spec.template.metadata.labels == current.spec.selector.matchLabels
}
```

_source: [policy/ocp/requiresinventory/deployment-has-matching-poddisruptionbudget](policy/ocp/requiresinventory/deployment-has-matching-poddisruptionbudget)_

## Deployment has matching PersistentVolumeClaim

**Severity:** violation

**Resources:** apps/Deployment

If Deployment has 'spec.template.spec.volumes.persistentVolumeClaim' set, there should be matching PersistentVolumeClaim.
If not, this would suggest a mistake.

### Rego

```rego
package ocp.requiresinventory.deployment_has_matching_pvc

import data.lib.konstraint

violation[msg] {
  konstraint.is_deployment

  deployment := konstraint.object
  deployment.spec.template.spec.volumes[_].persistentVolumeClaim

  not deployment_has_matching_persistentvolumeclaim(deployment, data.inventory.namespace[deployment.metadata.namespace])

  msg := konstraint.format(sprintf("%s/%s has persistentVolumeClaim in its spec.template.spec.volumes but could not find corrasponding v1:PersistentVolumeClaim.", [deployment.kind, deployment.metadata.name]))
}

deployment_has_matching_persistentvolumeclaim(deployment, manifests) {
  cached := manifests["v1"]["PersistentVolumeClaim"]
  current := cached[_]

  deployment.spec.template.spec.volumes[_].persistentVolumeClaim.claimName == current.metadata.name
}
```

_source: [policy/ocp/requiresinventory/deployment-has-matching-pvc](policy/ocp/requiresinventory/deployment-has-matching-pvc)_

## Deployment has a matching Service

**Severity:** violation

**Resources:** apps/Deployment

All Deployments should have matching Service, via 'spec.template.metadata.labels'.
Deployments without a Service are not accessible and should be questioned as to why.

### Rego

```rego
package ocp.requiresinventory.deployment_has_matching_service

import data.lib.konstraint

violation[msg] {
  konstraint.is_deployment

  deployment := konstraint.object

  not deployment_labels_matches_service_selector(deployment, data.inventory.namespace[deployment.metadata.namespace])

  msg := konstraint.format(sprintf("%s/%s does not have a v1:Service or its selector labels dont match. See: https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#service-and-replicationcontroller", [deployment.kind, deployment.metadata.name]))
}

deployment_labels_matches_service_selector(deployment, manifests) {
  cached := manifests["v1"]["Service"]
  current := cached[_]

  deployment.spec.template.metadata.labels == current.spec.selector
}
```

_source: [policy/ocp/requiresinventory/deployment-has-matching-service](policy/ocp/requiresinventory/deployment-has-matching-service)_

## Deployment has matching ServiceAccount

**Severity:** violation

**Resources:** apps/Deployment

If Deployment has 'spec.serviceAccountName' set, there should be matching ServiceAccount.
If not, this would suggest a mistake.

### Rego

```rego
package ocp.requiresinventory.deployment_has_matching_serviceaccount

import data.lib.konstraint

violation[msg] {
  konstraint.is_deployment

  deployment := konstraint.object
  deployment.spec.template.spec.serviceAccountName

  not deployment_has_matching_serviceaccount(deployment, data.inventory.namespace[deployment.metadata.namespace])

  msg := konstraint.format(sprintf("%s/%s has spec.serviceAccountName '%s' but could not find corrasponding v1:ServiceAccount.", [deployment.kind, deployment.metadata.name, deployment.spec.template.spec.serviceAccountName]))
}

deployment_has_matching_serviceaccount(deployment, manifests) {
  cached := manifests["v1"]["ServiceAccount"]
  current := cached[_]

  deployment.spec.template.spec.serviceAccountName == current.metadata.name
}
```

_source: [policy/ocp/requiresinventory/deployment-has-matching-serviceaccount](policy/ocp/requiresinventory/deployment-has-matching-serviceaccount)_

## Service has matching ServiceMonitor

**Severity:** violation

**Resources:** core/Service

All Service should have a matching ServiceMonitor, via 'spec.selector'.
Service without a ServiceMonitor are not being monitored and should be questioned as to why.

### Rego

```rego
package ocp.requiresinventory.service_has_matching_servicenonitor

import data.lib.konstraint

violation[msg] {
  konstraint.is_service

  service := konstraint.object

  not service_has_matching_servicemonitor(service, data.inventory.namespace[service.metadata.namespace])

  msg := konstraint.format(sprintf("%s/%s does not have a monitoring.coreos.com/v1:ServiceMonitor or its selector labels dont match. See: https://docs.openshift.com/container-platform/4.4/monitoring/monitoring-your-own-services.html", [service.kind, service.metadata.name]))
}

service_has_matching_servicemonitor(service, manifests) {
  cached := manifests["monitoring.coreos.com/v1"]["ServiceMonitor"]
  current := cached[_]

  service.spec.selector == current.spec.selector.matchLabels
}
```

_source: [policy/ocp/requiresinventory/service-has-matching-servicemonitor](policy/ocp/requiresinventory/service-has-matching-servicemonitor)_

## Image contains expected SHA in history.

**Severity:** violation

**Resources:** redhat-cop.github.com/PodmanHistory

Most images are built from a subset of authorised base images in a company,
this policy allows enforcement of that policy by checking for an expected SHA.

### Rego

```rego
package podman.history.contains_layer

violation[msg] {
  lower(input.apiVersion) == "redhat-cop.github.com/v1"
  lower(input.kind) == "podmanhistory"

  not image_history_contains_layer(input.items)

  msg := sprintf("%s: did not find expected SHA.", [input.image])
}

image_history_contains_layer(layers) {
  layers[_].id == "cd343f0d83042932fa992e095cd4a93a89a3520873f99b0e15fde69eb46e7e10"
}
```

_source: [policy/podman/history/contains-layer](policy/podman/history/contains-layer)_

## Image size is not greater than an expected value

**Severity:** violation

**Resources:** redhat-cop.github.com/PodmanImages

Typically, the "smaller the better" rule applies to images so lets enforce that.

### Rego

```rego
package podman.images.image_size_not_greater_than

import data.lib.memory

violation[msg] {
  lower(input.apiVersion) == "redhat-cop.github.com/v1"
  lower(input.kind) == "podmanimages"

  #NOTE: upperBound is an arbitrary number and it should be changed to what your company believes is the correct policy
  upperBound := 512

  image := input.items[_]
  sizeInMb := image.size / memory.mb
  sizeInMb > upperBound

  msg := sprintf("%s: has a size of '%fMi', which is greater than '%dMi' limit.", [input.image, sizeInMb, upperBound])
}
```

_source: [policy/podman/images/image-size-not-greater-than](policy/podman/images/image-size-not-greater-than)_
