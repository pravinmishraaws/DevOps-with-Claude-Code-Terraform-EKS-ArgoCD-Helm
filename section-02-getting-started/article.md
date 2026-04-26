# Section 2 — Getting Started

## What This Section Covers

1. Run all 8 services locally with Docker Compose — see the full application before building AWS infrastructure
2. Set up Claude Code for the `petclinic-platform` project
3. Connect Claude Code to Jira (optional — only used in this section)
4. Build Epic 1 — Foundation: S3 state backend, DynamoDB lock table, Terraform provider, directory structure

---

## Running the Application Locally

Before building infrastructure, run the application on your machine. One command starts all 8 services plus the full observability stack.

```bash
cd spring-petclinic-microservices
docker compose up -d
```

**Startup order is enforced automatically.** Config Server and Discovery Server start first. The other 6 services wait until those two pass health checks (`depends_on: condition: service_healthy`).

### What to explore

| URL | What it is |
|-----|-----------|
| http://localhost:8080 | Spring Petclinic app (API Gateway) |
| http://localhost:8761 | Eureka — service registry |
| http://localhost:9090 | Spring Boot Admin |
| http://localhost:9411 | Zipkin — distributed tracing |
| http://localhost:9091 | Prometheus — metrics |
| http://localhost:3030 | Grafana — dashboards |

Stop when done:
```bash
docker compose down
```

---

## Epic 1 — Foundation & Remote State

What you build in this section:

| Story | What it creates |
|-------|----------------|
| PetclinicPlatform1 | Terraform directory structure |
| PetclinicPlatform2 | S3 bucket + DynamoDB table (state backend) |
| PetclinicPlatform3 | Backend config for dev environment |
| PetclinicPlatform4 | Backend config for prod environment |
| PetclinicPlatform5 | AWS provider, versions.tf, tags |

After this section: `terraform init` succeeds in both dev and prod.

---

## Jira Setup (Optional)

You can connect Claude Code to Jira via the Atlassian MCP server. This is demonstrated once in this section — from Section 3 onwards, the course uses `docs/jira-backlog.md` instead (faster, no authentication needed).

To set up Jira:
1. Create a free Atlassian account at https://atlassian.com
2. Create a Jira project with key `PETPLAT`
3. Add stories from `petclinic-platform/docs/jira-backlog.md`
4. Authenticate Claude Code via `/mcp` → Atlassian OAuth

If you skip Jira, use `docs/jira-backlog.md` directly — same content.

---

## The Workflow You'll Use in Every Section

1. **Read the stories** — understand what's required and the acceptance criteria
2. **Read the spec** — `docs/technical-spec.md` has all the values (CIDRs, instance types, ports, etc.)
3. **Write one prompt** — point Claude Code to the stories and spec, tell it what to build
4. **Review the output** — open every file, check it against the acceptance criteria
5. **Deploy** — `terraform plan` then `terraform apply`
6. **Verify** — confirm in the AWS console or via CLI

---

## External Links

- [Docker Desktop](https://www.docker.com/products/docker-desktop/)
- [Spring Petclinic Microservices Repo](https://github.com/spring-petclinic/spring-petclinic-microservices)
- [Atlassian MCP Server](https://developer.atlassian.com/cloud/jira/platform/mcp/)
- [Terraform S3 Backend](https://developer.hashicorp.com/terraform/language/settings/backends/s3)
- [AWS S3 Documentation](https://docs.aws.amazon.com/s3/index.html)
- [AWS DynamoDB Documentation](https://docs.aws.amazon.com/dynamodb/index.html)
