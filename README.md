Jenkins test environment for git plugin merge request
===================================
<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:1 -->

- [Purpose](#purpose)
- [Use case](#use-case)
- [Test preparation](#test-preparation)
- [Test scenario](#test-scenario)
- [Test cases](#test-cases)

<!-- /TOC -->

# Purpose
Test merge request https://github.com/jenkinsci/git-plugin/pull/694 for Jenkins git-plugin

# Use case
Using same git repo in different Jenkins jobs causes git rejection while executing git push in case several jobs running at the same time modify something in the repo and try to push their changes. First one pushing wins, other are rejected.

# Test preparation
1. Determination of the expectations for test results of the merge request. The following test variants represents decisive. Their results represents the expected results of tests in Jenkins. Having two clones of this repo locally:
  1. in both clones the same file (`test-files/single-file.txt`) is going to be modified due to appending new rows. From first clone we perform a `git pull && git push` and afterwards we try a `git pull && git rebase && git push`` on the second repo clone. In this case a auto merge on rebase should be possible.
  2. in both clones two different files (`test-files/first-file.txt`,`test-files/second-file.txt`, each one in different git clone folder) are going to be modified due to appending new rows. From first clone we perform a `git pull && git push` and afterwards we try a `git pull && git rebase && git push`` on the second repo clone.  In this case a auto merge on rebase should be possible.
  3. in both clones the same file (`test-files/single-file.txt`) is going to be modified due to replacing all rows. From first clone we perform a `git pull && git push` and afterwards we try a `git pull && git rebase && git push`` on the second repo clone.  In this case a auto merge on rebase should be not possible.
2. A local Jenkins installation is used via docker (`https://hub.docker.com/r/jenkins/jenkins`) managed by docker-compose (`docker-compose.yml`).

# Test scenario
Two similar Jenkins jobs using the same (this) git repository are triggered simultaneously via a trigger job.
`Job A` has a sleep time of 60 seconds, `Job B` a sleep time of 20 seconds. Both jobs will be set up to push their modifications after sleep time expires.
Jenkins version: 2.174
Installation: default plugin selection

# Test cases
1. Jenkins with git-plugin excluding rebase patch.
  a. both jobs are going to modify the same file (`test-files/single-file.txt`) appending new rows.
    - Expected result: git push rejected for `Job A`, but accepted for `Job B`
  b. both jobs are going to modify two different files (`test-files/first-file.txt`,`test-files/second-file.txt`,each one a different in different job) appending new rows.
    - Expected result: git push rejected for `Job A`, but accepted for `Job B`
  c. both jobs are going to modify the same file (`test-files/single-file.txt`) replacing all rows.
    - Expected result: git push rejected for `Job A`, but accepted for `Job B`
    -
2. Jenkins with git-plugin including rebase patch.
  a. both jobs are going to modify the same file (`test-files/single-file.txt`) appending new rows.
    - Expected result: git push accepted for `Job A` and `Job B`
  b. both jobs are going to modify two different files (`test-files/first-file.txt`,`test-files/second-file.txt`,each one a different in different job) appending new rows.
    - Expected result: git push rejected for `Job A` and `Job B`
  c. both jobs are going to modify the same file (`test-files/single-file.txt`) replacing all rows.
    - Expected result: git push rejected for `Job A`, but accepted for `Job B`
