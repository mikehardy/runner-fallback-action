# Github Runner Fallback Action

<p align="center">
  <a href="https://github.com/jimmygchen/runner-fallback-action/actions"><img alt="javscript-action status" src="https://github.com/jimmygchen/runner-fallback-action/workflows/units-test/badge.svg"></a>
</p>

Github action to determine the availability of self-hosted runners, and fallback to a GitHub runner if the primary runners are offline.

This action uses [GitHub API](https://docs.github.com/en/rest/actions/self-hosted-runners?apiVersion=2022-11-28#list-self-hosted-runners-for-a-repository) to check the statuses of self hosted-runners that match specific labels, and outputs the runner label(s), or a fallback runner if the self-hosted runner(s) is unavailable.

This output can then used on the `runs-on` property of subsequent jobs. 

Note: In order to support an array of labels for the `runs-on` field, the output is formatted as a JSON string and needs to be parsed using `fromJson`. See example usage below.



## Usage

### ✏️ Inputs

#### Required

|       Name        |                                          Description                                                       |
| ----------------- | ---------------------------------------------------------------------------------------------------------- |
|  `github-token`   | A token that can access the `list action runners` for the given context (e.g. user repo, org, enterprise). |
|  `primary-runner` | A comma separated list of labels for the _primary_ runner (e.g. 'self-hosted,linux').                      |
| `fallback-runner` | A comma separated list of labels for the _fallback_ runner (e.g. 'self-hosted,linux').                     |


#### Optional
---

There are three ways runners can be allowed to run against a repo: User, Organization, Enterprise. The following options allow you to switch the implementation to use one of the other specified levels. **_Note:_** You can only provide one of the values.

|       Name       |                     Description                                    |
| ---------------- | ------------------------------------------------------------------ |
| `organization`   | The name of the github organization (e.g. `My-Github-Org`)         |
| `enterprise`     | The name of the github enterprise (e.g. `My-Github-Ent`)           |



### Example
```yaml
jobs:
  # Job to 
  determine-runner:
    runs-on: ubuntu-latest
    outputs:
      runner: ${{ steps.set-runner.outputs.use-runner }}
    steps:
      - name: Determine which runner to use
        id: set-runner
        uses: jimmygchen/runner-fallback-action@v1
        with:
          primary-runner: "self-hosted,linux"
          fallback-runner: "ubuntu-latest"
          github-token: ${{ secrets.YOUR_GITHUB_TOKEN }}

  another-job:
    needs: determine-runner
    runs-on: ${{ fromJson(needs.determine-runner.outputs.runner) }}
    steps:
      - name: Do something
        run: echo "Doing something on ${{ needs.determine-runner.outputs.runner }}"
```

Credit: this action is based on the pattern described by @ianpurton on [this feature request thread](https://github.com/orgs/community/discussions/20019#discussioncomment-5414593).
