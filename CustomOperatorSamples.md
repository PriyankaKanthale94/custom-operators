# Custom Operators for OpenShift Virtualization in Disconnected Clusters

## 1. Image Mirroring Lifecycle Operator (ImageMirror)

### Problem

In a disconnected environment, mirroring OpenShift releases, operator catalogs, and application images to an internal registry is often a manual process involving `oc-mirror`, configuration management, and troubleshooting.

### Why This Fits an Operator (OCP)

An operator can provide a declarative, Kubernetes-native way to define required mirrored content and continuously manage its synchronization.

### Operator Responsibilities

- Reconcile an `ImageMirror` CR defining releases, catalogs, operators, application images, and the destination registry.
- Trigger mirroring on CR changes, schedules, or manual requests.
- Manage the `oc-mirror` workflow.
- Apply generated resources such as `ImageDigestMirrorSet` (IDMS), `ImageTagMirrorSet` (ITMS), and `CatalogSource`.
- Track synchronization status and history.
- Handle retries and failures and generate alerts.
- Provide a consolidated `.status` with synchronization state, last successful sync, and errors.

### Not Covered by the Product

`oc-mirror` performs the mirroring operation but does not provide continuous reconciliation, scheduling, monitoring, and organization-specific lifecycle management.

---

## 2. Upgrade Readiness and Governance Operator (UpgradeReadiness)

### Problem

OpenShift upgrades require technical validation and organizational approval. In a disconnected enterprise environment, teams must manually check cluster health, alerts, nodes, workload disruption risks, internal image availability, maintenance windows, and change approvals.

### Why This Fits an Operator (OCP)

A custom operator can provide a single declarative framework to continuously evaluate whether a cluster is ready and approved for an upgrade. It does not replace the native OpenShift Cluster Version Operator (CVO).

### Operator Responsibilities

- Reconcile an `UpgradeReadiness` CR with the target version and upgrade policies.
- Perform pre-upgrade checks:
  - Verify that ClusterOperators, nodes, and MachineConfigPools are healthy.
  - Check for critical alerts.
  - Identify PodDisruptionBudget (PDB) and workload disruption risks.
- Validate disconnected readiness:
  - Confirm that the target release and required images are available internally.
  - Confirm that required catalogs are available.
- Enforce governance:
  - Maintenance windows.
  - Change approvals.
  - Upgrade freeze periods.
- Provide a consolidated status: `Ready`, `Blocked`, `Warning`, or `ApprovalPending`.
- Perform post-upgrade health validation.

### CVO Responsibilities

The native CVO performs the actual OpenShift platform upgrade and reconciles the cluster to the target version.

### Not Covered by the Product

The CVO performs upgrades but does not provide organization-specific governance, approval workflows, disconnected readiness checks, or custom workload risk management.

---

## 3. Cluster Configuration Drift Detection Operator (ClusterBaseline)

### Problem

The Machine Config Operator (MCO), CVO, and Operator Lifecycle Manager (OLM) detect drift only for resources they manage. There is no centralized mechanism to independently detect and classify configuration changes across an organization's approved cluster baseline, especially changes made outside the approved GitOps process.

### Why This Fits an Operator (OCP)

A custom operator can continuously monitor the live cluster and independently compare selected resources against an approved baseline managed through GitOps.

### Operator Responsibilities

- Reconcile a `ClusterBaseline` CR that references the approved GitOps baseline.
- Continuously compare live cluster resources with the approved desired state.
- Detect changes made outside the approved management process, such as `oc edit` or direct API calls.
- Detect drift across selected custom resources (CRs), RBAC, NetworkPolicies, operator configurations, and application configurations.
- Categorize drift as `Informational`, `Warning`, or `Critical`.
- Generate centralized drift and compliance reports.
- Optionally remediate unauthorized changes after appropriate policy or approval checks.
- Surface a consolidated drift status.

### Not Covered by the Product

Existing OpenShift operators reconcile resources they own, while GitOps applies the desired configuration. Neither independently provides organization-wide drift detection, classification, compliance reporting, and governance for selected resources.

---

## 4. Continuous Security Hardening Operator (SecurityBaseline)

### Problem

The Compliance Operator performs scheduled compliance scans against predefined profiles such as CIS, STIG, and PCI-DSS. However, security requirements and cluster workloads continuously change, and organizations may need continuous, event-driven enforcement of their own security standards.

### Why This Fits an Operator (OCP)

A custom operator can continuously monitor cluster events and enforce organization-specific security baselines whenever new resources or configuration changes are detected.

### Operator Responsibilities

- Reconcile a `SecurityBaseline` CR containing organization-specific security requirements.
- Trigger validation when:
  - New nodes join.
  - New namespaces are created.
  - New operators are installed.
- Enforce custom security policies, such as:
  - Required labels and annotations.
  - Namespace-specific pod security requirements.
  - Custom configuration baselines.
- Automatically apply NetworkPolicy baselines to new namespaces, such as default-deny policies with approved allow rules.
- Validate TLS configuration, certificate chains, and ingress security.
- Monitor operator security posture, including:
  - Appropriate RBAC.
  - Required NetworkPolicies.
  - Image security requirements.
- Generate centralized security posture and compliance reports.

### Not Covered by the Product

Existing compliance tools provide profile-based scanning, but they do not provide continuous, event-driven enforcement of all organization-specific security policies across the cluster.

---



[Updating a disconnected cluster using the OpenShift Update Service (OSUS)](https://docs.redhat.com/en/documentation/openshift_container_platform/4.21/html-single/disconnected_environments/index#updating-disconnected-cluster-osus)
