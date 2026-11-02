# EKS Upgrade Report: 1.31 -> 1.32

## Why

The repo pinned EKS **1.31**. The newest supported EKS version is **1.37**.
EKS upgrades **one minor version at a time**, so this PR steps to **1.32**.
Repeat this process (1.32 -> 1.33 -> ...) to reach 1.37.

## What this PR changes

- `eks:version` and nodegroup version: `1.31` -> `1.32`.
- Add-ons bumped to versions that support 1.32:
  - `vpc-cni`: `v1.18.5-eksbuild.1` -> `v1.19.2-eksbuild.1`
  - `coredns`: `v1.11.3-eksbuild.1` -> `v1.11.4-eksbuild.1`
  - `kube-proxy`: `v1.31.0-eksbuild.5` -> `v1.32.0-eksbuild.2`

> Add-on versions here are reasonable defaults. Before applying, confirm the
> exact latest build for 1.32 with:
> `aws eks describe-addon-versions --kubernetes-version 1.32 --addon-name <name>`.

## Upgrade checklist (do in order)

1. **Bump the version pin** — done in this PR (`eks-version.yaml`).
2. **Control plane first, then nodes.** Upgrade the EKS control plane to 1.32,
   then node groups, then add-ons. Never skip a minor version.
3. **Check add-on compatibility** for 1.32:
   - vpc-cni, coredns, kube-proxy, aws-ebs-csi-driver.
   - Verify each with `aws eks describe-addon-versions --kubernetes-version 1.32`.
4. **Node AMI / launch template.** Make sure the AMI (or launch template) uses a
   1.32 EKS-optimized image. Managed node groups pick it up on update; custom
   launch templates must be bumped manually.
5. **Deprecated / removed APIs to check before 1.32:**
   - `flowcontrol.apiserver.k8s.io/v1beta3` (`FlowSchema`,
     `PriorityLevelConfiguration`) is **removed in 1.32** — migrate to `v1`.
   - `anonymous` auth config changes and the removal of several long-deprecated
     feature gates land in 1.32 — confirm none are referenced.
   - Confirm no use of already-removed betas: `policy/v1beta1 PodSecurityPolicy`
     (gone since 1.25), `batch/v1beta1 CronJob`, `autoscaling/v2beta2 HPA`.
   - Run a scan: `kube-no-trouble (kubent)` or `pluto detect-files` against your
     manifests.
6. **Test in non-prod first.** Apply to a staging cluster, confirm workloads and
   add-ons are healthy, then promote.

## Application compatibility

The only application in this repo is `sample-react-app/` — a static React 18 +
Vite single-page app.

- **No Kubernetes API usage.** The app does not call the Kubernetes API, use
  client libraries, CRDs, or in-cluster service accounts. It is served as static
  files, so removed/deprecated K8s APIs do not affect it.
- **Dependencies** (`react ^18.3.1`, `react-dom ^18.3.1`, `vite ^5.3.4`,
  `@vitejs/plugin-react ^4.3.1`) are unrelated to the cluster version and need no
  change for the 1.31 -> 1.32 bump.
- **Verdict:** no application changes are required to support EKS 1.32.

## This repo

This is a sample repo: the only operational file is `eks-version.yaml`
(the version pin the auto-upgrade job reads). `sample-react-app/` is unrelated
app code and is not touched.
