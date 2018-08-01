# GitHub Webhooks with Google Cloud Functions

A [Google Cloud Function](https://cloud.google.com/functions/) that handles [GitHub Webhook](https://developer.github.com/webhooks/) events.

Implemented for the [Node 8 runtime](https://cloud.google.com/functions/docs/concepts/nodejs-8-runtime) with no additional runtime dependencies.

## GitHub to Trello

The default handler, `githubToTrello`, posts comments and attachments to Trello when there is a GitHub push or pull request.

For example, Alice's team uses a Trello board, where a card is created per feature. When Alice starts working on a card, she follows a git branch naming convention that allows her to just copy-and-paste the latter part of the card's url:

> https://trello.com/c/nqPiDKmw/9-grand-canyon-national-park

becomes a new branch with:
```bash
git checkout -b nqPiDKmw/9-grand-canyon-national-park
```

Now whenever Alice pushes to GitHub, a webhook fires to our Cloud Function, which parses the branch name for the card id (here, `nqPiDKmw`), and automatically adds a comment to the card that links back to the head commit of the push.

Similarly, when Alice is ready to open a pull request, the webhook fires an event, and our Cloud Function adds a link to the PR as an attachment to the card.

## GitHub to ???
Not a Trello user? Just add a different handler under `src/handler/` and point to it from `src/index.js`.

Make a pull request!

## Deploy

1. Trello
  * Get your Developer API Key & generate a Token: https://trello.com/app-key
  * Set environment variables for the API key and token:
  ```bash
  export TRELLO_API_KEY=<your key here>
  export TRELLO_TOKEN=<your token here>
  ```
2. Google Cloud Functions
  * If you've never used gcloud or deployed a Cloud Function before, run through the [Quickstart](https://cloud.google.com/functions/docs/quickstart#functions-update-install-gcloud-node8) to make sure you have a GCP project with the Cloud Functions API enabled before proceeding.
  * Generate a secret token to [validate GitHub requests](https://developer.github.com/webhooks/securing/), e.g.:
  ```bash
  export GITHUB_SECRET=`node -e "require('crypto').randomBytes(20, (e, buf) => console.log(buf.toString('hex')));"`
  ```
  * Fork/clone this repo
  * Within the repo, deploy this cloud function with:
  ```
  gcloud beta functions deploy githubWebhookHandler \
  --trigger-http --runtime nodejs8 --memory 128MB \
  --set-env-vars GITHUB_SECRET=$GITHUB_SECRET,TRELLO_API_KEY=$TRELLO_API_KEY,TRELLO_TOKEN=$TRELLO_TOKEN \
  --project `gcloud config list --format 'value(core.project)'`
  ```
  * Note the URL of your Cloud Function (also obtainable with: `gcloud functions describe githubToTrello --format 'value(httpsTrigger.url)'`)
3. GitHub
  * Add webhook (e.g. `https://github.com/<user-or-org/<repo>/settings/hooks/new`)
    * *Payload URL*: the URL of your Cloud Function, e.g. `
https://<GCP_REGION>-<PROJECT_ID>.cloudfunctions.net/githubToTrello`
    * *Content type*: `application/json`
    * *Secret*: the token (`$GITHUB_SECRET`) you generated in step 2
    * *Let me select individual events*: `Pushes` and `Pull Requests`


## Testing

### Prerequisites
* Node 8
* npm 5.2.0 or later (yarn should be fine as well)

### Unit tests
```bash
npm install
npm test
```

### Ad-hoc tests

Start up the emulator:

```bash
npx functions start --tail true --verbose true
```

In another terminal/session/screen:

```
curl -H "X-Hub-Signature: sha1=3dacb5b880e25ff9d874df3daa457f3193cac51d" \
-H "X-GitHub-Event: pull_request" -H "X-GitHub-Delivery: 123" \
-H "Content-Type: application/json" -d '{"action":"opened","pull_request":{"html_url":"https://github.com/github/linguist/pull/11","head":{"ref":"nqPiDKmw/9-grand-canyon-national-park"}}}' \
http://localhost:8010/<projectId>/<region>/githubToTrello
```

## Contributing
Contributions welcome! Please see [CONTRIBUTING.md](docs/CONTRIBUTING.md).

## License
This project is released under the ISC license, which can be found in [LICENSE](LICENSE).

## References
* Google Cloud Functions
  * [HTTP Triggers](https://cloud.google.com/functions/docs/calling/http)
  * [Using Environment Variables](https://cloud.google.com/functions/docs/env-var)
  * [Node.js Emulator](https://cloud.google.com/functions/docs/emulator)
  * [Testing and CI/CD](https://cloud.google.com/functions/docs/bestpractices/testing)
* GitHub
  * [Webhooks](https://developer.github.com/webhooks/)
  * [Event Types & Payloads](https://developer.github.com/v3/activity/events/types/)
  * [Securing your webhooks](https://developer.github.com/webhooks/securing/)
* Trello
  * [Developer API Keys](https://trello.com/app-key)
  * [REST API: Add a new comment to a card](https://trello.readme.io/reference#cardsidactionscomments)
