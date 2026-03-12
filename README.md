# kestros-test

[![GitHub release](https://img.shields.io/github/v/release/kestros/kestros-test)](https://github.com/kestros/kestros-test/releases)

## Overview

kestros-test is a synthetic test pipeline repository used to validate the Kestros multi-agent workflow end-to-end. It exercises the full agent lifecycle, from BA scoping and Lead Developer guidelines through Developer implementation, QA review, and final Danny approval. The stories tracked on the `test` board are synthetic and do not represent real product work — they exist solely to verify that each stage of the pipeline functions correctly and that agents hand off work as expected.

## Installation

### Prerequisites

- **Java 11** or higher
- **Maven 3.6** or higher
- Git

### Steps

1. **Clone the repository**

   ```bash
   git clone https://github.com/kestros/kestros-test.git
   cd kestros-test
   ```

2. **Build the project**

   ```bash
   mvn clean install
   ```

   This compiles the source, runs all tests, and packages the bundle.

3. **Deploy (optional)**

   If you are deploying to a local Apache Sling instance, install the generated bundle via the Felix Web Console:

   ```bash
   curl -u admin:<password> \
     -F "action=install" \
     -F "bundlestart=true" \
     -F "bundlefile=@target/kestros-test-0.1.0-SNAPSHOT.jar" \
     "http://localhost:8080/system/console/bundles"
   ```

   Replace `<password>` and the port with the values for your local instance.
