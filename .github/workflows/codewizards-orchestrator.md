---
on:
  workflow_dispatch:
    inputs:
      environment:
        type: choice
        required: true
        options:
          - dev
          - staging
          - prod
        description: Deployment environment
      build_only:
        type: boolean
        required: false
        default: false
        description: Build only without deployment
      run_tests:
        type: boolean
        required: false
        default: true
        description: Run integration tests
      dry_run:
        type: boolean
        required: false
        default: false
        description: Dry run mode (no actual deployment)

permissions: read-all

safe-outputs:
  assign-to-agent:
  add-comment:

engine: copilot

---

# CodeWizards Multi-Service Deployment Orchestrator

---

## COMMON OPERATIONS

### Validate Configuration

Check all required inputs are provided:
- Environment is one of: dev, staging, prod
- Build only flag is valid
- Run tests flag is valid
- Dry run flag is valid

If any validation fails, report errors and stop deployment.

### Check for Duplicate Running Workflows

For each service in the repository list below, check if there are already running workflows with the exact same parameters:
- Same environment
- Same build_only setting
- Same run_tests setting
- Same dry_run setting

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
- If a step has been running for more than 10 minutes without producing any new logs:
  - Mark the step as stuck
  - Cancel the workflow run
  - Log the stuck step details: workflow name, step name, duration without logs

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
  - Service name, Environment, Build Only flag
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
- **Deployment Configuration**: Environment, Build Only, Run Tests, Dry Run status
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
- environment
- build_only
- run_tests
- dry_run

#### Service List - Backend

1. **Vehicle Booking System**
   - Repository: tjpaga/CodeWizards/vehicle-boot
   - Workflow Path: .github/workflows/build.yml@master
   - Type: Java/Maven/Spring Boot service
   - Description: Core vehicle booking application with REST APIs and Cucumber integration tests

### Build Operations

For the Vehicle Booking System service:
- Trigger build workflow using the service's repository and workflow path
- Pass build-specific parameters
- Monitor status independently using common monitoring operation with stuck detection
- Validate test results if run_tests is enabled

### Complete Deployment

Use common operations to:
- Handle failures and timeouts
- Create issues for failed/timed out deployments
- Assign issues to Copilot for remediation
- Get Copilot remediation for failed/timed out deployments
- Generate final deployment summary
 
---

## DEPLOYMENT EXECUTION

### Build and Test Vehicle Booking System

1. Trigger build workflow from tjpaga/CodeWizards/vehicle-boot@master
2. If run_tests is enabled, execute test suite including:
   - Unit tests
   - Integration tests (Cucumber)
   - Test result reporting
3. If build_only is false, proceed to deployment
4. Monitor build progress with stuck step detection

### Handle Deployment

If build_only is false:
- Deploy successful builds to the specified environment
- Skip deployment for any failed builds
- Provide deployment status and health checks

### Complete Execution

Use common operations to:
- Monitor workflow
- Handle any failures
- Create issues for investigation
- Provide Copilot remediation recommendations
- Generate comprehensive deployment summary
