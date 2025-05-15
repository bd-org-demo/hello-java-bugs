# Black Duck Software Integration Lab 1

The goal of this lab is to provide hands on experience integrating a **Polaris** scan into a **GitHub** workflow using the [Black Duck Security Scan Action](https://github.com/marketplace/actions/black-duck-security-scan) and demonstrating its post scan capabilities. As part of the lab, we will:
1. setup a GitHub workflow and execute a full scan
2. break the build based on a [policy](https://polaris.blackduck.com/developer/default/polaris-documentation/t_post_scan_policies) defined in Polaris
3. review the findings in Polaris
4. review the code scanning findings in the GitHub Advanced Security tab
5. introduce a vulnerable code change, triggering a PR scan which adds a comment to the Pull Request

This repository contains everything you need to complete the lab except for the prerequisites listed below.

# Prerequisites

1. [signup](https://github.com/signup) for a free GitHub Account
2. [create](https://polaris.blackduck.com/developer/default/polaris-documentation/t_make-token) a Polaris Access Token

# Clone repository

1. Clone this repository into your GitHub account via _GitHub → New → Import a Repository_
   - enter https://github.com/blackduck-se/bd-integrations-lab1.git
   - enter repository name, e.g. hello-java
   - leave as public (required for GHAS on free accounts)

# Setup workflow

1. Confirm all GitHub Actions are allowed via _GitHub → Project → Settings → Actions → General → Actions Permissions_
2. Confirm GITHUB_TOKEN has workflow read & write permissions via _GitHub → Project → Settings → Actions → General → Workflow Permissions_
3. Add the following variables, adding POLARIS_ACCESSTOKEN as a **secret** via _GitHub → Project → Settings → Secrets and Variables → Actions_
   - POLARIS_SERVERURL
   - POLARIS_ACCESSTOKEN
4. Add a coverity.yaml to the project repository via _GitHub → Project → Add file → Create new file_

```
capture:
  build:
    clean-command: mvn -B clean
    build-command: mvn -B -DskipTests package
analyze:
  checkers:
    webapp-security:
      enabled: true
```

5. Log into Polaris and [create an application](https://polaris.blackduck.com/developer/default/polaris-documentation/t_gs-app-superuser) and assign SAST and SCA subscriptions.
6. Create a new workflow via _GitHub → Project → Actions → New Workflow → Setup a workflow yourself_
   - Note: application name must match what created in Polaris, e.g. chuckaude-hello-java ← **replace my name with your name**

```
# example workflow for Polaris scans using the Black Duck Security Scan Action
# https://github.com/marketplace/actions/black-duck-security-scan
name: Polaris
on:
  push:
    branches: [ main, master, develop, stage, release ]
  pull_request:
    branches: [ main, master, develop, stage, release ]
  workflow_dispatch:
jobs:
  polaris:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout Source
      uses: actions/checkout@v4
    - name: Setup Java JDK
      uses: actions/setup-java@v4
      with:
        java-version: 21
        distribution: temurin
        cache: maven
    - name: Polaris Scan
      uses: blackduck-inc/black-duck-security-scan@v2.1.1
      with:
        polaris_server_url: ${{ vars.POLARIS_SERVERURL }}
        polaris_access_token: ${{ secrets.POLARIS_ACCESSTOKEN }}
        polaris_assessment_types: 'SAST,SCA'
        polaris_application_name: chuckaude-${{ github.event.repository.name }}
        polaris_prComment_enabled: 'true'
        polaris_reports_sarif_create: 'true'
        polaris_upload_sarif_report: 'true'
        github_token: ${{ secrets.GITHUB_TOKEN }}
        # include_diagnostics: true
#    - name: Save Logs
#      if: always()
#      uses: actions/upload-artifact@v4
#      with:
#        name: bridge-logs
#        path: ${{ github.workspace }}/.bridge
#        include-hidden-files: true
```

# Full Scan

1. Monitor your workflow run and wait for the scan to complete via _GitHub → Project → Actions → Polaris → Most recent workflow run → Polaris_ **Milestone 1** :heavy_check_mark:
   - Note that the scan completes and the workflow passes. This is because the default policy is "notify on critical & high issues".
2. Log into Polaris and [create a policy](https://polaris.blackduck.com/developer/default/polaris-documentation/t_post_scan_policies) that breaks the build and assign it to your project.
3. Rerun workflow, and once it completes, select _Summary_ in upper left to see policy enforcement and a failed workflow. **Milestone 2** :heavy_check_mark:
4. Click on the link in the console output to view the findings in Polaris. **Milestone 3** :heavy_check_mark:
5. View findings in GitHub Advanced Security tab via _GitHub → Project → Security → Code scanning_ **Milestone 4** :heavy_check_mark:

# PR scan

1. Edit pom.xml via _GitHub → Project → Code → pom.xml → Edit pencil icon upper right_
   - change log4j version from 2.14.1 to 2.15.0
2. Click on _Commit Changes_, select create a **new branch** and start a PR
3. Review changes and click on _Create Pull Request_
4. Monitor workflow run via _GitHub → Project → Actions → Polaris → Most recent workflow run → Polaris_
5. Once workflow completes, navigate back to the PR and review the PR comment via _GitHub → Project → Pull requests_ **Milestone 5** :heavy_check_mark:

# Congratulations

You have now configured a Polaris workflow in GitHub and demonstrated all the functionality of the [Black Duck Security Scan Action](https://github.com/marketplace/actions/black-duck-security-scan). :clap: :trophy:
