# GitHub notifications manager

This repo contains a script to help you manage the notifications you get from Exercism's GitHub repos.
It does this by allowing you to bulk update your repo subscription status for those GitHub repos.

This is especially useful when joining a GitHub team that will automatically (and undesirably) subscribe you to multiple repositories.
By running the script before and after joining the team, you can see which repos have changed status and then revert them to their existing state.

## Usage

Explanations of the various functionality is detailed below, but an expected workflow looks like:

### 1. Get the repo!

```bash
gh repo clone exercism/gh-notifications-manager
cd gh-notifications-manager
```

### 2. Export your current notification preferences

```bash
bin/manage export
```

This creates a `subscriptions.json`.
Back this file up, as you'll want this copy of it later!

### 3. Tell admins you're ready

When you've done this, tell us and we'll add you to whatever teams you're being added to.
This will likely subscribe you to lots of new repos.

### 4. Reset your notifications

An early version of this script had a bug where archived repos were also exported to the `subscriptions.json` file.
If your original `subscriptions.json` file contains more Exercism repos than you expected then you can simply run:

```bash
bin/manage update
```

This will sync GitHub back to your original file, effectively undoing all the new notification subscriptions you just had.

**Alternatively,** if your old file only had repositiories you were subscribed to in it, you can now re-export, and the file will be updated with all the new repos.
You can then run the `unsubscribe_new` command to update their statuses in the file, and then `update` to sync GitHub:

```bash
bin/manage export          # Updates subscriptions.json with new subscriptions
bin/manage unsubscribe_new # Changes the state of new repos to be UNSUBSCRIBED
bin/manage update          # Updates the changes on GitHub
```

## Prerequisites

The manage script requires these tools be installed:

- [GitHub CLI](https://cli.github.com/): used to download and update subscriptions (v2.48.0 or greater)
- [jq](https://jqlang.github.io/jq/): used to process the subscriptions which are stored in a JSON file

## Configuring GitHub CLI

As the manage script uses the [GitHub CLI](https://cli.github.com/) to access GitHub, the GitHub CLI's authenticated user must be a member of the Exercism GitHub organization.
If it is not, the script will output an error message.

There are two possible fixes:

1. You've previously authenticated the GitHub account you're using for Exercism, in which case you run:

```shell
gh auth switch -u <github_username_for_exercism>
```

2. You've not yet authenticated the GitHub account you're using for Exercism, in which case you run:

```shell
gh auth login
```

## Export subscriptions

To export the subscription status of Exercism repos you're subscribed to, run:

```shell
bin/manage export
```

This will create a `subscriptions.json` file containing the (unarchived) Exercism repos and your subscription status.
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
- `new`: allows you to quickly find new entries

The `new` value is determined by looking at the current `subscriptions.json` file (i.e. the result of the last `export` action) _before_ the `export` action overwrites its contents.
The `new` value is set to `true` when either:

- The `subscriptions.json` file did not exist
- The `subscriptions.json` file contained an entry for the repo

In all other cases, the `new` value is set to `false`.
The very first export will thus set the `new` field to `true` for all repos.

### Review your subscriptions

Because the subscriptions file can be quite lengthy, you can scroll through them by status by running:

```shell
bin/manage review
```

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

## Updating to latest version

Please make sure to always run the latest version of the `bin/manage` script by running `git pull` first.
