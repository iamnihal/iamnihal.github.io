---
title: "Utilizing Github Actions to Serve a tool via Web"
date: 2023-12-31T17:17:04+05:30
draft: false
toc: false
images:
tags:
  - go, github
---
**TL;DR:** I employed GitHub Actions to execute a Go utility, essentially utilizing GitHub Actions as a server to serve my Go tool via a Web interface.

---

Just for fun, I wanted to serve one of my Go tools [gh-archive](https://github.com/iamnihal/gh-archive) through a web interface. After some thought, I decided to make use of (or exploit) GitHub Actions.


# Strategy:

1. Gather user input through an HTML form (hosted via GitHub pages).
2. Trigger a dispatch event to create a workflow in GitHub with a unique name.
3. The GitHub Workflow will perform the tasks of building, running, and saving the output file of the Go program.
4. Retrieve the list of all workflow and iterate through the JSON results to match the workflow name.
5. Check the status of that workflow, if it's completed or not.
6. If the workflow status is completed, retrieve the downloadable URL of the artifacts, download, and extract them.
7. Display the content extracted from the output file back to the user.

To execute these steps, I skimmed through the GitHub REST API Documentation to identify the necessary APIs for the tasks at hand.

# GitHub Workflow:
```yaml
name: Go
run-name: "${{ github.event.inputs.run_id }}"

on:
  workflow_dispatch:
    inputs:
      topic:
        description: 'Repository topic'
        required: true
      repo:
        description: 'Number of repositories to check'
        required: false
      run_id:
        description: 'Workflow name'
        required: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3

    - name: Fetch Go Program Repository
      run: git clone https://github.com/iamnihal/gh-archive.git gh-archive

    - name: Set up Go
      uses: actions/setup-go@v4
      with:
        go-version: '1.21.3'

    - name: Build
      run: go build -v -o main .
      working-directory: ./gh-archive

    - name: Run
      run: ./gh-archive/main -t "${{ github.event.inputs.topic }}" -n "${{ github.event.inputs.repo }}" -o ./gh-archive/output.txt

    - name: Save Output
      uses: actions/upload-artifact@v2
      with:
        name: go-output
        path: ./gh-archive/output.txt

  access-artifact:
    needs: build
    runs-on: ubuntu-latest
    steps:
    - name: Download Artifact
      uses: actions/download-artifact@v2
      with:
        name: go-output
        path: ./downloaded-artifact

    - name: Print Contents
      run: |
        cat ./downloaded-artifact/output.txt
```
The workflow is pretty much self-explanatory. The workflow is triggered by a manual dispatch event (workflow_dispatch), allowing the user to input values such as the repository topic, the number of repositories to check (repo), and the workflow name (run_id). Then it goes on to set up Go, building, running, and saving the output file of the Go program to the GitHub storage.


# index.html
[Click here](https://github.com/iamnihal/gh-archive-web/blob/main/index.html) to view the HTML source code.

Within the HTML file, there are two crucial JavaScript functions:
1. `triggerWorkflow(topic, repo)` - This function initiates a dispatch event to GitHub Actions, subsequently retrieving the list of all workflow runs for a repository. It then matches the "run_id" to identify the corresponding workflow run.

**APIs:**

```
/repos/OWNER/REPO/actions/workflows/WORKFLOW_ID/dispatches

```
```
/repos/OWNER/REPO/actions/runs
```

1. `checkWorkflowCompletion()` - This function retrieves details about a specific workflow run to assess its completion status. If the status is "completed," it proceeds to download the artifacts, unzip them, perform JSON parsing, and present the results to the user.

**APIs:**


```
/repos/OWNER/REPO/actions/runs/RUN_ID

```
```
/repos/OWNER/REPO/actions/runs/RUN_ID/artifacts
```

# demo

You can try it here [https://iamnihal.github.io/gh-archive-web/](https://iamnihal.github.io/gh-archive-web/)

{{< rawhtml >}}
<video width=100% controls autoplay>
    <source src="/posts/demo.mp4" type="video/mp4">
    Your browser does not support the video tag.
</video>
{{< /rawhtml >}}

---