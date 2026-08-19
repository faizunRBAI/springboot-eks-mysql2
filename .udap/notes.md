# springboot-eks-mysql2 — working notes

## Project
- Blueprint: spring-boot-eks@1.0.1
- Database module: mysql
- Build tool: maven
- Cloud: AWS us-east-1, target: eks
- VCS: GitHub

## Status
- [x] Meta approved
- [x] Template selected (spring-boot-eks@1.0.1, database=mysql, build_tool=maven)
- [x] Design applied (apply_template scope=design)
- [x] Design confirmed
- [x] Plan approved
- [x] apply_template scope=files (39 files materialised)
- [x] MySQL-specific fixes applied:
  - infra/rds.tf: engine_version 8.0 → 8.4 (LTS; 8.0 EOL April 2026)
  - infra/versions.tf: Blueprint tag nodejs-eks → spring-boot-eks
  - infra/variables.tf: db_instance_class description fixed (was "Postgres")
- [x] validate_project: PASS (44 files)
- [x] test_project: PASSED — Semgrep 0 findings, Gitleaks clean, Checkov advisory
- [ ] create_repo_and_push
- [ ] deploy

## Key decisions
- MySQL 8.4 LTS (not 8.0 which is EOL)
- RDS db.t4g.micro, single-AZ, encrypted, 7-day backups
- sslMode=VERIFY_IDENTITY in JDBC URL — RDS CA bundle imported into JVM trust store in Dockerfile
- Flyway vendor-aware migrations: classpath:db/migration/{vendor} → picks mysql/V1__init.sql
- 7 parallel security gates block provision (Checkstyle, JUnit, Semgrep, Gitleaks, SBOM, licence, Trivy IaC)
- Destroy workflow deletes K8s Service first to release cloud LB before terraform destroy
- caching_sha2_password (MySQL 8.4 default) — com.mysql:mysql-connector-j supports it natively
- No mysql_native_password: disabled in 8.4, removed in 9.0

## Known pitfalls for this stack
- EKS control plane takes ~15min to provision; node group another ~5min
- RDS first provision ~10min; retry hits existing state cleanly via -reconfigure backend flags
- K8s LB must be deleted before terraform destroy (handled by custom destroy workflow)
- Don't add deletion_protection=true without also adding lifecycle prevent_destroy

## Sandbox gaps (expected, not project defects)
- mvn steps not rehearsed (setup-java@v4 runs in CI only)
- Trivy install needs sudo (GitHub runner is privileged, sandbox is not)
- These stages will run normally in CI
