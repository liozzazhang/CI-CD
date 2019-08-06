# CI / CD

## OverView
The new architecture is based on  [Gitlab-CI](https://about.gitlab.com/product/continuous-integration/)  and  [Gitlab Flow](https://docs.gitlab.com/ee/workflow/gitlab_flow.html).
## Architecture
![](images/ci-cd-architecture.png)
## Components
### Code
By default, each commit will trigger a pipeline automatically exclude commit message contains **[skip ci].**
### CI PIPELINE
![](images/image2019-8-1_14-6-17.png)
This components contains **FIVE STAGES** which are **BasicCheck, Build, SelfCheck, DeployDev, Auto-test**. It automatic runs totally.

**BasicCheck**
- check Dockerfile
- check harbor permissions
- check image name

**Build**
- build docker image
- generate helm package
- update helm version in consul

**SelfCheck**
- scan image security
- analyse image size

**DeployDev**
- auto deploy to Dev

**AutoTest**
- health check

### MERGE
![](images/image2019-8-1_14-8-4.png)
> This component connects CI and CD pipeline, also it plays an important role in cross branches operation. Usually, engineer develops in dev branch. CI pipeline ensures the new feature quality. Once some feature needs integration test, engineer will merge related code to sit branch which will enter in CD pipeline.

### CD PIPELINE
![](images/image2019-8-1_14-8-58.png)
- build sit

- deploy sit

- deploy uat

- deploy prod

- smoke test

### ARCHIVE

![](images/image2019-8-1_14-9-14.png)

> Each release of prod, we will archive not only gitlab, but also helm package in case of roll back.

### Integration & Manual

#### OverView
First of all, everything is based on GitLab CI which means you must have **a gitlab project** to start integration. I will provide an example to explain how to integrate.
#### Example

**Step 1**
Create a file named *.gitlab-ci.yml* under your project root dir. 

![](images/manual-01.png)
> This project, unified-exam-service, is going to integrate with GitLab CI. The .gitlab-ci.yml is located in the project root.

**Step 2**
Modify .gitlab-ci-yml as follows.
```bash
# vi .gitlab-ci.yml

variables:
  INCLUDE: “main.yml” # this is required! It defines what template to include. At present, we only provide `main.yml` template.

init-project:
  script:
    - git archive —remote=git@gitlab.aidoin.com:DevOps/templates.git  HEAD:gitlab-ci init.py |tar -x # this is required! Please DO NOT change it!
    - python3 init.py # this is required! Please DO NOT change it!
  tags:
    - init # this is required! Please DO NOT change it!
  artifacts:
    paths:
      - gitlab-ci-init.yml # this is required! Please DO NOT change it!
```
**Step 3**
1. Commit and push .gitlab-ci.yml. Then check your project website left menu ![](images/manual-02.png).
![](images/manual-03.png)

2. You will find a pipeline which looks like above image. Then click ![](images/manual-04.png)icon.
![](images/manual-05.png)

3. Then click ![](images/manual-06.png), you will see the notice message “please download artifacts and rename gitlab-ci-init.yml to .gitlab-ci.yml under your project root dir!”
![](images/manual-07.png)

4. Then click the ![](images/manual-08.png) in the right area.

5. At last overwrite your .gitlab-ci.yml with the downloaded file then commit and push it!
![](images/manual-09.png)

6. If you see the above pipeline under your CI/CD menu, congratulations! Happy GitLab CI!

### Requirement

1. All build is based on Dockerfile.

2. All deploy is based on Helm.

### Advantages & Disadvantages

![](images/94DC291F-8E0C-46EB-BAD4-303E27544D25.png)