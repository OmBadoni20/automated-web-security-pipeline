# Automated Web Application Security Testing Pipeline

## Overview

This project automates web application security testing using Jenkins,
Docker, and OWASP ZAP.

The pipeline scans a deliberately vulnerable OWASP Juice Shop training
application, generates security reports, and applies a severity-based
security gate to determine the Jenkins build result.

## Tools Used

- Jenkins — Pipeline orchestration
- Docker — Runs the application and scanner
- OWASP ZAP — Web application security scanning
- OWASP Juice Shop — Vulnerable test application
- Git and GitHub — Version control

## Pipeline Stages

1. Check Docker connectivity.
2. Run the OWASP ZAP baseline scan.
3. Archive HTML and JSON security reports.
4. Evaluate the security gate using the selected risk threshold.
5. Complete the pipeline and report the build result.

## Security Gate

- Medium threshold: blocks Medium- and High-risk findings.
- High threshold: blocks High-risk findings.

The pipeline accepts ZAP exit codes 0, 1, and 2 so it can process
the generated report and apply its own security gate.

## Test Target

The local Juice Shop application is accessed by the scanner at:

http://host.docker.internal:3000

Only scan applications you own or are authorized to test.

## Reports

Jenkins archives the following files after a scan:

- zap-report.html
- zap-report.json

## Running the Pipeline

The Jenkins pipeline is defined in the root-level Jenkinsfile.

The Jenkins environment requires Docker access and the Pipeline
Utility Steps plugin for JSON parsing.

## Limitations

The ZAP baseline scan primarily performs passive security testing.
It does not replace a full penetration test or comprehensive
dynamic security testing.
