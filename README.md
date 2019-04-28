Jenkins test environment for git plugin merge request
===================================
<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:1 -->

- [Purpose](#purpose)
- [Use case](#use-case)
- [Test preparation](#test-preparation)
- [Test scenario](#test-scenario)
- [Test environment](#test-environment)
- [Test cases](#test-cases)
- [Results](#results)
  - [Test preparation results](#test-preparation-results)
    - [Case 1.a (Screen: `results/preparation/case-1a/`)](#case-1a-screen-resultspreparationcase-1a)

<!-- /TOC -->

# Purpose
Test merge request https://github.com/jenkinsci/git-plugin/pull/694 for Jenkins git-plugin

# Use case
Using same git repo in different Jenkins jobs causes git rejection while executing git push in case several jobs running at the same time modify something in the repo and try to push their changes. First one pushing wins, other are rejected.

# Test preparation
1. Determination of the expectations for test results of the merge request. The following test variants represents decisive. Their results represents the expected results of tests in Jenkins. Having two clones of this repo locally:
  - **Case a.** in both clones the same file (`test-files/single-file.txt`) is going to be modified due to appending new rows. From first clone we perform a `git pull && git push` and afterwards we try a `git pull && git rebase && git push`` on the second repo clone. In this case a auto merge on rebase should be possible.
  - **Case b.** in both clones two different files (`test-files/first-file.txt`,`test-files/second-file.txt`, each one in different git clone folder) are going to be modified due to appending new rows. From first clone we perform a `git pull && git push` and afterwards we try a `git pull && git rebase && git push`` on the second repo clone.  In this case a auto merge on rebase should be possible.
  - **Case c.** in both clones the same file (`test-files/single-file.txt`) is going to be modified due to replacing all rows. From first clone we perform a `git pull && git push` and afterwards we try a `git pull && git rebase && git push`` on the second repo clone.  In this case a auto merge on rebase should be not possible.
2. A local Jenkins installation is used via docker (`https://hub.docker.com/r/jenkins/jenkins`) managed by docker-compose (`docker-compose.yml`).

# Test scenario
Two similar Jenkins jobs using the same (this) git repository are triggered simultaneously via a trigger job.
`Job A` has a sleep time of 60 seconds, `Job B` a sleep time of 20 seconds. Both jobs will be set up to push their modifications after sleep time expires.

# Test environment
- Operating System: Linux, Ubuntu 18.04.2 LTS
- Kernel Version: 4.18.0-18-generic
- Architecture: x86_64
- CPUs: 8
- Total Memory: 15.36GiB
- Docker version: 18.09.2
- Git versions:
  - locally: 2.17.1
  - remote: github.com
- Jenkins version: 2.174
  - For all test cases:
    - Installation procedure: default plugin selection
  - For test case 2 (Jenkins with git-plugin including rebase patch):
    - Post installation of git client v3.0.0-beta9-rc1944 over v2.7.7 from https://ci.jenkins.io/job/Plugins/job/git-client-plugin/job/master/295/artifact/org/jenkins-ci/plugins/git-client/3.0.0-beta9-rc1944.926401690c63/git-client-3.0.0-beta9-rc1944.926401690c63.hpi
    - Post installation of git plugin v4.0.0-beta9-rc3094 over v3.9.4 from https://ci.jenkins.io/job/Plugins/job/git-plugin/job/PR-694/5/artifact/org/jenkins-ci/plugins/git/4.0.0-beta9-rc3094.5a8adf136d1b/git-4.0.0-beta9-rc3094.5a8adf136d1b.hpi


# Test cases
1. Jenkins with git-plugin excluding rebase patch:
  - **Test Case 1.a.** both jobs are going to modify the same file (`test-files/single-file.txt`) appending new rows.
    - Expected result: git push rejected for `Job A`, but accepted for `Job B`
  - **Test Case 1.b.** both jobs are going to modify two different files (`test-files/first-file.txt`,`test-files/second-file.txt`,each one a different in different job) appending new rows.
    - Expected result: git push rejected for `Job A`, but accepted for `Job B`

2. Jenkins with git-plugin including rebase patch:
  - **Test Case 2.a.** both jobs are going to modify the same file (`test-files/single-file.txt`) appending new rows.
    - Expected result: git push rejected for `Job A` and `Job B`
  - **Test Case 2.b.** both jobs are going to modify two different files (`test-files/first-file.txt`,`test-files/second-file.txt`,each one a different in different job) appending new rows.
    - Expected result: git push accepted for `Job A` and `Job B`


# Results

## Test preparation results

### Case a ([results/preparation/case-a/CASE-A.md][e8a25f3b])

**1.** As expected git push have been rejected from second git repo clone
- ![1](results/preparation/case-a/1-git-commit-push.jpg)

**2.** Not as expected rebase auto-merge failed
- ![2](results/preparation/case-a/3-git-rebase.jpg)

**3.** Accordingly case **Case c.** is obsolete.

### Case a ([results/preparation/case-b/CASE-B.md][9dbffb0f])

**1.** As expected git push have been rejected from second git repo clone
- ![1](results/preparation/case-b/2-git-push.jpg)

**2.** As expected git push have been accepted after `git pull && git rebase` from second git repo clone
- ![2](results/preparation/case-b/4-git-push-after-rebase.jpg)



[e8a25f3b]: results/preparation/case-a/CASE-A.md "CASE-A.md"
[9dbffb0f]: results/preparation/case-b/CASE-B.md "CASE-B.md"
