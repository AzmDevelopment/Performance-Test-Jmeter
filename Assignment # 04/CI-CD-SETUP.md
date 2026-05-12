# CI/CD Pipeline Setup for JMeter Tests

This document explains how to set up and use the CI/CD pipelines for automated JMeter performance testing of the Marketplace API.

## Overview

Three CI/CD pipeline configurations are provided:
1. **GitHub Actions** - [`.github/workflows/jmeter-test.yml`](.github/workflows/jmeter-test.yml:1)
2. **GitLab CI** - [`.gitlab-ci.yml`](.gitlab-ci.yml:1)
3. **Jenkins** - [`Jenkinsfile`](Jenkinsfile:1)

All pipelines perform the same core tasks:
- Install Apache JMeter on Ubuntu runner
- Execute [`MarketPlaceTest.jmx`](MarketPlaceTest.jmx:1)
- Generate test results and HTML reports
- Archive artifacts (results.jtl and HTML report)
- Validate test success/failure

---

## GitHub Actions

### Setup

1. **File Location**: [`.github/workflows/jmeter-test.yml`](.github/workflows/jmeter-test.yml:1)

2. **Automatic Triggers**:
   - Push to `main` or `develop` branches
   - Pull requests to `main` branch
   - Daily schedule at 2 AM UTC
   - Manual trigger via workflow_dispatch

3. **No Additional Setup Required** - The workflow is ready to use once committed to your repository.

### Features

✅ **Automated Installation**
- Java 17 (Temurin distribution)
- Apache JMeter 5.6.3

✅ **Test Execution**
- Runs JMeter in non-GUI mode
- Generates results.jtl file
- Creates HTML dashboard report

✅ **Artifacts**
- `jmeter-results` - Contains results.jtl (30-day retention)
- `jmeter-html-report` - Contains HTML dashboard (30-day retention)

✅ **PR Comments**
- Automatically posts test summary to pull requests
- Shows success rate and failure count

✅ **Failure Detection**
- Pipeline fails if any test assertions fail
- Displays failure count in logs

### Usage

#### View Results
1. Go to **Actions** tab in your GitHub repository
2. Click on the workflow run
3. Scroll to **Artifacts** section
4. Download `jmeter-results` or `jmeter-html-report`

#### Manual Trigger
1. Go to **Actions** tab
2. Select "JMeter Performance Test" workflow
3. Click **Run workflow**
4. Select branch and click **Run workflow**

### Example Output

```
✅ All tests passed successfully
Total samples: 2
Failed samples: 0
```

---

## GitLab CI

### Setup

1. **File Location**: [`.gitlab-ci.yml`](.gitlab-ci.yml:1)

2. **Automatic Triggers**:
   - Push to `main` or `develop` branches
   - Merge requests

3. **Requirements**:
   - GitLab Runner with Docker executor
   - Runner tagged with `docker`

### Features

✅ **Two-Stage Pipeline**
- **test** - Runs JMeter tests
- **report** - Publishes HTML report to GitLab Pages (main branch only)

✅ **Automated Installation**
- Java 17 (OpenJDK)
- Apache JMeter 5.6.3

✅ **Test Execution**
- Runs on Ubuntu 22.04 Docker image
- Generates results.jtl and HTML report

✅ **Artifacts**
- Results stored for 30 days
- HTML report available in job artifacts
- GitLab Pages deployment (main branch)

✅ **JUnit Report Integration**
- Results.jtl exposed as JUnit report
- Visible in merge request widget

### Usage

#### View Results in GitLab

1. **Job Artifacts**:
   - Go to **CI/CD > Pipelines**
   - Click on pipeline
   - Click **Browse** next to artifacts
   - Download or view files

2. **GitLab Pages** (main branch only):
   - Access at: `https://<username>.gitlab.io/<project-name>/`
   - HTML report automatically published

#### Customize Runner Tags

If your runner uses different tags, modify:
```yaml
tags:
  - your-runner-tag
```

### Example Output

```
=== JMeter Test Results Summary ===
Total samples: 2
Successful: 2
Failed: 0
✅ All tests passed successfully
```

---

## Jenkins

### Setup

1. **File Location**: [`Jenkinsfile`](Jenkinsfile:1)

2. **Requirements**:
   - Jenkins with Pipeline plugin
   - Agent with `wget`, `tar`, and Java installed
   - HTML Publisher plugin (for HTML reports)

3. **Create Pipeline Job**:
   ```
   1. New Item → Pipeline
   2. Configure → Pipeline section
   3. Definition: Pipeline script from SCM
   4. SCM: Git
   5. Repository URL: <your-repo-url>
   6. Script Path: Jenkinsfile
   7. Save
   ```

### Features

✅ **Parameterized Build**
- `JMETER_VERSION` - JMeter version (default: 5.6.3)
- `THREAD_COUNT` - Number of threads/users (default: 1)
- `LOOP_COUNT` - Number of iterations (default: 1)
- `BASE_URL_1` - Marketplace API host (default: marketplace-qa.azm-dev.com)

