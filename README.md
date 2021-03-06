# The Rustat Hristov Slack bot

**Rusi:**
> btw, I am thinking about adding a bot to help me set my slack status easier... something to help with the whole WFH situation :confused:

> I want to have pre-defined statuses, like ":hamburger: ...lunch" and ":afk: away from the keyboard", and others

> and that would be easy to trigger with `/status lunch` or `/status afk` or `/status clear` ...

> actually that exists :smile:

> it is not as nice as a one word status msg... but it's usable :slightly_smiling_face:

**Me:**
> So what you want is actually pre-defined status messages that may be activated with a shortcut/keyword?

**Rusi:**
> yes, and maybe some other functions similar to slack's workflows - say I set my status with `/status lunch` - maybe after an hour, a message will popup with a button - "are you back" or something like that

> or I can have `/status meeting 30` to set a 30 minute meeting status message

> the idea is that you should be able to see my slack status, and know what's going on - similar to when we are in the office together - you can just look at my desk and see me working, or having a meeting, or eating, or not being there :smile:

## Installation

<a href="https://slack.com/oauth/v2/authorize?client_id=426270598675.1023720930629&scope=chat:write,commands,incoming-webhook,users.profile:read&user_scope=users.profile:read,users.profile:write,users:read"><img alt="Add to Slack" height="40" width="139" src="https://platform.slack-edge.com/img/add_to_slack.png" srcset="https://platform.slack-edge.com/img/add_to_slack.png 1x, https://platform.slack-edge.com/img/add_to_slack@2x.png 2x"></a>

## Usage

TL;DR
```
/rustat help
/rustat add <key> <message>
/rustat remove <key>
/rustat list
/rusi <key> [<expiry>]
/rusi clear
```

> Welcome to :sparkles:*The Rustat Hristov Slack bot*:sparkles:!
> 
> A "rustat" is a predefined :sparkles:fancy status message:sparkles:. ~It can also mean "R. U. status", that is, "Are you <status>?", like "Are you `lunch`?"~ NOPE.
> 
> `/rustat help` -- Print this help text
> 
> **Manage your rustats**
>
> `/rustat add <key> <message>` -- Save a rustat with the shortcut keyword `<key>`; if `<message>` starts with an emoji, that emoji will be used as status emoji
>
> `/rustat remove <key>` -- Deletes a rustat with the shortcut keyword > `<key>`
>
> `/rustat list` -- List all saved rustats
> 
> **Set your status message**
>
> `/rusi <key> [<expiry>]` -- Activate the rustat with the shortcut keyword `<key>` that will (optionally) automatically be cleared by `<expiry` > (_[in human language](https://github.com/wanasit/chrono)!>_); e.g. `/rusi lunch`, `/rusi sleep until 10pm`, `/rusi meeting for 30 minutes` or `/rusi call until 2am EST`
>
> `/rusi clear` -- Clear your current status message (whether it's a > rustat or not)

## Development

### Setup

Requires:
- NodeJS 12.x
- [Serverless Framework](https://serverless.com/framework/docs/getting-started#install-via-npm)
    ```sh
    npm i -g serverless
    ```
- [ngrok](https://ngrok.com/)
  - On macOS, with Homebrew: `brew install ngrok`
- a Slack app -- follow the initial setup here: https://slack.dev/bolt/tutorial/getting-started
  - Additional steps
    - Create slash commands for `/rustat` and `/rusi`
    - Set OAuth redirect URL

Create `.env` then set the environment variables with values taken from your Slack app
- supports `.env.<sls env>` where `<sls env>` is the value used in the `serverless` command invoked
- e.g. `.env.dev` for `sls deploy --env dev`
```sh
cp example.env .env
```

Install app dependencies
```sh
npm i
sls dynamodb install
```

### Running locally

Run ngrok
```sh
ngrok http 3000
```

Copy the ngrok URL and paste it into:
- `.env` for `SLACK_REDIRECT_URL`
- the Slack app's configured slash commands and OAuth redirect URL

Start serverless-offline
- this watches for changes in the TypeScript source code and automatically rebuilds
- however, it does not watch for .env changes -- you need to restart in that case
- also, somehow, serverless-offline buries all console.log output ¯\_(ツ)_/¯
- the alternative is to run Slack Bolt locally as well, but that also is another PITA because it doesn't watch for local changes
```sh
npm run start:offline
```

### DynamoDB data model

#### `installations`

Slack workspace installations

| Entity         | Hash                           | Range           |
|----------------|--------------------------------|-----------------|
| `installation` | `PK#<enterprise_id>#<team_id>` | `#SK#<user_id>` |

#### `rustats`

| Entity                    | Hash                  | Range                   | Other attributes |
|---------------------------|-----------------------|-------------------------|------------------|
| `rustat`                  | `rustat#<username>`   | `#key#<username>#<key>` | `key`, `message` |

Ref: https://aws.amazon.com/getting-started/projects/design-a-database-for-a-mobile-app-with-dynamodb/4/
