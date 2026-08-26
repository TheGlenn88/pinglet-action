# Pinglet Notify

[![GitHub Marketplace](https://img.shields.io/badge/Marketplace-Pinglet%20Notify-blue?logo=github)](https://github.com/marketplace/actions/pinglet-notify)

Send a push notification to a [Pinglet](https://pinglet.dev) topic from any GitHub Actions workflow: deploy results, CI failures, release announcements, straight to your phone. No SDK, no app config in the workflow; one step.

Anyone who should receive the alerts just opens your topic's share link in the [Pinglet app](https://pinglet.dev). No account needed to receive.

## Quick start

1. Mint an API key in your [Pinglet dashboard](https://pinglet.dev) and add it to your repository as a secret named `PINGLET_KEY`.
2. Add a step:

```yaml
- name: Notify deploy
  uses: TheGlenn88/pinglet-action@v1
  with:
    key: ${{ secrets.PINGLET_KEY }}
    topic: acme/deploys
    title: "Deploy done"
    message: "${{ github.repository }} ${{ github.ref_name }} is live"
    level: success
```

The topic is created on the first publish. Subscribe to it in the Pinglet app via the share link from your dashboard (or from the publish response).

## Alert on failure

Add a final step that only fires when the job failed. `priority: urgent` breaks through Do Not Disturb:

```yaml
- name: Alert on failure
  if: failure()
  uses: TheGlenn88/pinglet-action@v1
  with:
    key: ${{ secrets.PINGLET_KEY }}
    topic: acme/ci
    title: "CI failed"
    message: "${{ github.workflow }} failed on ${{ github.ref_name }} (${{ github.sha }})"
    level: error
    priority: urgent
```

## Inputs

| Input | Required | Description |
|---|---|---|
| `key` | yes | Pinglet API key (`pinglet_...`). Use a repository secret. |
| `topic` | yes | Topic address as `namespace/name`, e.g. `acme/deploys`. |
| `message` | yes | Notification body. |
| `title` | no | Notification title. Defaults to the topic name. |
| `level` | no | Display severity: `info`, `success`, `warning`, `error`. |
| `priority` | no | `silent`, `normal` or `urgent`. Urgent breaks through DND. |
| `badges` | no | JSON object of up to 3 short pills for the card, e.g. `{"env":"prod","run":"${{ github.run_number }}"}`. |
| `data` | no | JSON object of metadata for the detail sheet (Pro plans). |
| `dry-run` | no | `true` prints the payload without sending. |

## Outputs

| Output | Description |
|---|---|
| `response` | The publish response JSON: message id, shareable subscription URL, and the delivery tally. |

## Notes

- Requests retry up to 3 times and fail the step (with the API's error body) on a non-2xx response.
- The full message format (levels, priorities, badges, metadata) is documented at [pinglet.dev/docs/messages](https://pinglet.dev/docs/messages/).
- Runs anywhere `bash`, `curl` and `jq` exist, which includes all GitHub-hosted runners.

## License

MIT
