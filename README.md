# hubot-slack-github-issues - WORK IN PROGRESS!

[![Build Status](https://travis-ci.org/18F/hubot-slack-github-issues.svg?branch=master)](https://travis-ci.org/18F/hubot-slack-github-issues)

[Hubot](https://hubot.github.com/) plugin that creates
[GitHub](https://github.com/) issues from [Slack](https://slack.com/) messages
that receive a specific emoji reaction.

This plugin is for use by organizations that use Slack to communicate and who
use GitHub to track issues. The goal is to enable team members to document or
to act upon important parts of conversations more easily.

## How it works

It works by registering [receive middleware](https://hubot.github.com/docs/scripting/#receive-middleware)
that listens for [`reaction_added` events](https://api.slack.com/events/reaction_added).
When team members add an emoji reaction to a message, the resulting event is
matched against a set of [configuration rules](#configuration).
If a match is found, the plugin will [retrieve the list of
reactions](https://api.slack.com/methods/reactions.get) for the message.

Provided that the message has not already been processed, the plugin will
[create a GitHub issue](https://developer.github.com/v3/issues/#create-an-issue) for the
message based on the the rule. The issue will contain a link to the message.
At this point, the plugin will [add a reaction to the
message](https://api.slack.com/methods/reactions.add) with an emoji indicating
success, and the issue URL is posted to the channel in which the message
appeared.

## Installation

1. In your Hubot repository, run:
  ```bash
  $ npm install '18f/hubot-slack#3.4.2-handle-reaction-added' \
      hubot-slack-github-issues --save
  ```

1. Include the plugin in your `external-scripts.json`.

  ```json
  [
    "hubot-slack-github-issues"
  ]
  ```

### Note regarding near-term dependencies

This plugin depends upon
[18F/hubot-slack#3.4.2-handle-reaction-added](https://github.com/18F/hubot-slack/tree/3.4.2-handle-reaction-added),
which in turn depends upon
[18F/node-slack-client#1.5.0-handle-reaction-added](https://github.com/18F/node-slack-client/tree/1.5.0-handle-reaction-added).
These packages are custom forks of the
[hubot-slack](https://www.npmjs.com/package/hubot-slack) and 
[slack-client](https://www.npmjs.com/package/slack-client) packages that have
had support for the [`reaction_added`
event](https://api.slack.com/events/reaction_added) patched in.

When those official packages have been updated to include `reaction_added`
support, this plugin will be updated to use those packages instead of the
custom versions.

## Configuration

You'll need to create a JSON file conforming to the following schema:

* *githubUser*: GitHub organization or username owning all repositories
* *githubTimeout*: GitHub API timeout limit in milliseconds
* *slackTimeout*: Slack API timeout limit in milliseconds
* *successReaction* emoji used to indicate an issue was successfully filed
* *rules*: defines each condition that will result in a new GitHub issue
  * *reactionName* name of the reaction emoji triggering the rule
  * *githubRepository*: GitHub repository belonging to *githubUser* to which
    to post issues
  * *channelNames (optional)*: name of the Slack channels triggering the rule;
    leave undefined to match messages in _any_ Slack channel

For example:

```json
{
  "githubUser": "18F",
  "githubTimeout": 5000,
  "rules": [
    {
      "reactionName": "evergreen_tree",
      "githubRepository": "handbook"
    }
  ]
}
```

The following environment variables must also be set:

* `HUBOT_GITHUB_TOKEN`: GitHub API token
* `HUBOT_SLACK_TOKEN`: Slack API token (also needed by
  [`hubot-slack`](https://www.npmjs.com/package/hubot-slack))

The following environment variables are optional:
* `HUBOT_SLACK_GITHUB_ISSUES_CONFIG_PATH`: the path to the configuration file;
  defaults to `config/slack-github-issues.json`

## Developing

Ensure that [Node.js](https://nodejs.org/) is installed on your system. This
plugin requires version 4.2 or greater or version 5 or greater. You
may wish to use a version manager such as
[nvm](https://github.com/creationix/nvm) to manage different Node.js versions.

After cloning this repository, do the following to ensure your installation is
in a good state:

```sh
$ cd hubot-slack-github-issues
$ npm install
$ npm run-script lint
$ npm test
```

After making changes, run `npm run-script lint` and `npm test` frequently. Add
new tests in [the `test` directory](./test/) for any new functionality, or to
reproduce any bugs you intend to fix.

If you'd like to contribute to this repository, please follow our
[CONTRIBUTING guidelines](./CONTRIBUTING.md).

## Public domain

This project is in the worldwide [public domain](LICENSE.md). As stated in
[CONTRIBUTING](CONTRIBUTING.md):

> This project is in the public domain within the United States, and copyright
> and related rights in the work worldwide are waived through the
> [CC0 1.0 Universal public domain dedication](https://creativecommons.org/publicdomain/zero/1.0/).
>
> All contributions to this project will be released under the CC0 dedication.
> By submitting a pull request, you are agreeing to comply with this waiver of
> copyright interest.
