The OpenCV repository has several branches with different contribution policies.

Common rules for all branches
-----------------------------

* Your development branch name must differ from the names of branches in the upstream OpenCV repository, i.e. your branch must **NOT** be named _2.4_, _3.4_, _4.x_, _5.x_, or _master_/_next_
  (it's a technical requirement specific to our continuous integration system to properly test multi-repository patches with opencv_extra/opencv_contrib)
* Multiple related commits should be [squashed](http://git-scm.com/book/en/Git-Tools-Rewriting-History#Squashing-Commits) into one.
  A pull request must contain either a single commit, or several unrelated commits (e.g, commit with test + commit with code fix)


Branches and contribution policies in OpenCV
--------------------------------------------

Please target Pull Requests (PR) to the right branches. Quick summary of the rules is presented in the table below. More formal description is in the sections after the table.

### Which branch should I target my PR?

Policy                                |   5.x              |    4.x                |
------------------------------------- | ------------------ | --------------------- |
Preserve API compatibility            | :heavy_check_mark: | :heavy_check_mark:    |
Preserve ABI compatibility            | :x:                | :x::grey_exclamation: |
Change applicable only to this branch | :heavy_check_mark: | :heavy_check_mark:    |
Bugfix / minor fix                    | :point_right:      | :heavy_check_mark:    |
Optimization                          | :point_right:      | :heavy_check_mark:    |
Small feature                         | :point_right:      | :heavy_check_mark:    |
Large feature                         | :heavy_check_mark: | :x:                   |
Branch alias (do not use!)            | next               | master                |

- :grey_exclamation: - avoid major breakages
- :point_right: - rebase to previous branch if patch is applicable, it will be ported to other branches by the Core team in a week or two

### 5.x

This is the development branch for upcoming [5.x](https://github.com/opencv/opencv/tree/5.x) releases.

* API compatibility must be preserved
* If your pull request contains a bug fix which is also applicable to the _3.4_ or _4.x_ branch, you should choose that branch as "base"
* If you've already created pull request based on the _5.x_ branch, but it is also applicable to _4.x_, you will be asked to rebase it to 4.x/3.4 branches, see the instruction in the following section.


### 4.x

This is the branch for [4.x](https://github.com/opencv/opencv/tree/4.x) releases.

* API compatibility must be preserved
* Incompatible improvements or large features should be targeted to _5.x_ branch
* Compatible improvements or small features should go to _4.x_ branch
* If your pull request contains a bug fix which is also applicable to the _3.4_ branch, you should choose that branch as "base"
* If you've already created pull request based on the _4.x_ branch, but it is also applicable to _3.4_, you will be asked to rebase it to 3.4, see the instruction in the following section.
* We will merge changes from _4.x_ into _5.x_ regularly (weekly/bi-weekly)
  There is no regular process for backporting patches from _5.x_ branch

### EOL branches

EOL branches:
- "3.4": this is the branch for [3.4.x](https://github.com/opencv/opencv/tree/3.4) releases. This release series is EOL in 2023. CI is turned off, so no more patches are accepted to 3.4 branch.

- "2.4": this is the branch for [2.4.x](https://github.com/opencv/opencv/tree/2.4) releases. This release series is EOL in 2020. CI is turned off, so no more patches are accepted to 2.4 branch.


### master / next

"master" branch is a shadow copy of 4.x development branch kept for compatibility purposes. After 5.0 release this branch will shadow _5.x_ branch.
"next" branch is a shadow copy of 5.x development branch kept for compatibility purposes. It will be removed after 5.0 release.
Merge to this branches is prohibited. Use 4.x / 5.x as target branches for your PRs.


Rebasing a pull request between branches
----------------------------------------

_If you can not do this by yourself, please ask maintainers for help._

Example of rebasing from 5.x branch to 4.x branch:
* do not close the existing pull request
* change the "base" branch of the pull request:
  * open PR on GitHub in your browser
  * press the "Edit" button near the pull request title
  * choose "4.x" from the dropdown list
* rebase your commits from 5.x onto 4.x:
  * `git checkout <your-branch>`
  * (optional) create a backup branch: `git branch <your-branch>-backup`
  * `git remote update upstream` (assuming _upstream_ [is pointing](https://help.github.com/articles/configuring-a-remote-for-a-fork/) to the `opencv/opencv` GitHub repository)
  * `git rebase -i --onto upstream/4.x upstream/5.x`
  * an editor will be opened. Please check the list of commits - there should be only your commits - save and exit
  * `git push --force origin <your-branch>` (assuming _origin_ [is pointing](https://help.github.com/articles/configuring-a-remote-for-a-fork/) to `<your-username>/opencv` forked GitHub repository)


Related articles
----------------

* [GitHub Flow](https://guides.github.com/introduction/flow/) guide
* [Forking Projects](https://guides.github.com/activities/forking/) guide
* [ABI Compliance Checker](https://lvc.github.io/abi-compliance-checker/) tool
