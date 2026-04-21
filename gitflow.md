# Gitflow

Gitflow workflow is a branching model for git developed by Vincent Driessen in 2010 and became the most popular branching workflow out there. This workflow aims to manage large projects by defining a strict branching model to facilitate the project development and release .  
Gitflow is based on two main branches `master` and `develop` and other supporting branches. It defines specific roles to different branches and specifies how and when these branches should interact.

### Git-flow Extension (Optional)

Gitflow by itself is a work pattern which does not nee any extra tools besides regular [Git commands](https://git-scm.com/docs). However, there are some tools that help facilitate the use of Gitflow and enforce naming and branching conventions - one of which is the Gitflow Extension

- On OSX systems, execute brew `install git-flow`
- On Windows git-flow commands are available with the regular [Git installation](https://git-scm.com/download/win).

We will discuss how to use the extension for every usecase in the coming sections.

### Develop and Master Branches

This workflow uses two branches to record the history of the project:

- `origin/master` is the main branch where the source code of HEAD always reflects a production-ready state.
- `origin/develop` to be the main branch where the source code of HEAD always reflects a state with the latest delivered development changes for the next release.

 <img src="https://wac-cdn.atlassian.com/dam/jcr:2bef0bef-22bc-4485-94b9-a9422f70f11c/02%20(2).svg?cdnVersion=1429" alt="drawing" width="50%" style=" display: block; margin-left: auto; margin-right: auto;margin-bottom: 5%;margin-top: 5%;"/>

When changes on the `develop` branch are stable for a release, they are somehow merged with the `master` branch (merging methods will come later on).
Hence, each merge with the `master` branch is a **new production release** by defenition. This rule is very strict and implies that we may implement a [Git hook script](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) that automatically builds and rolls-out code in the `master` branch to production servers everytime there is a new commit on `master`.

##### Initializing the Repository

- **Without the git-flow extension:**
  First, it is required to complement the default `master` with a `develop` branch. To do this, one developer should open an empty `develop` branch locally from `origin/master` and push it to the server:

  ```shell
  $ git branch develop
  $ git push -u origin develop
  ```

- **When using the git-flow extension:**
  When executing the command `git flow init`, git-flow extension will produce the following output:

  ```shell
  $ git flow init


  Initialized empty Git repository in ~/project/.git/
  No branches exist yet. Base branches must be created now.
  Branch name for production releases: [master]
  Branch name for "next release" development: [develop]


  How to name your supporting branch prefixes?
  Feature branches? [feature/]
  Release branches? [release/]
  Hotfix branches? [hotfix/]
  Support branches? [support/]
  Version tag prefix? []


  $ git branch
  * develop
   master
  ```

  > Notice that the original prefix for each branch type can be customized

### Supporting branches

Along with the main branches `master` and `develop`, this model uses several supporting branches to help track features easily, prepare for production releases and fix live production problems quickly.
Unlike the main branches, these branches always have a short time-span, since they will be removed eventually.

- `feature/*` — feature branches are used to develop new features for the upcoming releases.
- `release/*` — release branches support preparation of a new production release. They allow many minor bug to be fixed and preparation of meta-data for a release.
- `hotfix/*` — hotfix branches are necessary to act immediately upon an undesired status of `master`.

Each of these branches have a defined purpose and are bound to strict rules concerning their orginal `HEAD` from which they are pulled and their corresponding target to which they are merged. These are regular old git branches, however, their naming is based on their usecase.

#### Feature Branches

Each new feature for a release has its own branch. However, instead of branching off of `master`, feature branches use `develop` as their parent branch. The feature branch exists as long as the feature is being developed, so once a feature is complete, it gets merged back into `develop` or gets discarded. Features should never interact directly with master.

 <img src="https://wac-cdn.atlassian.com/dam/jcr:b5259cce-6245-49f2-b89b-9871f9ee3fa4/03%20(2).svg?cdnVersion=1429" alt="drawing" width="50%" style=" display: block; margin-left: auto; margin-right: auto;margin-bottom: 5%;margin-top: 5%;"/>

| May branch off from: | Should merge back into: | Naming convention: |
| -------------------- | ----------------------- | ------------------ |
| `develop`            | `develop`               | `feature/*`        |

##### Opening a feature branch

Assuming this new feature branch will have the name `feature/my_new_feature`

- **Without the git-flow extension:**

  ```shell
  $ git checkout -b feature/my_new_feature develop
  ```

- **When using the git-flow extension:**
  ```shell
  $ git flow feature start my_new_feature
  ```
  > Notice how the git-flow extension does not require adding the prefix `feature/` as it has been preconfigured

##### Finishing a feature branch

- **Without the git-flow extension:**

  1 - Merge with `--no-ff` flag (will discuss why)

  ```shell
  $ git checkout develop
  Switched to branch 'develop'
  $ git merge --no-ff <feature_branch>
  Updating ea1b82a..05e9557
  (Summary of changes)
  ```

  2 - Delete the fetaure branch

  ```shell
  $ git branch -d <feature_branch>
  Deleted branch <feature_branch> (was 05e9557).
  ```

  3- Push to `develop`

  ```shell
  $ git push origin develop
  ```

- **When using the git-flow extension:**
  ```shell
  $ git flow feature finish -F <feature_branch>
  ```
  The `-F` flag is used to delete the feature branch (based on git-flow pattern)

#### Release Branches

Once `develop` acquires enough features for a release based on sprint-planning or an predefined MVP, a release branch is forked from `develop`. Opening this branch implies the start of the next release cycle, so no new features can be added after to this current release branch, however, only the following can be added: bug fixes, documentation, and other release-oriented tasks. Only at this point, a version number for the upcoming release is assigned. Once the software is ready to ship, the release branch is merged into `master` and tagged with the release version number. In addition, it should be merged back into develop, since the work on develop may have progressed with new features for the next release.

This separation of concerns makes it enables one team to work on final fixes and tweaks to the current release, while another team can in parallel work on new features for an upcoming release.

 <img src="https://wac-cdn.atlassian.com/dam/jcr:a9cea7b7-23c3-41a7-a4e0-affa053d9ea7/04%20(1).svg?cdnVersion=1429" alt="drawing" width="50%" style=" display: block; margin-left: auto; margin-right: auto;margin-bottom: 5%;margin-top: 5%;"/>

| May branch off from: | Should merge back into: | Naming convention: |
| -------------------- | ----------------------- | ------------------ |
| `develop`            | `develop` and `master`  | `release/*`        |

##### Opening a release branch

Assuming changes in `develop` are finalized for the "new release" and this release will have a version number `0.1.0` :

- **Without the git-flow extension:**

  ```shell
  $ git checkout -b release/0.1.0 develop
  Switched to a new branch "release/0.1.0"
  ```

- **When using the git-flow extension:**
  ```shell
  $ git flow release start 0.1.0
  Switched to a new branch 'release/0.1.0'
  ```
  > Notice how the git-flow extension does not require adding the prefix `release/` as it has been preconfigured

After opening a new release branch named `release/0.1.0`, it is likely to make these changes:

```
$ ./bump-version.sh 0.1.0
Files modified successfully, version bumped to 0.1.0.
$ git commit -a -m "Bumped version number to 0.1.0"
[release/0.1.0 74d9424] Bumped version number to 0.1.0
1 files changed, 1 insertions(+), 1 deletions(-)
```

Here, `bump-version.sh` is just a fictional shell script that modifies some files to reflect the new version. Of course, these changes might be done manually (changing version number in `package.json` for example). Then, the bumped version number is committed.

This new branch could exist there for a while, until the release is finally merged with `master`. During this time, developers can apply bug fixes to this branch as opposed to fixing them on `develop`. However, adding new large features here is strictly prohibited as these changes should have been pre-planned for the release and merged into develop in the first place. In this case, these changes can wait for the next release and worked on in feature branches.

##### Finishing a release branch

Once the the release branch is ready for production, some steps are made to deploy. First, the release branch is merged into master since, by definition, every commit to `master` is a new release. Next, this commit on master should be tagged with this release version for future reference. Finally, the changes made on the release branch show be merged back into `develop`, so new releases can include changes from preevious releases.

- **Without the git-flow extension:**

  1- Merge the release with `master`

  ```shell
  $ git checkout master
  Switched to branch 'master'
  $ git merge --no-ff release/0.1.0
  Merge made by recursive.
  (Summary of changes)
  ```

  2- Tag the new production release

  ```shell
  $ git tag -a 0.1.0
  ```

  3- Merge the new release with `develop` (be aware of possible merge conflicts)

  ```shell
  $ git checkout develop
  Switched to branch 'develop'
  $ git merge --no-ff release/0.1.0
  Merge made by recursive.
  (Summary of changes)
  ```

  4- Remove obsolete release branch

  ```shell
  $ git branch -d release/0.1.0
  Deleted branch release/0.1.0 (was ff452fe).
  ```

- **When using the git-flow extension:**
  ```shell
  $ git flow release finish -F '0.1.0'
  ```
  > Notice how the git-flow extension handles merging with `master` & `develop` and automatically tags the production version

Finally, to push tags to remote:

```
$ git push origin --tags
```

> Note: To sign the tag cryptographically, use the `-s` or `-u <key>` flags.

#### Hotfix Branches

Although similar to release branches in the way they prepare new production releases, these branches are dedicated for sudden or unplanned changes. Their role arises when an undesired behaviour is detected in a live production version. When a critical bug in a production version is to be immediately resolved, a hotfix branch is branched off from the corresponding tag on the `master` branch that marks the production version.

At this stage, team members working on `develop` branch can continue, while in the meantime someone is preparing a quick production fix.

 <img src="https://wac-cdn.atlassian.com/dam/jcr:61ccc620-5249-4338-be66-94d563f2843c/05%20(2).svg?cdnVersion=1429" alt="drawing" width="50%" style=" display: block; margin-left: auto; margin-right: auto;margin-bottom: 5%;margin-top: 5%;"/>

| May branch off from: | Should merge back into: | Naming convention: |
| -------------------- | ----------------------- | ------------------ |
| `master`             | `develop` and `master`  | `hotfix/*`         |

##### Opening a hotfix branch

Assuming a critical bug appears on production version `0.1.0` which is currently live, but changes in `develop` are still unstable, a hotfix brach is pulled from master. This is very similar to opening a release branch:

- **Without the git-flow extension:**

  ```shell
  $ git checkout -b hotfix/0.1.1 master
  Switched to a new branch "hotfix/0.1.1"
  ```

- **When using the git-flow extension:**
  ```shel
  $ git flow hotfix start 0.1.1
  Switched to a new branch 'hotfix/0.1.1'
  ```
  > Notice how the git-flow extension does not require adding the prefix `hotfix/` as it has been preconfigured

After opening a new hotfix branch named `hotfix/0.1.1`, it is likely to make these changes:

```shell
$ ./bump-version.sh 0.1.1
Files modified successfully, version bumped to 0.1.1.
$ git commit -a -m "Bumped version number to 0.1.1"
[hotfix/0.1.1 41e61bb] Bumped version number 0.1.1
1 files changed, 1 insertions(+), 1 deletions(-)
```

Again, `bump-version.sh` is just a fictional shell script that modifies some files to reflect the new version. Of course, these changes might be done manually (changing version number in `package.json` for example). Then, the bumped version number is committed.

##### Finishing a hotfix branch

After applying fixes, this branch needs to be merged back into `master` and `develop`, in order to make sure that the bugfix is included in the next release as well. This is very similar to how release branches are finished.

- **Without the git-flow extension:**

  1- Merge the hotfix with `master`

  ```shell
  $ git checkout master
  Switched to branch 'master'
  $ git merge --no-ff hotfix/0.1.1
  Merge made by recursive.
  (Summary of changes)
  ```

  2- Tag the new production release

  ```shell
  $ git tag -a 0.1.1
  ```

  3- Merge the new hotfix with `develop`

  ```shell
  $ git checkout develop
  Switched to branch 'develop'
  $ git merge --no-ff hotfix/0.1.1
  Merge made by recursive.
  (Summary of changes)
  ```

  _Exception_: If a release branch already exixts, the hotfix changes are merged to that release branch, instead of `develop`. Hotfix changes will eventially be merged automatically with `develop` when release branch is finished and merged again with `develop`. (On a side note, If work in develop immediately requires this bugfix and cannot wait for the release branch to be finished, it is safe to merge the bugfix into develop before waiting for release.)

  4- Remove obsolete hotfix branch

  ```shell
  $ git branch -d hotfix/0.1.1
  Deleted branch hotfix/0.1.1 (was abbe5d6).
  ```

- **When using the git-flow extension:**
  ```shell
  git flow hotfix finish -F '0.1.1'
  ```
  > Notice how the git-flow extension handles merging with `master` & `develop` and automatically tags the production version

Finally, to push tags to remote:

```shell
$ git push origin --tags
```

> Note: To sign the tag cryptographically, use the `-s` or `-u <key>` flags.

### About fast forwards

The `--no-ff` flag means that the merge always makes a new commit object, even if the merge could be performed with a `fast-forward`. This step avoids losing data about the existence of a feature branch and groups together all commits under their original feature.

When using `fast-forward`, it cannot be observed from the git history which set of commit objects are related to a feature; it is requried to manually read all the log messages. Moreover, reverting a whole feature (set of commits) is avery hard, unlike in the other case where `--no-ff` is used.

### Git-flow Summary

Here is a brief summary on the whole flow:

1- A `develop` branch is opened from `master`
2- A release branch is opened from `develop`
3- Feature branches are opened from `develop`
4- When a feature is done it is merged into the `develop` branch
5- When the release branch is done it is merged into `develop` and `master`
6- If an issue in `master` is detected a hotfix branch is opened from `master`
7- Once the hotfix is complete it is merged to both `develop` and `master`

### Sources

- [Vincent Driessen's original article](https://nvie.com/posts/a-successful-git-branching-model/)
- [Gitflow workflow on BitBucket](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow)

Jihad Al KHURFAN
