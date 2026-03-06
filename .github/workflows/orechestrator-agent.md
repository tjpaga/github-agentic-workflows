---
on:
  workflow_dispatch:
    inputs:
      market:
        type: choice
        required: true
        options:
          - DE
          - AT
          - NGL
        description: Market to deploy
      environment:
        type: choice
        required: true
        options:
          - dev
          - e2e
          - tst
        description: ENVIRONMENT (dev, tst, e2e)
      infrastructure_branch:
        type: string
        required: false
        default: 'develop'
        description: INFRA_BRANCH
      dry_run:
        type: boolean
        required: false
        default: false
        description: DRY_RUN
      deploy_infrastructure:
        type: boolean
        required: false
        default: true
        description: DEPLOY_INFRASTRUCTURE
      graviton:
        type: boolean
        required: false
        default: false
        description: GRAVITON (build for ARM64/Graviton architecture)
      disable_console_logs:
        type: boolean
        required: false
        default: false
        description: DISABLE_CONSOLE_LOGS (for UI services)
permissions: read-all
safe-outputs:
  assign-to-agent:
  add-comment:
engine: copilot
---
 
# Multi-Service Deployment Orchestrator
 
---
 
## COMMON OPERATIONS
 
### Validate Configuration
Check all required inputs are provided:
- Market is one of: DE, AT, NGL
- Environment is one of: dev, e2e, tst
- Infrastructure branch is valid
 
If any validation fails, report errors and stop deployment.
 
### Check for Duplicate Running Workflows
 
For each service in the repository list below, check if there are already running workflows with the exact same parameters:
- Same market
- Same environment
- Same infrastructure_branch
- Same dry_run setting
- Same deploy_infrastructure setting
- Same graviton setting
 
If duplicate workflows are found:
- List all duplicate workflow runs with their run IDs
- Cancel the duplicate workflow runs
- Wait for cancellation to complete
- Log which workflows were cancelled
 
Do this check for all service repositories before proceeding with deployments.
 
### Monitor All Deployments
 
For each triggered workflow:
- Poll workflow status every 30 seconds
- Track which services are: in-progress, succeeded, or failed
- Continue monitoring all services regardless of individual failures
- Do not let one service failure stop monitoring of others
- Maintain independent status tracking for each service
 
**Detect and Handle Stuck Steps:**
- For each in-progress workflow, check the logs of the currently running step
- Track the timestamp of the last log output for each step
- If a step has been running for more than 5 minutes without producing any new logs:
  - Mark the step as stuck
  - Cancel the workflow run
  - Log the stuck step details: workflow name, step name, duration without logs
  - Record this as a timeout failure
 
### Handle Failures
 
For any failed deployments:
- Log the specific service name and failure reason
- Identify which step failed in the workflow
- Capture full error logs and stack traces
- For timeout failures, include: stuck step name, last log timestamp, duration
- Keep track of all failed services separately
- Do not trigger cascading failures
- Do not rollback successful deployments
- Keep successful deployments running and intact
 
### Get Copilot Remediation
 
For each failed service:
- Compile failure information:
  - Service name, Market, Environment, Infrastructure Branch
  - Failed workflow run URL
  - Failure type: error or timeout
  - For timeout failures: stuck step name, last log time, timeout duration
  - Error logs and failure reason
  - Steps that failed
  - Timestamp of failure
 
- Request Copilot to analyze the failure and provide:
  - Root cause analysis
  - Remediation steps
  - Recommended fixes
  - Prevention measures
 
- Collect Copilot's remediation response
 
### Generate Deployment Summary
 
Provide comprehensive summary with:
- **Cancelled Duplicates**: List any duplicate workflows that were stopped
- **Deployment Configuration**: Market, Environment, Infrastructure Branch, Dry Run status, Graviton, Deploy Infrastructure
- **Successful Services**: List all services that completed successfully with timestamps
- **Failed Services**: List all services that failed with:
  - Error reason or timeout details
  - Stuck step information (if timeout)
  - Created issue URL
  - Copilot remediation steps
  - Recommended actions
- **Timed Out Services**: List services that were stopped due to stuck steps
- **In-Progress Services**: List any services still running
- **Text Actions**:
  - For failed services: Copilot's recommended remediation steps
  - Links to created issues for tracking
---
 
## REPOSITORY-SPECIFIC OPERATIONS
 
### Backend Services Configuration
 
The following backend services should be deployed independently with these parameters:
- market
- environment
- infrastructure_branch
- dry_run
- deploy_infrastructure
- graviton
 
#### Service List - Backend
 
1. **DHS** (Document Handling Service)
   - Repository: DHP/document-handling-service-backend
   - Workflow Path: .github/workflows/github-build.yml@develop
   - Type: Java/Maven service
 
2. **DSS** (Document Store Service)
   - Repository: DHP/docstore-backend
   - Workflow Path: .github/workflows/github-build.yml@develop
   - Type: Java/Maven service
 
3. **MDS** (Manage Documents Service)
   - Repository: DHP/manage-documents-service-backend
   - Workflow Path: .github/workflows/github-build.yml@develop
   - Type: Java/Maven service
 
4. **DHJ** (Document Handling Jobs)
   - Repository: DHP/document-handling-jobs
   - Workflow Path: .github/workflows/github-build.yml@develop
   - Type: Java/Maven service
 
5. **DRJ** (Document Retrieval Jobs)
   - Repository: DHP/document-retrieval-jobs
   - Workflow Path: .github/workflows/github-build.yml@develop
   - Type: Java/Maven service
 
6. **DWS** (Document Wrapper Service)
   - Repository: DHP/document-wrapper-service-backend
   - Workflow Path: .github/workflows/github-build.yml@develop
   - Type: Java/Maven service
 
### Frontend Services Configuration
 
The following frontend services should be deployed independently with these parameters:
- market
- environment
- infrastructure_branch
- dry_run
- deploy_infrastructure
- graviton
- disable_console_logs
 
#### Service List - Frontend
 
1. **DSUI** (Document Store UI)
   - Repository: DHP/docstore-ui
   - Workflow Path: .github/workflows/github-build.yml@develop
   - Type: Angular/UI service
 
2. **MDSUI** (Manage Documents UI)
   - Repository: DHP/manage-documents-ui
   - Workflow Path: .github/workflows/github-build.yml@develop
   - Type: Angular/UI service
 
---
 
## DEPLOYMENT EXECUTION
 
### Deploy All Backend Services
For each backend service listed above:
- Trigger deployment workflow using the service's repository and workflow path
- Pass backend-specific parameters
- Monitor status independently using common monitoring operation with stuck detection
 
### Deploy All Frontend Services
For each frontend service listed above:
- Trigger deployment workflow using the service's repository and workflow path
- Pass frontend-specific parameters (including disable_console_logs)
- Monitor status independently using common monitoring operation with stuck detection
 
### Complete Deployment
Use common operations to:
- Handle failures and timeouts
- Create issues for failed/timed out deployments
- Assign issues to Copilot for remediation
- Get Copilot remediation for failed/timed out deploymentsopilot's recommendations
 
 