💼 Case Study: DevSecOps Pipeline Proposal During Velera–PSCU Merger

Context:
During my internship at Velera, a financial management company for credit unions, the organization was merging with PSCU. As part of the merger, the engineering teams were migrating all repositories from Bitbucket to GitHub and re-evaluating their CI/CD practices.

Given the nature of the business — handling sensitive member and transaction data — I proposed building a DevSecOps pipeline to integrate security at every stage of the software delivery lifecycle, ensuring compliance with financial security standards and minimizing post-deployment vulnerabilities.

🧩 My Contribution — Designing the Secure CI/CD Pipeline

I proposed a pipeline in Jenkins + GitHub that followed the shift-left and shift-right security approach, embedding security from developer commit to production deployment.

1️⃣ Developer Stage – Shift Left Security

Implemented Git Hooks with Talisman to prevent accidental commits of secrets, API keys, or credentials.

This helped reduce secret exposure before code even reached the repository.

2️⃣ Build Stage – Quality and Static Analysis

The pipeline ran unit tests and mutation testing (PIT) to ensure testing depth and quality.

Integrated SonarQube for SAST (Static Application Security Testing) to detect code vulnerabilities early.

Configured quality gates so that builds failed automatically if code coverage or critical issues fell below threshold.

3️⃣ Dependency and Image Scanning

Used OWASP Dependency-Check to identify open-source library vulnerabilities (CVEs).

Built Docker images and scanned them with Trivy, covering both OS-level and application-level packages.

Integrated OPA Conftest and Kubesec to enforce secure configuration policies on Dockerfiles and Kubernetes manifests (e.g., no privileged containers, read-only root filesystem).

4️⃣ Deployment and Runtime Security

Automated Kubernetes deployment with rollback checks — if health checks failed, Jenkins triggered an automatic rollback to the previous stable version.

Performed DAST (Dynamic Application Security Testing) using OWASP ZAP against the running app via its OpenAPI spec to catch runtime issues like insecure headers or content-type mismatches.

5️⃣ Post-Deployment Monitoring & Compliance

Integrated Slack notifications so the team received real-time updates about build and scan results.

Proposed manual approval gates for production promotion to maintain separation of duties.

Added runtime and cluster security checks using Kube-bench (CIS Benchmark), Falco for threat detection, and Prometheus/Grafana for performance and anomaly monitoring.

🔒 Outcome & Impact

Created a repeatable, secure CI/CD framework that could be used across teams post-merger.

Helped demonstrate to leadership how DevSecOps practices align with PCI-DSS and SOC 2 compliance needs by ensuring traceability, secure configurations, and vulnerability awareness at every stage.

Shifted security from being a one-time audit to a continuous, automated process.

🎯 Summary (How to Say It in the Interview)

“During my internship at Velera, while the company was merging with PSCU and migrating to GitHub, I proposed a full DevSecOps pipeline.
The goal was to integrate security at every stage — from developer commits to runtime monitoring.
We used Talisman to block secrets, SonarQube and PIT for code quality, OWASP Dependency-Check and Trivy for vulnerability scanning, OPA Conftest and Kubesec for secure configuration, OWASP ZAP for dynamic testing, and Falco with Kube-bench for runtime and cluster-level security.
The pipeline also had Slack notifications, approval gates, and automatic rollback on failed deployments.
This approach not only improved security posture but also aligned with compliance frameworks like PCI-DSS by ensuring that every build was verifiable, auditable, and secure.”