DevOps Assessment – Secure CI/CD & Kubernetes Troubleshooting

This repository contains my solutions to the DevOps assessment. The focus was on practical debugging, security-first delivery, and DevSecOps best practices.

Task A – Kubernetes Troubleshooting

Problem

A Kubernetes Deployment entered a CrashLoopBackOff state shortly after startup.

Approach
	•	Used kubectl get pods -w and kubectl describe pod to observe restarts.
	•	Identified repeated OOMKilled events with exit code 137.
	•	Confirmed the root cause was unbounded memory growth combined with a low memory limit.

Resolution
	•	Removed the memory-leaking workload.
	•	Replaced it with a minimal HTTP server to simulate a healthy service.
	•	Added resource limits that match expected usage.

The final manifest runs stably without memory growth or restarts.
Details and commands are documented in task-a/RCA.md.


Task B – Secure CI/CD Pipeline

Pipeline Overview

The GitHub Actions workflow performs the following:
	1.	Builds a Docker image for the Node.js application
	2.	Scans the image with Trivy
	3.	Fails the build on HIGH or CRITICAL vulnerabilities
	4.	Pushes the image to GitHub Container Registry (GHCR) only if the scan passes

Security Decisions
	•	Used a multi-stage Docker build to separate build-time tooling from runtime.
	•	Installed dependencies in the build stage only.
	•	Ran the final container as a non-root user.
	•	Shipped a minimal runtime image to reduce attack surface and scanner noise.

Proof of Security Gate
	•	An initial pipeline run failed due to HIGH vulnerabilities in the base image tooling.
	•	After hardening the Dockerfile, a subsequent run passed and pushed the image successfully.
	•	Both runs are visible in the GitHub Actions history.



Pipeline Steps

- Checkout Code

Uses actions/checkout@v4
Fetches full repository


- Build Docker Image

Multi-stage build (build → runtime)
Loads image locally for scanning
Tags with SHA and branch name


- Security Scan (Trivy)

Scans for HIGH/CRITICAL vulnerabilities
Outputs SARIF format for GitHub Security tab
Fails build if vulnerabilities found (exit-code: 1)


- Push to GHCR (only if scan passes)

Authenticates with GITHUB_TOKEN
Pushes with multiple tags
Image available at ghcr.io/emmexgdc/devops-assessment