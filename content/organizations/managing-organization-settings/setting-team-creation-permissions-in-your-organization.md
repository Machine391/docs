diff --git a/.github/workflows/aws.yml b/.github/workflows/aws.yml
new file mode 100644
index 00000000..e769d364
--- /dev/null
+++ b/.github/workflows/aws.yml
@@ -0,0 +1,94 @@
+# This workflow will build and push a new container image to Amazon ECR,
+# and then will deploy a new task definition to Amazon ECS, when there is a push to the "master" branch.
+#
+# To use this workflow, you will need to complete the following set-up steps:
+#
+# 1. Create an ECR repository to store your images.
+#    For example: `aws ecr create-repository --repository-name my-ecr-repo --region us-east-2`.
+#    Replace the value of the `ECR_REPOSITORY` environment variable in the workflow below with your repository's name.
+#    Replace the value of the `AWS_REGION` environment variable in the workflow below with your repository's region.
+#
+# 2. Create an ECS task definition, an ECS cluster, and an ECS service.
+#    For example, follow the Getting Started guide on the ECS console:
+#      https://us-east-2.console.aws.amazon.com/ecs/home?region=us-east-2#/firstRun
+#    Replace the value of the `ECS_SERVICE` environment variable in the workflow below with the name you set for the Amazon ECS service.
+#    Replace the value of the `ECS_CLUSTER` environment variable in the workflow below with the name you set for the cluster.
+#
+# 3. Store your ECS task definition as a JSON file in your repository.
+#    The format should follow the output of `aws ecs register-task-definition --generate-cli-skeleton`.
+#    Replace the value of the `ECS_TASK_DEFINITION` environment variable in the workflow below with the path to the JSON file.
+#    Replace the value of the `CONTAINER_NAME` environment variable in the workflow below with the name of the container
+#    in the `containerDefinitions` section of the task definition.
+#
+# 4. Store an IAM user access key in GitHub Actions secrets named `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`.
+#    See the documentation for each action used below for the recommended IAM policies for this IAM user,
+#    and best practices on handling the access key credentials.
+
+name: Deploy to Amazon ECS
+
+on:
+  push:
+    branches: [ "master" ]
+
+env:
+  AWS_REGION: MY_AWS_REGION                   # set this to your preferred AWS region, e.g. us-west-1
+  ECR_REPOSITORY: MY_ECR_REPOSITORY           # set this to your Amazon ECR repository name
+  ECS_SERVICE: MY_ECS_SERVICE                 # set this to your Amazon ECS service name
+  ECS_CLUSTER: MY_ECS_CLUSTER                 # set this to your Amazon ECS cluster name
+  ECS_TASK_DEFINITION: MY_ECS_TASK_DEFINITION # set this to the path to your Amazon ECS task definition
+                                               # file, e.g. .aws/task-definition.json
+  CONTAINER_NAME: MY_CONTAINER_NAME           # set this to the name of the container in the
+                                               # containerDefinitions section of your task definition
+
+permissions:
+  contents: read
+
+jobs:
+  deploy:
+    name: Deploy
+    runs-on: ubuntu-latest
+    environment: production
+
+    steps:
+    - name: Checkout
+      uses: actions/checkout@v3
+
+    - name: Configure AWS credentials
+      uses: aws-actions/configure-aws-credentials@v1
+      with:
+        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
+        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
+        aws-region: ${{ env.AWS_REGION }}
+
+    - name: Login to Amazon ECR
+      id: login-ecr
+      uses: aws-actions/amazon-ecr-login@v1
+
+    - name: Build, tag, and push image to Amazon ECR
+      id: build-image
+      env:
+        ECR_REGISTRY: ${{ steps.login-ecr.outputs.registry }}
+        IMAGE_TAG: ${{ github.sha }}
+      run: |
+        # Build a docker container and
+        # push it to ECR so that it can
+        # be deployed to ECS.
+        docker build -t $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG .
+        docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
+        echo "image=$ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG" >> $GITHUB_OUTPUT
+
+    - name: Fill in the new image ID in the Amazon ECS task definition
+      id: task-def
+      uses: aws-actions/amazon-ecs-render-task-definition@v1
+      with:
+        task-definition: ${{ env.ECS_TASK_DEFINITION }}
+        container-name: ${{ env.CONTAINER_NAME }}
+        image: ${{ steps.build-image.outputs.image }}
+
+    - name: Deploy Amazon ECS task definition
+      uses: aws-actions/amazon-ecs-deploy-task-definition@v1
+      with:
+        task-definition: ${{ steps.task-def.outputs.task-definition }}
+        service: ${{ env.ECS_SERVICE }}
+        cluster: ${{ env.ECS_CLUSTER }}
+        wait-for-service-stability: true
diff --git a/.github/workflows/codeql.yml b/.github/workflows/codeql.yml
new file mode 100644
index 00000000..c80d9916
--- /dev/null
+++ b/.github/workflows/codeql.yml
@@ -0,0 +1,89 @@
+# For most projects, this workflow file will not need changing; you simply need
+# to commit it to your repository.
+#
+# You may wish to alter this file to override the set of languages analyzed,
+# or to provide custom queries or build logic.
+#
+# ******** NOTE ********
+# We have attempted to detect the languages in your repository. Please check
+# the `language` matrix defined below to confirm you have the correct set of
+# supported CodeQL languages.
+#
+name: "CodeQL"
+
+on:
+  push:
+    branches: [ "master" ]
+  pull_request:
+    branches: [ "master" ]
+  schedule:
+    - cron: '15 16 * * 3'
+
+jobs:
+  analyze:
+    name: Analyze (${{ matrix.language }})
+    # Runner size impacts CodeQL analysis time. To learn more, please see:
+    #   - https://gh.io/recommended-hardware-resources-for-running-codeql
+    #   - https://gh.io/supported-runners-and-hardware-resources
+    #   - https://gh.io/using-larger-runners (GitHub.com only)
+    # Consider using larger runners or machines with greater resources for possible analysis time improvements.
+    runs-on: ${{ (matrix.language == 'swift' && 'macos-latest') || 'ubuntu-latest' }}
+    timeout-minutes: ${{ (matrix.language == 'swift' && 120) || 360 }}
+    permissions:
+      # required for all workflows
+      security-events: write
+
+      # only required for workflows in private repositories
+      actions: read
+      contents: read
+
+    strategy:
+      fail-fast: false
+      matrix:
+        include:
+        - language: ruby
+          build-mode: none
+        # CodeQL supports the following values keywords for 'language': 'c-cpp', 'csharp', 'go', 'java-kotlin', 'javascript-typescript', 'python', 'ruby', 'swift'
+        # Use `c-cpp` to analyze code written in C, C++ or both
+        # Use 'java-kotlin' to analyze code written in Java, Kotlin or both
+        # Use 'javascript-typescript' to analyze code written in JavaScript, TypeScript or both
+        # To learn more about changing the languages that are analyzed or customizing the build mode for your analysis,
+        # see https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/customizing-your-advanced-setup-for-code-scanning.
+        # If you are analyzing a compiled language, you can modify the 'build-mode' for that language to customize how
+        # your codebase is analyzed, see https://docs.github.com/en/code-security/code-scanning/creating-an-advanced-setup-for-code-scanning/codeql-code-scanning-for-compiled-languages
+    steps:
+    - name: Checkout repository
+      uses: actions/checkout@v4
+
+    # Initializes the CodeQL tools for scanning.
+    - name: Initialize CodeQL
+      uses: github/codeql-action/init@v3
+      with:
+        languages: ${{ matrix.language }}
+        build-mode: ${{ matrix.build-mode }}
+        # If you wish to specify custom queries, you can do so here or in a config file.
+        # By default, queries listed here will override any specified in a config file.
+        # Prefix the list here with "+" to use these queries and those in the config file.
+
+        # For more details on CodeQL's query packs, refer to: https://docs.github.com/en/code-security/code-scanning/automatically-scanning-your-code-for-vulnerabilities-and-errors/configuring-code-scanning#using-queries-in-ql-packs
+        # queries: security-extended,security-and-quality
+
+    # If the analyze step fails for one of the languages you are analyzing with
+    # "We were unable to automatically build your code", modify the matrix above
+    # to set the build mode to "manual" for that language. Then modify this step
+    # to build your code.
+    # ℹ️ Command-line programs to run using the OS shell.
+    # 📚 See https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions#jobsjob_idstepsrun
+    - if: matrix.build-mode == 'manual'
+      run: |
+        echo 'If you are using a "manual" build mode for one or more of the' \
+          'languages you are analyzing, replace this with the commands to build' \
+          'your code, for example:'
+        echo '  make bootstrap'
+        echo '  make release'
+        exit 1
+
+    - name: Perform CodeQL Analysis
+      uses: github/codeql-action/analyze@v3
+      with:
+        category: "/language:${{matrix.language}}"
diff --git a/.github/workflows/generator-generic-ossf-slsa3-publish.yml b/.github/workflows/generator-generic-ossf-slsa3-publish.yml
new file mode 100644
index 00000000..a36e782c
--- /dev/null
+++ b/.github/workflows/generator-generic-ossf-slsa3-publish.yml
@@ -0,0 +1,66 @@
+# This workflow uses actions that are not certified by GitHub.
+# They are provided by a third-party and are governed by
+# separate terms of service, privacy policy, and support
+# documentation.
+
+# This workflow lets you generate SLSA provenance file for your project.
+# The generation satisfies level 3 for the provenance requirements - see https://slsa.dev/spec/v0.1/requirements
+# The project is an initiative of the OpenSSF (openssf.org) and is developed at
+# https://github.com/slsa-framework/slsa-github-generator.
+# The provenance file can be verified using https://github.com/slsa-framework/slsa-verifier.
+# For more information about SLSA and how it improves the supply-chain, visit slsa.dev.
+
+name: SLSA generic generator
+on:
+  workflow_dispatch:
+  release:
+    types: [created]
+
+jobs:
+  build:
+    runs-on: ubuntu-latest
+    outputs:
+      digests: ${{ steps.hash.outputs.digests }}
+
+    steps:
+      - uses: actions/checkout@v3
+
+      # ========================================================
+      #
+      # Step 1: Build your artifacts.
+      #
+      # ========================================================
+      - name: Build artifacts
+        run: |
+            # These are some amazing artifacts.
+            echo "artifact1" > artifact1
+            echo "artifact2" > artifact2
+
+      # ========================================================
+      #
+      # Step 2: Add a step to generate the provenance subjects
+      #         as shown below. Update the sha256 sum arguments
+      #         to include all binaries that you generate
+      #         provenance for.
+      #
+      # ========================================================
+      - name: Generate subject for provenance
+        id: hash
+        run: |
+          set -euo pipefail
+
+          # List the artifacts the provenance will refer to.
+          files=$(ls artifact*)
+          # Generate the subjects (base64 encoded).
+          echo "hashes=$(sha256sum $files | base64 -w0)" >> "${GITHUB_OUTPUT}"
+
+  provenance:
+    needs: [build]
+    permissions:
+      actions: read   # To read the workflow path.
+      id-token: write # To sign the provenance.
+      contents: write # To add assets to a release.
+    uses: slsa-framework/slsa-github-generator/.github/workflows/generator_generic_slsa3.yml@v1.4.0
+    with:
+      base64-subjects: "${{ needs.build.outputs.digests }}"
+      upload-assets: true # Optional: Upload to a new release
diff --git a/.github/workflows/manual.yml b/.github/workflows/manual.yml
new file mode 100644
index 00000000..11b2e355
--- /dev/null
+++ b/.github/workflows/manual.yml
@@ -0,0 +1,32 @@
+# This is a basic workflow that is manually triggered
+
+name: Manual workflow
+
+# Controls when the action will run. Workflow runs when manually triggered using the UI
+# or API.
+on:
+  workflow_dispatch:
+    # Inputs the workflow accepts.
+    inputs:
+      name:
+        # Friendly description to be shown in the UI instead of 'name'
+        description: 'Person to greet'
+        # Default value if no value is explicitly provided
+        default: 'World'
+        # Input has to be provided for the workflow to run
+        required: true
+        # The data type of the input
+        type: string
+
+# A workflow run is made up of one or more jobs that can run sequentially or in parallel
+jobs:
+  # This workflow contains a single job called "greet"
+  greet:
+    # The type of runner that the job will run on
+    runs-on: ubuntu-latest
+
+    # Steps represent a sequence of tasks that will be executed as part of the job
+    steps:
+    # Runs a single command using the runners shell
+    - name: Send greeting
+      run: echo "Hello ${{ inputs.name }}"
diff --git a/.github/workflows/veracode.yml b/.github/workflows/veracode.yml
new file mode 100644
index 00000000..7fcb8968
--- /dev/null
+++ b/.github/workflows/veracode.yml
@@ -0,0 +1,59 @@
+# This workflow uses actions that are not certified by GitHub.
+# They are provided by a third-party and are governed by
+# separate terms of service, privacy policy, and support
+# documentation.
+
+# This workflow will initiate a Veracode Static Analysis Pipeline scan, return a results.json and convert to SARIF for upload as a code scanning alert
+
+name: Veracode Static Analysis Pipeline Scan
+
+on:
+  push:
+    branches: [ "master" ]
+  pull_request:
+    # The branches below must be a subset of the branches above
+    branches: [ "master" ]
+  schedule:
+    - cron: '38 12 * * 6'
+
+# A workflow run is made up of one or more jobs that can run sequentially or in parallel
+permissions:
+  contents: read
+
+jobs:
+  # This workflow contains a job to build and submit pipeline scan, you will need to customize the build process accordingly and make sure the artifact you build is used as the file input to the pipeline scan file parameter
+  build-and-pipeline-scan:
+    # The type of runner that the job will run on
+    permissions:
+      contents: read # for actions/checkout to fetch code
+      security-events: write # for github/codeql-action/upload-sarif to upload SARIF results
+      actions: read # only required for a private repository by github/codeql-action/upload-sarif to get the Action run status
+    runs-on: ubuntu-latest
+    steps:
+
+    # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it and copies all sources into ZIP file for submitting for analysis. Replace this section with your applications build steps
+    - uses: actions/checkout@v3
+      with:
+        repository: ''
+
+    - run: zip -r veracode-scan-target.zip ./
+
+    # download the Veracode Static Analysis Pipeline scan jar
+    - run: curl --silent --show-error --fail -O https://downloads.veracode.com/securityscan/pipeline-scan-LATEST.zip
+    - run: unzip -o pipeline-scan-LATEST.zip
+
+    - uses: actions/setup-java@v3
+      with:
+        java-version: 8
+        distribution: 'temurin'
+    - run: java -jar pipeline-scan.jar --veracode_api_id "${{secrets.VERACODE_API_ID}}" --veracode_api_key "${{secrets.VERACODE_API_KEY}}" --fail_on_severity="Very High, High" --file veracode-scan-target.zip
+      continue-on-error: true
+    - name: Convert pipeline scan output to SARIF format
+      id: convert
+      uses: veracode/veracode-pipeline-scan-results-to-sarif@ff08ae5b45d5384cb4679932f184c013d34da9be
+      with:
+        pipeline-results-json: results.json
+    - uses: github/codeql-action/upload-sarif@v2
+      with:
+        # Path to SARIF file relative to the root of the repository
+        sarif_file: veracode-results.sarif---
title: Setting team creation permissions in your organization
intro: You can allow all organization members to create teams or limit team creation to organization owners.
redirect_from:
  - /articles/setting-team-creation-permissions-in-your-organization
  - /github/setting-up-and-managing-organizations-and-teams/setting-team-creation-permissions-in-your-organization
versions:
  fpt: '*'
  ghes: '*'
  ghec: '*'
topics:
  - Organizations
  - Teams
shortTitle: Restrict team creation
---

Organization owners can set team creation permissions.

If you do not set team creation permissions, all organization members will be able to create teams by default.

{% data reusables.profile.access_org %}
{% data reusables.profile.org_settings %}
{% data reusables.organizations.member-privileges %}
1. Under "Team creation rules", select or deselect **Allow members to create teams**.
1. Click **Save**.