✅ **Multi-Stage Pipeline**
1. **Setup** - Clean workspace and create directories
2. **Install JMeter** - Download and extract JMeter
3. **Run JMeter Tests** - Execute test plan
4. **Analyze Results** - Parse and display summary
5. **Check Thresholds** - Validate success criteria

✅ **Artifacts**
- All test results archived
- HTML report published via HTML Publisher plugin

✅ **Post-Build Actions**
- Success/failure notifications (configurable)
- Workspace cleanup

### Usage

#### Run Pipeline

1. **Standard Run**:
   - Click **Build Now**
   - Uses default parameters

2. **Parameterized Run**:
   - Click **Build with Parameters**
   - Customize values:
     - Thread Count: 10
     - Loop Count: 5
     - Base URL: marketplace-qa.azm-dev.com
   - Click **Build**

#### View Results

1. **Console Output**:
   - Click on build number
   - Click **Console Output**

2. **HTML Report**:
   - Click on build number
   - Click **JMeter Performance Report** in sidebar

3. **Artifacts**:
   - Click on build number
   - Click **Build Artifacts**
   - Download files from `test-results/`

### Example Parameters

```groovy
JMETER_VERSION: 5.6.3
THREAD_COUNT: 10
LOOP_COUNT: 100
BASE_URL_1: marketplace-qa.azm-dev.com
```

### Notifications

Uncomment email sections in [`Jenkinsfile`](Jenkinsfile:1) to enable:

```groovy
emailext (
    subject: "JMeter Test Passed: ${env.JOB_NAME}",
    body: "Test completed successfully.",
    to: "team@example.com"
)
```

---

## Common Configuration

### JMeter Version

All pipelines use **Apache JMeter 5.6.3** by default. To change:

**GitHub Actions:**
```yaml
- name: Install JMeter
  run: |
    wget https://archive.apache.org/dist/jmeter/binaries/apache-jmeter-5.5.0.tgz
```

**GitLab CI:**
```yaml
variables:
  JMETER_VERSION: "5.5.0"
```

**Jenkins:**
```groovy
parameters {
    string(name: 'JMETER_VERSION', defaultValue: '5.5.0', ...)
}
```

### Test Plan Location

All pipelines expect [`MarketPlaceTest.jmx`](MarketPlaceTest.jmx:1) in the Assignment # 04 folder. To change:

**GitHub Actions:**
```yaml
- name: Run JMeter test
  run: |
    jmeter -n -t path/to/your-test.jmx ...
```

**GitLab CI:**
```yaml
script:
  - jmeter -n -t path/to/your-test.jmx ...
```

**Jenkins:**
```groovy
sh """
    jmeter -n -t path/to/your-test.jmx ...
"""
```

---

## Artifacts Generated

### 1. results.jtl

**Format**: XML/CSV (JMeter Test Log)

**Contents**:
- Request/response details
- Response times
- Success/failure status
- Timestamps
- Thread information

**Sample**:
```xml
<httpSample t="523" lt="456" ts="1744617711000" s="true" lb="Login" rc="200" rm="OK">
  <responseHeader>...</responseHeader>
</httpSample>
```

### 2. HTML Report

**Location**: `test-results/html-report/`

**Contents**:
- Dashboard with charts
- Statistics table
- Response time graphs
- Throughput analysis
- Error analysis

**Key Files**:
- `index.html` - Main dashboard
- `content/` - Charts and data
- `sbadmin2-1.0.7/` - CSS/JS assets

---

## Interpreting Results

### Success Criteria

✅ **Test Passes When**:
- All HTTP requests return status code 200
- No assertion failures
- results.jtl contains successful samples

❌ **Test Fails When**:
- Any HTTP request fails (non-200 status)
- Response assertions fail
- Connection errors occur

### Key Metrics

| Metric | Description | Good Value |
|--------|-------------|------------|
| **Response Time** | Time to receive response | < 1000ms |
| **Throughput** | Requests per second | Depends on load |
| **Error Rate** | % of failed requests | 0% |
| **Success Rate** | % of successful requests | 100% |

### Reading results.jtl

```bash
# Count total samples
grep -c '<httpSample' results.jtl

# Count failures
grep 'success="false"' results.jtl | wc -l

# Extract response times
grep -oP 't="\K[0-9]+' results.jtl
```

---

## Troubleshooting

### Common Issues

#### 1. JMeter Download Fails

**Error**: `wget: unable to resolve host address`

**Solution**:
- Check internet connectivity on runner
- Use mirror URL:
  ```bash
  wget https://dlcdn.apache.org/jmeter/binaries/apache-jmeter-5.6.3.tgz
  ```

#### 2. Java Not Found

**Error**: `java: command not found`

**Solution**:
- Ensure Java is installed on runner
- GitHub Actions: Uses `setup-java` action
- GitLab CI: Installs `openjdk-17-jdk`
- Jenkins: Requires Java on agent

#### 3. Test Failures

