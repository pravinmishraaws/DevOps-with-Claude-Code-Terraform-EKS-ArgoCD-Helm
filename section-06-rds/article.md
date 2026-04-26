# Section 6 — Database (RDS MySQL)

## What You Build

A managed MySQL database on Amazon RDS:

| Resource | Details |
|----------|---------|
| RDS Instance | db.t4g.micro (free tier), MySQL 8.0 |
| Deployment | Single-AZ (both dev and prod — cost optimization) |
| Storage | 20 GB gp2, encrypted at rest |
| DB Subnet Group | Covers both VPC subnets |
| Parameter Group | MySQL 8.0 with custom settings |
| Security | Accessible only from EKS node security group on port 3306 |
| Secrets | Credentials stored in AWS Secrets Manager |

---

## Which Services Use MySQL

3 of the 8 services connect to the database:

| Service | Database | Tables |
|---------|----------|--------|
| Customers Service | `petclinic` | owners, pets, types (3 tables) |
| Vets Service | `petclinic` | vets, specialties, vet_specialties (3 tables) |
| Visits Service | `petclinic` | visits (1 table) |

**One shared database, 7 tables.** Schema files are in `spring-petclinic-microservices/src/main/resources/db/mysql/`.

The GenAI Service has JPA/MySQL dependencies but no schema files — it doesn't use the database in the standard setup.

---

## Database Credentials Pattern

Credentials are never hardcoded. The pattern used throughout this course:

1. Terraform creates an RDS instance with a random password
2. Terraform stores `{username, password, host, port, dbname}` in AWS Secrets Manager
3. External Secrets Operator (Section 8) syncs that secret to a Kubernetes Secret
4. Pods reference the Kubernetes Secret via environment variables

This is the production-correct approach. No plaintext credentials in code, Git, or pod specs.

---

## Schema Initialization

RDS creates an empty database. The schema (tables) needs to be loaded separately. Two approaches used in this course:

1. **Init container** — a pod init container runs `mysql < schema.sql` before the service starts
2. **One-time job** — a Kubernetes Job that runs the SQL files

Claude Code generates the strategy based on the Jira story acceptance criteria.

---

## Jira Stories (Epic E-5)

| Story | What it creates |
|-------|----------------|
| PetclinicPlatform22 | RDS module — instance, subnet group, parameter group |
| PetclinicPlatform23 | Database credentials in Secrets Manager |
| PetclinicPlatform24 | Database initialization strategy |
| PetclinicPlatform25 | Wire RDS module into dev environment |
| PetclinicPlatform26 | Deploy and verify dev RDS |
| PetclinicPlatform27 | Wire RDS module into prod environment |

---

## After This Section

- RDS MySQL instance running in AWS
- Database credentials stored in Secrets Manager
- `mysql -h {endpoint} -u petclinic -p` connects from an EKS pod

---

## External Links

- [Amazon RDS MySQL Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_MySQL.html)
- [RDS Free Tier](https://aws.amazon.com/rds/free/)
- [AWS Secrets Manager](https://docs.aws.amazon.com/secretsmanager/latest/userguide/intro.html)
- [Terraform aws_db_instance](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/db_instance)
- [Spring Petclinic MySQL schema](https://github.com/spring-petclinic/spring-petclinic-microservices/tree/main/spring-petclinic-customers-service/src/main/resources/db/mysql)
