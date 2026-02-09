# Azure DevOps Pipeline — Pull & Push BizEvents Workflow

This README describes the Azure DevOps implementation of a Dynatrace Automation workflow that:

- Pulls completed Azure DevOps pipeline builds and timelines (stages/tasks)
- Transforms pipeline and stage data into Dynatrace BizEvents
- Pushes BizEvents to Dynatrace for pipeline observability.


---

## Overview

The workflow connects to Azure DevOps REST APIs to fetch completed builds and their timelines within a configurable lookback window (the example uses 1 hour). For each relevant build it:

1. Creates and ingests a pipeline-level BizEvent (project, pipeline id, start/end, duration, result, user, trigger)
2. Retrieves the build timeline and ingests stage/task BizEvents with duration and status

Event ingestion is done using `businessEventsClient.ingest` from the Dynatrace Automation SDK.

---

## Prerequisites

- Dynatrace tenant with Automation Engine enabled
- Access to Dynatrace Credential Vault or a secure store for Azure DevOps PATs (Personal Access Tokens)
- Azure DevOps organization and project access with a PAT that has `Build (read)` scope

References:
- [Azure DevOps REST API - Builds](https://learn.microsoft.com/en-us/rest/api/azure/devops/build/builds) 
- [Dynatrace Business Events API (environment API)](https://www.dynatrace.com/support/help/dynatrace-api/environment-api/business-events)

---

## Configuration

Edit the task inputs in the workflow to set the following values (placeholders in the sample script):

- `organization` — Azure DevOps organization name
- `project` — Azure DevOps project name
- `pat` — Personal Access Token (store in Dynatrace Credential Vault and reference by id in the workflow instead of hardcoding)
- `lookbackWindowMs` — lookback in milliseconds (script uses 60 * 60 * 1000 for 1 hour)

Security note: Never hardcode PATs in the workflow JSON or in source control. Use the Dynatrace Credential Vault and retrieve credentials in the function using `credentialVaultClient.getCredentialsDetails({ id: '<CREDENTIALS_UUID>' })`.

Example placeholder (do NOT commit values):

```js
const organization = 'orgname'; // replace with organization
const project = 'data-populate'; // replace with project
// pat should be provided via credential vault; example placeholder below
const pat = 'YOUR_PAT_HERE';
```

---

## How it works (high level)

1. The function calls Azure DevOps Builds API to list completed builds.
2. For each build within the lookback window it constructs a pipeline BizEvent and calls `businessEventsClient.ingest`.
3. It then calls the Timelines API for that build to enumerate timeline records (Jobs, Phases, Tasks).
4. For Task records that fall inside the lookback window it constructs stage/job BizEvents and ingests them.

Event shape examples are shown in the Examples section below.

---

## Deployment Steps

1. Upload/import the [workflow](./workflow%20template/pull-push-bizevents.json).
2. Modify the placeholders in the workflow as mentioned in **Configuration** section.
3. Enable and schedule the workflow (e.g., interval trigger every 60 minutes).
4. Monitor workflow executions in Dynatrace Automation and verify BizEvents in the Dynatrace UI.

---

## Example BizEvent payloads

Pipeline (build) event example:

```json
{
  "project-name": "data-populate",
  "pipeline-number": 123,
  "pipeline-started_at": 1672531200000,
  "pipeline-status": "succeeded",
  "pipeline-duration": 120000,
  "pipeline-username": "Jane Doe",
  "pipeline-triggered_by": "Push",
  "event.type": "pipeline",
  "event.provider": "azure-devops"
}
```

Stage / Task event example:

```json
{
  "jobname": "CI Pipeline",
  "stagename": "Run Tests",
  "startTime": "2024-01-01T12:00:00Z",
  "finishTime": "2024-01-01T12:05:00Z",
  "status": "succeeded",
  "stageDuration": 300000,
  "event.type": "job",
  "event.provider": "azure-devops"
}
```

---


## Troubleshooting

- 401/403 from Azure DevOps APIs: verify PAT scopes include Build (read) and the PAT is valid/non-expired. Use credential vault lookup to confirm stored secret.
- Empty builds list: confirm the organization/project values and API URL; verify builds exist and that build times fall within lookback window.

---

## Customizations & Extensions

- Increase/decrease lookback window or schedule frequency to match your needs.
- Add more metadata to events (branch, commit id, pipeline definition id, buildQueueTime) by reading other build or timeline fields.
- Correlate pipeline BizEvents with traces or metrics from instrumented CI/CD runners or agents by tagging events with trace IDs or CI runner hostnames.

---

## References

- Azure DevOps Builds API: https://learn.microsoft.com/en-us/rest/api/azure/devops/build/builds
- Azure DevOps Timeline API: https://learn.microsoft.com/en-us/rest/api/azure/devops/build/timelines
- Dynatrace Business Events API: https://www.dynatrace.com/support/help/dynatrace-api/environment-api/business-events
- Dynatrace Automation Functions & JS runtime: https://www.dynatrace.com/support/help/shortlink/automation-engine

---