**Error**: `❌ Test failed with X failures`

**Solution**:
1. Check API availability against your Marketplace QA endpoint (example: `curl https://marketplace-qa.azm-dev.com/health`)
2. Review results.jtl for error messages
3. Check assertion configuration in JMX file
4. Verify network connectivity from runner

#### 4. No Artifacts Generated

**Error**: Artifacts section is empty

**Solution**:
- Ensure `test-results/` directory is created
- Check JMeter command completed successfully
- Verify artifact paths in pipeline configuration

#### 5. HTML Report Not Generated

**Error**: `test-results/html-report/` is empty

**Solution**:
- Ensure `-e -o` flags are used in JMeter command
- Check JMeter log for errors
- Verify results.jtl is not empty

---

## Advanced Configuration

### Parallel Execution

**GitHub Actions** - Matrix strategy:
```yaml
strategy:
  matrix:
    thread_count: [1, 10, 50]
    loop_count: [1, 10, 100]
```

**GitLab CI** - Parallel jobs:
```yaml
jmeter_test:
  parallel:
    matrix:
      - THREADS: [1, 10, 50]
        LOOPS: [1, 10, 100]
```

### Custom JMeter Properties

Add to JMeter command:
```bash
jmeter -n -t test.jmx \
  -JbaseUrl=marketplace-qa.azm-dev.com \
  -Jthreads=10 \
  -Jduration=60
```

Then use in JMX:
```xml
<stringProp name="Argument.value">${__P(baseUrl,marketplace-qa.azm-dev.com)}</stringProp>
```

### Performance Thresholds

Add threshold checking:

```bash
# Check average response time
AVG_TIME=$(awk -F',' '{sum+=$2; count++} END {print sum/count}' results.jtl)
if [ "$AVG_TIME" -gt 1000 ]; then
  echo "Average response time exceeded threshold"
  exit 1
fi
```

### Notifications

**GitHub Actions** - Slack:
```yaml
- name: Slack Notification
  uses: 8398a7/action-slack@v3
  with:
    status: ${{ job.status }}
    text: 'JMeter test completed'
```

**GitLab CI** - Email:
```yaml
after_script:
  - echo "Test results" | mail -s "JMeter Results" team@example.com
```

**Jenkins** - Email (already configured in Jenkinsfile):
```groovy
emailext (
    subject: "Test Results",
    body: "Check attached report",
    to: "team@example.com"
)
```

---

## Best Practices

### 1. Version Control
- ✅ Commit JMX files to repository
- ✅ Use semantic versioning for test plans
- ✅ Document changes in commit messages

### 2. Artifact Retention
- ✅ Keep results for 30 days minimum
- ✅ Archive critical test runs permanently
- ✅ Clean up old artifacts regularly

### 3. Test Isolation
- ✅ Use unique test data per run
- ✅ Clean up test data after execution
- ✅ Avoid dependencies between tests

### 4. Monitoring
- ✅ Set up alerts for test failures
- ✅ Track performance trends over time
- ✅ Review reports regularly

### 5. Security
- ✅ Don't commit sensitive data in JMX files
- ✅ Use environment variables for credentials
- ✅ Restrict access to test results

---

## Performance Testing Scenarios

### Smoke Test (Current Setup)
```yaml
Threads: 1
Iterations: 1
Duration: ~2 seconds
Purpose: Validate functionality
```

### Load Test
```yaml
Threads: 50
Iterations: 100
Ramp-up: 30 seconds
Purpose: Test normal load
```

### Stress Test
```yaml
Threads: 200
Iterations: 500
Ramp-up: 60 seconds
Purpose: Find breaking point
```

### Spike Test
```yaml
Threads: 0 → 100 → 0
Duration: 5 minutes
Purpose: Test sudden load
```

---

## Resources

### Documentation
- [Apache JMeter](https://jmeter.apache.org/)
- [GitHub Actions](https://docs.github.com/en/actions)
- [GitLab CI/CD](https://docs.gitlab.com/ee/ci/)
- [Jenkins Pipeline](https://www.jenkins.io/doc/book/pipeline/)

### Test Plan Files
- [`MarketPlaceTest.jmx`](MarketPlaceTest.jmx:1) - JMeter test plan
- [`README.md`](../README.md:1) - Test documentation
- [`CI-CD-SETUP.md`](CI-CD-SETUP.md:1) - CI/CD setup and architecture notes

### Pipeline Files
- [`.github/workflows/jmeter-test.yml`](.github/workflows/jmeter-test.yml:1) - GitHub Actions
- [`.gitlab-ci.yml`](.gitlab-ci.yml:1) - GitLab CI
- [`Jenkinsfile`](Jenkinsfile:1) - Jenkins Pipeline

---

## Support

For issues or questions:
1. Check pipeline logs for error messages
2. Review JMeter log file (`jmeter.log`)
3. Verify test plan in JMeter GUI
4. Consult official documentation

**Last Updated**: April 2026
**Version**: 1.0
