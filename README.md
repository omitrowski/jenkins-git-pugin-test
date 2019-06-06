Jenkins test environment for git plugin merge request
===================================
<!-- TOC depthFrom:1 depthTo:6 withLinks:1 updateOnSave:1 orderedList:1 -->

1. [Purpose](#purpose)
2. [Use case](#use-case)
3. [Test preparation](#test-preparation)
4. [Test preparation results](#test-preparation-results)
		1. [Case a (Details: [results/preparation/case-a/CASE-A.md][e8a25f3b])](#case-a-details-resultspreparationcase-acase-amde8a25f3b)
		2. [Case b (Details: [results/preparation/case-b/CASE-B.md][9dbffb0f])](#case-b-details-resultspreparationcase-bcase-bmd9dbffb0f)
5. [Test scenario](#test-scenario)
6. [Test cases](#test-cases)
7. [Test results](#test-results)
	1. [Take 1](#take-1)
			1. [Case 1.a. (Details: results/take-1/tests-1.a)](#case-1a-details-resultstake-1tests-1a)
			2. [Case 1.b. (Details: results/take-1/tests-1.b)](#case-1b-details-resultstake-1tests-1b)
			3. [Case 2.a. (Details: results/take-1/tests-2.a)](#case-2a-details-resultstake-1tests-2a)
			4. [Case 2.b. (Details: results/take-1/tests-2.b)](#case-2b-details-resultstake-1tests-2b)
			5. [Case 3.a. (Details: results/take-1/tests-3.a)](#case-3a-details-resultstake-1tests-3a)
			6. [Case 3.b. (Details: results/take-1/tests-3.b)](#case-3b-details-resultstake-1tests-3b)
			7. [Take 1 final git repo log](#take-1-final-git-repo-log)
			8. [Take 1 conclusions](#take-1-conclusions)
	2. [Take 2](#take-2)
			1. [Case 2.a. (Details: results/take-2/tests-2.a)](#case-2a-details-resultstake-2tests-2a)
			2. [Case 2.b. (Details: results/take-2/tests-2.b)](#case-2b-details-resultstake-2tests-2b)
			3. [Case 3.a. (Details: results/take-2/tests-3.a)](#case-3a-details-resultstake-2tests-3a)
			4. [Case 3.b. (Details: results/take-2/tests-3.b)](#case-3b-details-resultstake-2tests-3b)
			5. [Take 2 final git repo log](#take-2-final-git-repo-log)
			6. [Take 2 conclusions](#take-2-conclusions)

<!-- /TOC -->

# Purpose
Test merge request https://github.com/jenkinsci/git-plugin/pull/694 for Jenkins git-plugin

# Use case
Using same git repo in different Jenkins jobs causes git rejection while executing git push in case several jobs running at the same time modify something in the repo and try to push their changes. First one pushing wins, other are rejected.

# Test preparation
1. Determination of the expectations for test results of the merge request. The following test variants represents decisive. Their results represents the expected results of tests in Jenkins. Having two clones of this repo locally:
  - **Case a.** in both clones the same file (`test-files/single-file.txt`) is going to be modified due to appending new rows. From first clone we perform a `git pull && git push` and afterwards we try a `git pull && git rebase && git push` on the second repo clone. In this case a auto merge on rebase should be possible.
  - **Case b.** in both clones two different files (`test-files/first-file.txt`,`test-files/second-file.txt`, each one in different git clone folder) are going to be modified due to appending new rows. From first clone we perform a `git pull && git push` and afterwards we try a `git pull && git rebase && git push` on the second repo clone.  In this case a auto merge on rebase should be possible.
  - **Case c.** in both clones the same file (`test-files/single-file.txt`) is going to be modified due to replacing all rows. From first clone we perform a `git pull && git push` and afterwards we try a `git pull && git rebase && git push` on the second repo clone.  In this case a auto merge on rebase should be not possible.
2. A local Jenkins installation is used via docker (`https://hub.docker.com/r/jenkins/jenkins`) managed by docker-compose (`docker-compose.yml`).


# Test preparation results

### Case a (Details: [results/preparation/case-a/CASE-A.md][e8a25f3b])

**1.** As expected git push have been rejected from second git repo clone
- ![1](results/preparation/case-a/1-git-commit-push.jpg)

**2.** Not as expected rebase auto-merge failed
- ![2](results/preparation/case-a/3-git-rebase.jpg)

**3.** Accordingly case **Case c.** is obsolete.

### Case b (Details: [results/preparation/case-b/CASE-B.md][9dbffb0f])

**1.** As expected git push have been rejected from second git repo clone
- ![1](results/preparation/case-b/2-git-push.jpg)

**2.** As expected git push have been accepted after `git pull && git rebase` from second git repo clone
- ![2](results/preparation/case-b/4-git-push-after-rebase.jpg)


# Test scenario
Two similar Jenkins jobs using the same (this) git repository are triggered simultaneously via a trigger job.
`Job A` has a sleep time of 60 seconds, `Job B` a sleep time of 20 seconds. Both jobs will be set up to push their modifications after sleep time expires.

# Test cases
As result of the the preparation process the following test cases and expected results have been determinated:

**1. Jenkins with git-plugin excluding rebase patch:**
  - **Test Case 1.a.** both jobs are going to modify the same file (`test-files/single-file.txt`) appending new rows.
    - Expected result: git push rejected for `Job A`, but accepted for `Job B`
  - **Test Case 1.b.** both jobs are going to modify two different files (`test-files/first-file.txt`,`test-files/second-file.txt`,each one a different in different job) appending new rows.
    - Expected result: git push rejected for `Job A`, but accepted for `Job B`

**2. Jenkins with git-plugin including rebase patch but option `rebase before push` not activated:**
  - **Test Case 2.a.** both jobs are going to modify the same file (`test-files/single-file.txt`) appending new rows.
    - Expected result: git push rejected for `Job A`, but accepted for `Job B`
  - **Test Case 2.b.** both jobs are going to modify two different files (`test-files/first-file.txt`,`test-files/second-file.txt`,each one a different in different job) appending new rows.
    - Expected result: git push rejected for `Job A`, but accepted for `Job B`

**3. Jenkins with git-plugin including rebase patch and activated option `rebase before push`:**
  - **Test Case 3.a.** both jobs are going to modify the same file (`test-files/single-file.txt`) appending new rows.
    - Expected result: git push rejected for `Job A`, but accepted for `Job B`
  - **Test Case 3.b.** both jobs are going to modify two different files (`test-files/first-file.txt`,`test-files/second-file.txt`,each one a different in different job) appending new rows.
    - Expected result: git push accepted for `Job A` and `Job B`

# Test results

## Take 1
#### Case 1.a. (Details: results/take-1/tests-1.a)
  - Expected result: git push rejected for `Job A`, but accepted for `Job B`
  - **Tested OK**
  - Job A
```bash
17:19:44  > git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master
17:19:46 ERROR: Failed to push branch master to origin
17:19:46 hudson.plugins.git.GitException: Command "git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master" returned status code 1:
17:19:46 stdout:
17:19:46 stderr: To github.com:omitrowski/jenkins-git-pugin-test.git
17:19:46  ! [rejected]        HEAD -> master (fetch first)
17:19:46 error: failed to push some refs to 'git@github.com:omitrowski/jenkins-git-pugin-test.git'
17:19:46 hint: Updates were rejected because the remote contains work that you do
17:19:46 hint: not have locally. This is usually caused by another repository pushing
17:19:46 hint: to the same ref. You may want to first integrate the remote changes
17:19:46 hint: (e.g., 'git pull ...') before pushing again.
17:19:46 hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```
  - Job B
```bash
17:19:04 + git commit -am 'modification from Jenkins Test case 1.x/Test Job B - Test case 1a (5)'
17:19:04 [detached HEAD 4f16fa8] modification from Jenkins Test case 1.x/Test Job B - Test case 1a (5)
17:19:04  1 file changed, 1 insertion(+)
17:19:04 using credential jenkins-ssh-key
17:19:04 Pushing HEAD to branch master at repo origin
17:19:04  > git --version # timeout=10
17:19:04 using GIT_SSH to set credentials jenkins-pk
17:19:04  > git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master
17:19:08 Finished: SUCCESS
```

#### Case 1.b. (Details: results/take-1/tests-1.b)
  - Expected result: git push rejected for `Job A`, but accepted for `Job B`
  - **Tested OK**
  - Job A
```bash
17:31:44  > git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master
17:31:46 ERROR: Failed to push branch master to origin
17:31:46 hudson.plugins.git.GitException: Command "git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master" returned status code 1:
17:31:46 stdout:
17:31:46 stderr: To github.com:omitrowski/jenkins-git-pugin-test.git
17:31:46  ! [rejected]        HEAD -> master (fetch first)
17:31:46 error: failed to push some refs to 'git@github.com:omitrowski/jenkins-git-pugin-test.git'
17:31:46 hint: Updates were rejected because the remote contains work that you do
17:31:46 hint: not have locally. This is usually caused by another repository pushing
17:31:46 hint: to the same ref. You may want to first integrate the remote changes
17:31:46 hint: (e.g., 'git pull ...') before pushing again.
17:31:46 hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```
  - Job B
```bash
17:31:04 + git commit -am 'modification from Jenkins Test case 1.x/Test Job B - Test case 1b (1)'
17:31:04 [detached HEAD b89755d] modification from Jenkins Test case 1.x/Test Job B - Test case 1b (1)
17:31:04  1 file changed, 1 insertion(+)
17:31:04 using credential jenkins-ssh-key
17:31:04 Pushing HEAD to branch master at repo origin
17:31:04  > git --version # timeout=10
17:31:04 using GIT_SSH to set credentials jenkins-pk
17:31:04  > git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master
17:31:08 Finished: SUCCESS
```

#### Case 2.a. (Details: results/take-1/tests-2.a)
  - Expected result: git push rejected for `Job A`, but accepted for `Job B`
  - **Tested OK**
  - Job A
```bash
17:47:52  > git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master # timeout=10
17:47:54 ERROR: Failed to push branch master to origin
17:47:54 hudson.plugins.git.GitException: Command "git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master" returned status code 1:
17:47:54 stdout:
17:47:54 stderr: To github.com:omitrowski/jenkins-git-pugin-test.git
17:47:54  ! [rejected]        HEAD -> master (fetch first)
17:47:54 error: failed to push some refs to 'git@github.com:omitrowski/jenkins-git-pugin-test.git'
17:47:54 hint: Updates were rejected because the remote contains work that you do
17:47:54 hint: not have locally. This is usually caused by another repository pushing
17:47:54 hint: to the same ref. You may want to first integrate the remote changes
17:47:54 hint: (e.g., 'git pull ...') before pushing again.
17:47:54 hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```
  - Job B
```bash
17:47:13 + git commit -am 'modification from Jenkins Test case 2.x/Test Job B - Test case 1a (1)'
17:47:13 [detached HEAD 4831e71] modification from Jenkins Test case 2.x/Test Job B - Test case 1a (1)
17:47:13  1 file changed, 1 insertion(+)
17:47:13 using credential jenkins-ssh-key
17:47:13 Pushing HEAD to branch master at repo origin
17:47:13  > git --version # timeout=10
17:47:13 using GIT_SSH to set credentials jenkins-pk
17:47:13  > git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master # timeout=10
```

#### Case 2.b. (Details: results/take-1/tests-2.b)
  - Expected result: git push rejected for `Job A`, but accepted for `Job B`
  - **Tested OK**
  - Job A
```bash
17:52:13  > git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master # timeout=10
17:52:14 ERROR: Failed to push branch master to origin
17:52:14 hudson.plugins.git.GitException: Command "git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master" returned status code 1:
17:52:14 stdout:
17:52:14 stderr: To github.com:omitrowski/jenkins-git-pugin-test.git
17:52:14  ! [rejected]        HEAD -> master (fetch first)
17:52:14 error: failed to push some refs to 'git@github.com:omitrowski/jenkins-git-pugin-test.git'
17:52:14 hint: Updates were rejected because the remote contains work that you do
17:52:14 hint: not have locally. This is usually caused by another repository pushing
17:52:14 hint: to the same ref. You may want to first integrate the remote changes
17:52:14 hint: (e.g., 'git pull ...') before pushing again.
17:52:14 hint: See the 'Note about fast-forwards' in 'git push --help' for details.
```
  - Job B
```bash
17:51:32 + git commit -am 'modification from Jenkins Test case 2.x/Test Job B - Test case 1b (1)'
17:51:32 [detached HEAD 1fb41db] modification from Jenkins Test case 2.x/Test Job B - Test case 1b (1)
17:51:32  1 file changed, 1 insertion(+)
17:51:32 using credential jenkins-ssh-key
17:51:32 Pushing HEAD to branch master at repo origin
17:51:32  > git --version # timeout=10
17:51:32 using GIT_SSH to set credentials jenkins-pk
17:51:32  > git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master # timeout=10
17:51:35 Finished: SUCCESS
```

#### Case 3.a. (Details: results/take-1/tests-3.a)
  - Expected result: git push rejected for `Job A`, but accepted for `Job B`
  - **Tested OK**
  - Job A
```bash
18:14:32  > git fetch --tags --progress git@github.com:omitrowski/jenkins-git-pugin-test.git +refs/heads/*:refs/remotes/origin/* # timeout=10
18:14:34  > git rev-parse HEAD^{commit} # timeout=10
18:14:34  > git rev-parse origin/master^{commit} # timeout=10
18:14:34  > git rebase origin/master # timeout=10
18:14:34  > git rebase --abort # timeout=10
18:14:34 ERROR: Failed to push branch master to origin
18:14:34 hudson.plugins.git.GitException: Could not rebase origin/master
18:14:34 	at org.jenkinsci.plugins.gitclient.CliGitAPIImpl$4.execute(CliGitAPIImpl.java:831)
18:14:34 	at hudson.plugins.git.GitPublisher.perform(GitPublisher.java:310)
18:14:34 	at hudson.tasks.BuildStepMonitor$1.perform(BuildStepMonitor.java:20)
18:14:34 	at hudson.model.AbstractBuild$AbstractBuildExecution.perform(AbstractBuild.java:741)
18:14:34 	at hudson.model.AbstractBuild$AbstractBuildExecution.performAllBuildSteps(AbstractBuild.java:690)
18:14:34 	at hudson.model.Build$BuildExecution.post2(Build.java:186)
18:14:34 	at hudson.model.AbstractBuild$AbstractBuildExecution.post(AbstractBuild.java:635)
18:14:34 	at hudson.model.Run.execute(Run.java:1843)
18:14:34 	at hudson.model.FreeStyleBuild.run(FreeStyleBuild.java:43)
18:14:34 	at hudson.model.ResourceController.execute(ResourceController.java:97)
18:14:34 	at hudson.model.Executor.run(Executor.java:429)
18:14:34 Caused by: hudson.plugins.git.GitException: Command "git rebase origin/master" returned status code 128:
18:14:34 stdout: First, rewinding head to replay your work on top of it...
18:14:34 Applying: modification from Jenkins Test case 3.x/Test Job A - Test case 3a (1)
18:14:34 Using index info to reconstruct a base tree...
18:14:34 M	test-files/single-file.txt
18:14:34 Falling back to patching base and 3-way merge...
18:14:34 Auto-merging test-files/single-file.txt
18:14:34 CONFLICT (content): Merge conflict in test-files/single-file.txt
18:14:34 Patch failed at 0001 modification from Jenkins Test case 3.x/Test Job A - Test case 3a (1)
18:14:34 The copy of the patch that failed is found in: .git/rebase-apply/patch
18:14:34
18:14:34 When you have resolved this problem, run "git rebase --continue".
18:14:34 If you prefer to skip this patch, run "git rebase --skip" instead.
18:14:34 To check out the original branch and stop rebasing, run "git rebase --abort".
```
  - Job B
```bash
18:13:53  > git fetch --tags --progress git@github.com:omitrowski/jenkins-git-pugin-test.git +refs/heads/*:refs/remotes/origin/* # timeout=10
18:13:54  > git rev-parse HEAD^{commit} # timeout=10
18:13:54  > git rev-parse origin/master^{commit} # timeout=10
18:13:54  > git rebase origin/master # timeout=10
18:13:54 Pushing HEAD to branch master at repo origin
18:13:54 using GIT_SSH to set credentials jenkins-pk
18:13:54  > git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master # timeout=10
18:13:57 Finished: SUCCESS
```

#### Case 3.b. (Details: results/take-1/tests-3.b)
  - Expected result: git push accepted for `Job A` and `Job B`
  - **Tested OK**
  - Job A
```bash
18:19:17  > git fetch --tags --progress git@github.com:omitrowski/jenkins-git-pugin-test.git +refs/heads/*:refs/remotes/origin/* # timeout=10
18:19:19  > git rev-parse HEAD^{commit} # timeout=10
18:19:19  > git rev-parse origin/master^{commit} # timeout=10
18:19:19  > git rebase origin/master # timeout=10
18:19:19 Pushing HEAD to branch master at repo origin
18:19:19 using GIT_SSH to set credentials jenkins-pk
18:19:19  > git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master # timeout=10
18:19:22 Finished: SUCCESS
```
  - Job B
```bash
18:18:38  > git fetch --tags --progress git@github.com:omitrowski/jenkins-git-pugin-test.git +refs/heads/*:refs/remotes/origin/* # timeout=10
18:18:39  > git rev-parse HEAD^{commit} # timeout=10
18:18:39  > git rev-parse origin/master^{commit} # timeout=10
18:18:39  > git rebase origin/master # timeout=10
18:18:39 Pushing HEAD to branch master at repo origin
18:18:39 using GIT_SSH to set credentials jenkins-pk
18:18:39  > git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master # timeout=10
18:18:42 Finished: SUCCESS
```
- Endurance test run over 60 times the Test Case 3.b. without issues.

#### Take 1 final git repo log
```bash
* 41af67f modification from Jenkins Test case 3.x/Test Job A - Test case 3b (1)
* d6de91f modification from Jenkins Test case 3.x/Test Job B - Test case 3b (1)
* 2615e10 modification from Jenkins Test case 3.x/Test Job B - Test case 3a (1)
* 1fb41db modification from Jenkins Test case 2.x/Test Job B - Test case 1b (1)
* 4831e71 modification from Jenkins Test case 2.x/Test Job B - Test case 1a (1)
* b89755d modification from Jenkins Test case 1.x/Test Job B - Test case 1b (1)
* 4f16fa8 modification from Jenkins Test case 1.x/Test Job B - Test case 1a (5)
* 89c6a10 modification from Jenkins  (4)
* 1f248ae modification from Jenkins  (3)
* 7160927 modification from Jenkins  (2)
* 94d2b61 [MOD] some editing
* 869caab [MOD] Test preparation case b
* 665263c Test preparation case b, manual modification from clone #2
* 8380bf6 Test preparation case b, manual modification from clone #1
* 25f22cf [FIX] reset first file
* 659685b [MOD] Test preparation case a
*   7d4cd7d Merge branch 'master' of github.com:omitrowski/jenkins-git-pugin-test
|\
| * b64f1bc manual modification from clone #1
* | 9ba7116 manual modification from clone #2
|/
* c61e8f5 [MOD] readme formatting
* 0b15f4f [ADD] test scenario ideas
* 3da336b Initial commit
```

#### Take 1 conclusions

 - The patch from the merge request behaves predictable and meets the expected functionality.
 - It does not change the git plugin behavior if option `rebase before push` is not activated.
 - In case of errors the messages are understandable and helpful.
 - As expected in the git log you don't see merges on successful rebase.
 - Endurance test run over 60 times without issues.

## Take 2

#### Case 2.a. (Details: results/take-2/tests-2.a)
  - Expected result: git push rejected for `Job A`, but accepted for `Job B`
  - **Tested OK**
  - Job A
```bash
21:38:09  > git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master # timeout=10
21:38:10 ERROR: Failed to push branch master to origin
21:38:10 hudson.plugins.git.GitException: Command "git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master" returned status code 1:
21:38:10 stdout:
21:38:10 stderr: To github.com:omitrowski/jenkins-git-pugin-test.git
21:38:10  ! [rejected]        HEAD -> master (fetch first)
21:38:10 error: failed to push some refs to 'git@github.com:omitrowski/jenkins-git-pugin-test.git'
```
  - Job B
```bash
21:37:29 + git commit -am 'modification from Jenkins Test case 2.x/Test Job B - Test case 1a (2)'
21:37:29 [detached HEAD c412ada] modification from Jenkins Test case 2.x/Test Job B - Test case 1a (2)
21:37:29  1 file changed, 1 insertion(+)
21:37:29 using credential jenkins-ssh-key
21:37:29 Pushing HEAD to branch master at repo origin
21:37:29  > git --version # timeout=10
21:37:29 using GIT_SSH to set credentials jenkins-pk
21:37:29  > git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master # timeout=10
```

#### Case 2.b. (Details: results/take-2/tests-2.b)
  - Expected result: git push rejected for `Job A`, but accepted for `Job B`
  - **Tested OK**
  - Job A
```bash
21:53:51 ERROR: Failed to push branch master to origin
21:53:51 hudson.plugins.git.GitException: Command "git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master" returned status code 1:
21:53:51 stdout:
21:53:51 stderr: To github.com:omitrowski/jenkins-git-pugin-test.git
21:53:51  ! [rejected]        HEAD -> master (fetch first)
21:53:51 error: failed to push some refs to 'git@github.com:omitrowski/jenkins-git-pugin-test.git'
```
  - Job B
```bash
21:53:10 + git commit -am 'modification from Jenkins Test case 2.x/Test Job B - Test case 1b (2)'
21:53:10 [detached HEAD 6feb9dc] modification from Jenkins Test case 2.x/Test Job B - Test case 1b (2)
21:53:10  1 file changed, 1 insertion(+)
21:53:10 using credential jenkins-ssh-key
21:53:10 Pushing HEAD to branch master at repo origin
```

#### Case 3.a. (Details: results/take-2/tests-3.a)
  - Expected result: git push rejected for `Job A`, but accepted for `Job B`
  - **Tested OK**
  - Job A
```bash
21:59:39 [detached HEAD 3c19e0a] modification from Jenkins Test case 3.x/Test Job A - Test case 3a (2)
21:59:39  1 file changed, 1 insertion(+)
21:59:39 using credential jenkins-ssh-key
21:59:39 Fetch and rebase with master of origin
21:59:39 Fetching upstream changes from git@github.com:omitrowski/jenkins-git-pugin-test.git
21:59:39  > git --version # timeout=10
21:59:39 using GIT_SSH to set credentials jenkins-pk
21:59:39  > git fetch --tags --progress git@github.com:omitrowski/jenkins-git-pugin-test.git +refs/heads/*:refs/remotes/origin/* # timeout=10
21:59:41  > git rev-parse HEAD^{commit} # timeout=10
21:59:41  > git rev-parse origin/master^{commit} # timeout=10
21:59:41  > git rebase origin/master # timeout=10
21:59:41  > git rebase --abort # timeout=10
21:59:41 ERROR: Failed to push branch master to origin
...
21:59:41 M	test-files/single-file.txt
21:59:41 Falling back to patching base and 3-way merge...
21:59:41 Auto-merging test-files/single-file.txt
21:59:41 CONFLICT (content): Merge conflict in test-files/single-file.txt
21:59:41 Patch failed at 0001 modification from Jenkins Test case 3.x/Test Job A - Test case 3a (2)
21:59:41 The copy of the patch that failed is found in: .git/rebase-apply/patch
21:59:41
21:59:41 When you have resolved this problem, run "git rebase --continue".
21:59:41 If you prefer to skip this patch, run "git rebase --skip" instead.
21:59:41 To check out the original branch and stop rebasing, run "git rebase --abort".
...
21:59:41 stderr: error: Failed to merge in the changes.
```
  - Job B
```bash
21:59:00 + git commit -am 'modification from Jenkins Test case 3.x/Test Job B - Test case 3a (2)'
21:59:00 [detached HEAD a7e84fe] modification from Jenkins Test case 3.x/Test Job B - Test case 3a (2)
21:59:00  1 file changed, 1 insertion(+)
21:59:00 using credential jenkins-ssh-key
21:59:00 Fetch and rebase with master of origin
21:59:00 Fetching upstream changes from git@github.com:omitrowski/jenkins-git-pugin-test.git
21:59:00  > git --version # timeout=10
21:59:00 using GIT_SSH to set credentials jenkins-pk
21:59:00  > git fetch --tags --progress git@github.com:omitrowski/jenkins-git-pugin-test.git +refs/heads/*:refs/remotes/origin/* # timeout=10
21:59:02  > git rev-parse HEAD^{commit} # timeout=10
21:59:02  > git rev-parse origin/master^{commit} # timeout=10
21:59:02  > git rebase origin/master # timeout=10
21:59:02 Pushing HEAD to branch master at repo origin
```

#### Case 3.b. (Details: results/take-2/tests-3.b)
  - Expected result: git push accepted for `Job A` and `Job B`
  - **Tested OK**
  - Job A
```bash
22:03:50 diff --git a/test-files/first-file.txt b/test-files/first-file.txt
22:03:50 index 664cef9..62d170e 100644
22:03:50 --- a/test-files/first-file.txt
22:03:50 +++ b/test-files/first-file.txt
22:03:50 @@ -66,3 +66,4 @@ modification from Jenkins Test case 3.x/Test Job A - Test case 3b (64)
22:03:50  modification from Jenkins Test case 3.x/Test Job A - Test case 3b (65)
22:03:50  modification from Jenkins Test case 3.x/Test Job A - Test case 3b (66)
22:03:50  modification from Jenkins Test case 3.x/Test Job A - Test case 3b (67)
22:03:50 +modification from Jenkins Test case 3.x/Test Job A - Test case 3b (70)
22:03:50 + sleep 60
22:04:50 + git commit -am 'modification from Jenkins Test case 3.x/Test Job A - Test case 3b (70)'
22:04:50 [detached HEAD e46797e] modification from Jenkins Test case 3.x/Test Job A - Test case 3b (70)
22:04:50  1 file changed, 1 insertion(+)
22:04:50 using credential jenkins-ssh-key
22:04:50 Fetch and rebase with master of origin
22:04:50 Fetching upstream changes from git@github.com:omitrowski/jenkins-git-pugin-test.git
22:04:50  > git --version # timeout=10
22:04:50 using GIT_SSH to set credentials jenkins-pk
22:04:50  > git fetch --tags --progress git@github.com:omitrowski/jenkins-git-pugin-test.git +refs/heads/*:refs/remotes/origin/* # timeout=10
22:04:52  > git rev-parse HEAD^{commit} # timeout=10
22:04:52  > git rev-parse origin/master^{commit} # timeout=10
22:04:52  > git rebase origin/master # timeout=10
22:04:52 Pushing HEAD to branch master at repo origin
22:04:52 using GIT_SSH to set credentials jenkins-pk
22:04:52  > git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master # timeout=10
22:04:56 Finished: SUCCESS
```
  - Job B
```bash
22:03:50 diff --git a/test-files/second-file.txt b/test-files/second-file.txt
22:03:50 index 53a55a2..7a921fc 100644
22:03:50 --- a/test-files/second-file.txt
22:03:50 +++ b/test-files/second-file.txt
22:03:50 @@ -71,3 +71,4 @@ modification from Jenkins Test case 3.x/Test Job B - Test case 3b (66)
22:03:50  modification from Jenkins Test case 3.x/Test Job B - Test case 3b (67)
22:03:50  modification from Jenkins Test case 3.x/Test Job B - Test case 3b (68)
22:03:50  modification from Jenkins Test case 2.x/Test Job B - Test case 1b (2)
22:03:50 +modification from Jenkins Test case 3.x/Test Job B - Test case 3b (70)
22:03:50 + sleep 20
22:04:10 + git commit -am 'modification from Jenkins Test case 3.x/Test Job B - Test case 3b (70)'
22:04:10 [detached HEAD cbea7d3] modification from Jenkins Test case 3.x/Test Job B - Test case 3b (70)
22:04:10  1 file changed, 1 insertion(+)
22:04:10 using credential jenkins-ssh-key
22:04:10 Fetch and rebase with master of origin
22:04:10 Fetching upstream changes from git@github.com:omitrowski/jenkins-git-pugin-test.git
22:04:10  > git --version # timeout=10
22:04:10 using GIT_SSH to set credentials jenkins-pk
22:04:10  > git fetch --tags --progress git@github.com:omitrowski/jenkins-git-pugin-test.git +refs/heads/*:refs/remotes/origin/* # timeout=10
22:04:13  > git rev-parse HEAD^{commit} # timeout=10
22:04:13  > git rev-parse origin/master^{commit} # timeout=10
22:04:13  > git rebase origin/master # timeout=10
22:04:13 Pushing HEAD to branch master at repo origin
22:04:13 using GIT_SSH to set credentials jenkins-pk
22:04:13  > git push git@github.com:omitrowski/jenkins-git-pugin-test.git HEAD:master # timeout=10
22:04:17 Finished: SUCCESS
```
- Endurance test run over 60 times the Test Case 3.b. without issues.

#### Take 2 final git repo log
```bash
* 9cfe746 (HEAD -> master, origin/master, origin/HEAD) modification from Jenkins Test case 3.x/Test Job A - Test case 3b (70)
* cbea7d3 modification from Jenkins Test case 3.x/Test Job B - Test case 3b (70)
* a7e84fe modification from Jenkins Test case 3.x/Test Job B - Test case 3a (2)
* 6feb9dc modification from Jenkins Test case 2.x/Test Job B - Test case 1b (2)
* c412ada modification from Jenkins Test case 2.x/Test Job B - Test case 1a (2)
...
```

#### Take 2 conclusions

 - The patch from the merge request behaves predictable and meets the expected functionality.
 - It does not change the git plugin behavior if option `rebase before push` is not activated.
 - In case of errors the messages are understandable and helpful.
 - As expected in the git log you don't see merges on successful rebase.
 - Endurance test run over 90 times without issues.

[e8a25f3b]: results/preparation/case-a/CASE-A.md "CASE-A.md"
[9dbffb0f]: results/preparation/case-b/CASE-B.md "CASE-B.md"
