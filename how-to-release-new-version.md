# Release process
## Workflow diagram
- The third and fourth steps are optional and apply only if a runtime error is identified during RC1 validation, requiring a hotfix to be applied to the release branch.
- Opening a PR to `main` can be done either immediately or after testing—both approaches are acceptable.

```mermaid
stateDiagram-v2
    direction LR
    s1 --> s2
    s2 --> s3
    s3 --> s4
    s4 --> s5
    s1: create release-branch from develop
    state s1 {
        s11: push to release
        state s11 {
            s110-->s111
            s111-->s112
            s112-->s113
            s113-->s114
            s110: run tests
            s111: build Docker
            s112: validate RC tag
            s113: push RC tag
            s114: publish Docker to GHCR
        }
    }
    s2: open PR to main
    state s2 {
        s21: PR to main

        state s21 {
            s210: validate tag
            s211: run tests
            s212: build Docker
            s211-->s212
        }
    }
    s3: hotfix PR to release-branch (optional)
    state s3 {
        direction LR
        s31-->s32
        s31: push to hotfix
        state s31 {
            s311: run tests
        }
        s32: PR to release
        state s32 {
            s321: run tests
            s322: build Docker
            s321-->s322
        }
    }
    s4: merge hotfix PR to release
    state s4 {
        direction LR
        s41-->s42
        s41: push to release
        s42: PR to main
        state s41 {
            s410-->s411
            s411-->s412
            s412-->s413
            s413-->s414
            s410: run tests
            s411: build Docker
            s412: validate RC tag
            s413: push RC tag
            s414: publish Docker to GHCR
        }
        state s42 {
            s420: validate tag
            s421: run tests
            s422: build Docker
            s421-->s422
        }
    }
    s5: merge release PR to main
    state s5 {
        s51: push to main
        state s51 {
            s510-->s511
            s511-->s512
            s512-->s513
            s513-->s514
            s514-->s515
            s515-->s516
            s510: run tests
            s511: build Docker
            s512: validate stable tag
            s513: push stable tag
            s514: publish Docker to GHCR
            s515: create PR from main to develop
            s516: create GitHub release
        }
    }
```

## Testing steps

#### 1. Create a release branch and push it to github
```
git checkout -b release/1.6.6 develop
git push origin release/1.6.6
```
> [!info] triggers push to release action:
> - runs tests
> - builds docker
> - validates version
> - creates RC tag
> - publishes RC1 package with RC1 and `UAT` tags
> - check https://github.com/ank1m/batchee/pkgs/container/batchee

#### 2. Create PR to `main` branch: https://github.com/ank1m/batchee/compare/main...release/1.6.6
> [!info] triggers PR to main action:
> - runs tests
> - builds docker
> - validates version

#### 3. If bug is found during RC1 validation, open bugfix branch and create PR to `release` branch

##### 3.1 Create `bugfix` branch, push the fix
```
git checkout -b bugfix/test release/1.6.6
git commit --allow-empty -m "fixing some imaginary error"
git push origin bugfix/test
```
> [!info] triggers push to bugfix action:
> - runs tests

##### 3.2 Create PR to `release` branch: https://github.com/ank1m/batchee/compare/release/1.6.6...bugfix/test
> [!info] triggers PR to bugfix action:
> - runs tests
> - builds docker

#### 4 Merge PR
> [!info] triggers push to release action:
> - runs tests
> - builds docker
> - validates version
> - creates next RC tag
> - publishes package with next RC tag and updates `UAT` tag
> - check https://github.com/ank1m/batchee/pkgs/container/batchee

> [!info] also re-triggers PR to main action if PR was submitted, see #2

#### 5 Merge PR from `release` branch to `main`
> [!info] triggers push to main action:
> - runs tests
> - builds docker
> - validates version
> - creates stable tag
> - publishes package with stable tag, `OPS`, and `latest` tags
> - check https://github.com/ank1m/batchee/pkgs/container/batchee
> - creates auto-PR to backmerge `main` to `develop`: https://github.com/ank1m/batchee/pulls
> - creates GitHub release with auto-generated notes: https://github.com/ank1m/batchee/releases

#### 6 Approve and Merge auto-PR to backmerge `main` to `develop`
