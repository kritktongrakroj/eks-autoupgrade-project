# EKS Upgrade Report: 1.30 -> 1.31

## Why

The repo pinned EKS **1.30**. The newest supported EKS version is **1.37**.
EKS upgrades **one minor version at a time**, so this PR steps to **1.31**.
Repeat this process (1.31 -> 1.32 -> ...) to reach 1.37.

## What this PR changes

- `eks:version` and nodegroup version: `1.30` -> `1.31`.
- Add-ons bumped to versions that support 1.31:
  - `vpc-cni`: `v1.18.3-eksbuild.2` -> `v1.18.5-eksbuild.1`
  - `coredns`: `v1.11.1-eksbuild.9` -> `v1.11.3-eksbuild.1`
  - `kube-proxy`: `v1.30.0-eksbuild.3` -> `v1.31.0-eksbuild.5`

> Add-on versions here are reasonable defaults. Before applying, confirm the
> exact latest build for 1.31 with:
> `aws eks describe-addon-versions --kubernetes-version 1.31 --addon-name <name>`.

## Upgrade checklist (do in order)

1. **Bump the version pin** — done in this PR (`eks-version.yaml`).
2. **Control plane first, then nodes.** Upgrade the EKS control plane to 1.31,
   then node groups, then add-ons. Never skip a minor version.
3. **Check add-on compatibility** for 1.31:
   - vpc-cni, coredns, kube-proxy, aws-ebs-csi-driver.
   - Verify each with `aws eks describe-addon-versions --kubernetes-version 1.31`.
4. **Node AMI / launch template.** Make sure the AMI (or launch template) uses a
   1.31 EKS-optimized image. Managed node groups pick it up on update; custom
   launch templates must be bumped manually.
5. **Deprecated / removed APIs to check before 1.31:**
   - `FlowSchema` / `PriorityLevelConfiguration`: `flowcontrol.apiserver.k8s.io/v1beta3` removed in 1.32 — migrate to `v1`.
   - Confirm no use of already-removed betas: `policy/v1beta1 PodSecurityPolicy` (gone since 1.25), `batch/v1beta1 CronJob`, `autoscaling/v2beta2 HPA`.
   - Run a scan: `kubectl` + `kube-no-trouble (kubent)` or `pluto detect-files` against your manifests.
6. **Test in non-prod first.** Apply to a staging cluster, confirm workloads and
   add-ons are healthy, then promote.

## This repo

This is a sample repo: the only operational file is `eks-version.yaml`
(the version pin the auto-upgrade job reads). `sample-react-app/` is unrelated
app code and is not touched.
