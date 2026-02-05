# The Task

## DevOps Troubleshooting Assessment

### Task Overview

1. **Deploy** the provided `broken-manifest.yaml` to a local Kubernetes cluster (Minikube or Kind).
2. **Diagnose** why the pod enters a `CrashLoopBackOff` state.
3. **Root Cause Analysis (RCA):**
	- Provide a Markdown document explaining your findings.
4. **Submit a Fixed Manifest:**
	- Ensure the service remains stable after your fix.

---

## Candidate Submission Guide

- **Format:** Link to a public GitHub repository.
- **Access:** Add the hiring lead (as specified in the email) as a collaborator.
- **Evidence:**
  - The repository must show a **Failed** run (where security caught a vulnerability).
  - The repository must show a **Successful** run (after potentially updating the base image or ignoring non-fixed vulnerabilities if allowed).
