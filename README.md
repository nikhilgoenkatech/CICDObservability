# CI/CD Observability with Dynatrace
This repository provides reference implementations that demonstrate how to achieve CI/CD observability with Dynatrace across four major CI/CD platforms:

1. GitHub Actions
2. GitLab CI
3. Azure DevOps Pipelines
4. Jenkins

The repository contains both primary approaches for enabling CI/CD observability:


**Workflow Observability**  

Capturing pipeline stages, job execution, gate evaluations, approvals, promotions, and SDLC flow.
Mapping delivery processes into traceable, structured workflow events.



**OpenTelemetry‑based Instrumentation**  

Emitting traces, logs, and metrics directly from the CI/CD system using OpenTelemetry components.
Exporting telemetry to Dynatrace through OTLP.



Together, these approaches provide complete visibility into software delivery processes—spanning build, test, deploy, and release workflows—while integrating seamlessly with Dynatrace for analysis, dashboards, and automation.  

> For CICD Observability using SDLC events, refer to the [help](https://docs.dynatrace.com/docs/shortlink/pipeline-observability-ingest-sdlc-events) documentation 