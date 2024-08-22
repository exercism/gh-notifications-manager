# GitHub notifications manager

This repo contains a script to help you manage the notifications you gets from Exercism's GitHub repos.
It does this by allowing you to bulk update your repo subscription status for those GitHub repos.

This is specifically useful when joining a GitHub team that will automatically and undesirably subscribe you to multiple repositories.
By running the script before and after joining the team, you can see which repos have changed status and then revert them to their existing state.

## Usage

Explanations of the various functionality is detailed below, but an expected workflow looks like:
```bash
bin/manage export          # Creates a record of your subscriptions

# Manually: Backup file
# Manually: Join relevant teams

bin/manage export          # Updates the record with new subscriptions
bin/manage unsubscribe_new # Changes the state in the file to UNSUBSCRIBED for all new repos
bin/manage update          # Updates the changes on GitHub
```


## Prerequisites

The manage script requires these tools be installed:

- [GitHub CLI](https://cli.github.com/): used to download and update subscriptions
- [jq](https://jqlang.github.io/jq/): used to process the subscriptions which are stored in a JSON file

## Export subscriptions

To export the subscription status of Exercism repos you're subscribed to, run:

```shell
bin/manage export
```

This will create a `subscriptions.json` file containing the Exercism repos and their subscription status.
**It is recommended that you back up this file before continuining**.

### subscriptions.json file

The downloaded `subscriptions.json` file will look like this:

```json
[
  {
    "repo": "exercism/csharp",
    "status": "UNSUBSCRIBED",
    "new": false
  },
  {
    "repo": "exercism/csharp-test-runner",
    "status": "SUBSCRIBED",
    "new": false
  }
]
```

It is an array of objects, where each object has the following fields:

- `repo`: the full repo name (will always start with `exercism/`)
- `status`: the subscription status of the repo, one of:
  - `"SUBSCRIBED"`: notified of all conversations
  - `"UNSUBSCRIBED"`: only notified when participating or @mentioned
  - `"IGNORE"`: never notified (ignored)
- `new`: indicates if this repo wasn't in the previous `subscriptions.json` file.
  This allows you to quickly find new entries.

### Ignored repos

As Exercism has a _lot_ of repos, we opted _not_ to include repos with subscription status set to `"IGNORE"` in the `subscriptions.json` file.
If you'd like change the subscription status of an ignored repo, feel free to manually add them to the `subscriptions.json` file.

You can then bulk update the subscription status with the [`update` command](#update-subscriptions).

## Update subscriptions

To update the subscriptions in GitHub based on the contents of the `subscriptions.json` file, run:

```shell
bin/manage update
```

This loops over the subscriptions in the `subscriptions.json` file and updates the GitHub repo's subscription status.

Note: if the repo subscription status in the `subscriptions.json` file matches the GitHub repo's subscription status, we skip updating the repository.

## Unsubscribe from new repos

When a new repository is created, the initial subscription status is `"SUBSCRIBED"`.
You might want to change these to `"UNSUBSCRIBED"`, which you can do by running:

```shell
bin/manage unsubscribe_new
```

This will update the `subscriptions.json` file and changes the `status` field from `"SUBSCRIBED"` to `"UNSUBSCRIBED"` for all repos where the `new` field is `true`.

Note: this will _not_ update the GitHub repo's subscription status.
For that, you'll need to run the [update command](#update-subscriptions).
