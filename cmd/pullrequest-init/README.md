# pullrequest-init

pullrequest-init fetches pull request data from the given URL and places it in
the provided path.

This binary outputs a generic pull request object into `pr.json`, as well as
provider specific payloads.

Currently supported providers:

*   GitHub

## Generic pull request payload

The payload generated by default aims to abstract the base features of any pull
request provider. Since there is no one true spec for pull requests, not every
feature may be available. The payload is output to `$PATH/pr.json` and looks
like:

```json
{
    "Type": "github",
    "ID": 188184279,
    "Head": {
        "Repo": "https://github.com/wlynch/test.git",
        "Branch": "dev",
        "SHA": "9bcde245572c74329827acdcab88792ebb84d578"
    },
    "Base": {
        "Repo": "https://github.com/wlynch/test.git",
        "Branch": "master",
        "SHA": "1e80c83ed01e187669b836096335d1c8a2c57182"
    },
    "Comments": [
        {
            "Text": "test comment",
            "Author": "wlynch",
            "ID": 494418247,
            "Raw": "/tmp/prtest/github/comments/494418247.json"
        }
    ],
    "Labels": [
        {
            "Text": "my-label"
        }
    ],
    "Statuses": [
        {
            "Code": "success",
            "Description": "Job succeeded.",
            "ID": "pull-tekton-pipeline-go-coverage",
            "URL": "https://tekton-releases.appspot.com/build/tekton-prow/pr-logs/pull/tektoncd_pipeline/895/pull-tekton-pipeline-go-coverage/1141483806818045953/"
        },
    ],
    "Raw": "/tmp/prtest/github/pr.json",
    "RawStatus": "/tmp/pr/github/status.json"
}
```

## GitHub

GitHub pull requests will output these additional files:

*   `$PATH/github/pr.json`: The raw GitHub payload as specified by
    https://developer.github.com/v3/pulls/#get-a-single-pull-request
*   `$PATH/github/status.json`: The raw GitHub combined status payload as
    specified by
    https://developer.github.com/v3/repos/statuses/#get-the-combined-status-for-a-specific-ref
*   `$PATH/github/comments/#.json`: Comments associated to the PR as specified
    by https://developer.github.com/v3/issues/comments/#get-a-single-comment

For now, these files are *read-only*.

The binary will look for GitHub credentials in the `$GITHUBTOKEN` environment
variable. This should generally be specified as a secret with the field name
`githubToken` in the `PullRequestResource` definition.

### Status code conversion

Tekton Status Code | GitHub Status State
------------------ | -------------------
success            | success
neutral            | success
queued             | pending
in_progress        | pending
failure            | failure
unknown            | error
error              | error
timeout            | error
canceled           | error
action_required    | error
