Matrix ChatGPT Bot
==================

Note: This is unmaintained. You should use [Baibot](https://github.com/etkecc/baibot) instead of this project.

Talk to ChatGPT via any Matrix client!

![Screenshot of Element iOS app showing conversation with bot](img/matrix-chatgpt.png)

A Matrix bot that uses [waylaidwanderer/node-chatgpt-api](https://github.com/waylaidwanderer/node-chatgpt-api) to access the official ChatGPT API.

## Warning for users upgrading from version 2.x
OpenAI released the [official API for ChatGPT](https://openai.com/blog/introducing-chatgpt-and-whisper-apis). Thus, we no longer have to use any older models or any models which kept on being turned off by OpenAI. This means the bot is now way more stable and way faster. However, please note: The usage of the API is **no longer free**. If you use this bot, your OpenAI account **will be charged**! You might want to limit your budget in your account using the [OpenAI website](https://platform.openai.com/account/billing).
You need to remove the `CHATGPT_MODEL` variable from your environment, if you changed the value.

# Usage
1. Create a room
2. Add the bot
3. Start chatting.

# Features
- Shows typing indicator as ChatGPT is thinking!
- Supports encryption
- Stores context for ChatGPT conversations

# Configure

Create a copy of the example `.env` file

```
cp .env.example .env
```

Adjust all required settings in the `.env` file before running. Optional settings can also be adjusted later.

## Prerequsistes

### Matrix
- You need a Matrix account on [Matrix.org](https://matrix.org) (or any other server) for the bot user. 
- By default, anyone that knows the name of your bot can invite it to rooms or chat with it.
- Restrict access with `MATRIX_BLACKLIST` or `MATRIX_WHITELIST`
- Restrict access with `MATRIX_ROOM_BLACKLIST` or `MATRIX_ROOM_WHITELIST`
- Restrict access with `MATRIX_POWER_LEVEL` (defaults to 0)
- When using a self-hosted setup, you could wildcard all your users with `MATRIX_WHITELIST=:yourhomeserver.example`.

### OpenAI / ChatGPT
- You need to have an account at [openai.com](https://openai.com/). Please note that the usage of the ChatGPT-API is not free.
- Create a [API Key](https://platform.openai.com/account/api-keys). Then, set `OPENAI_API_KEY` in your `.env` file
- You can change the chat-model by setting the `CHATGPT_API_MODEL` in your `.env` file. ChatGPT is the `gpt-3.5-turbo`-model which is the default. Please note that depending on the model your OpenAI account will be charged.
- You can change the API-URL to use another base than the official OpenAI-endpoint. To do so, change the `CHATGPT_REVERSE_PROXY` in the `.env` file.

## Setup

- Set `MATRIX_BOT_USERNAME`
- Set `MATRIX_BOT_PASSWORD` (you can remove this later if you want)
- Run the app using one of the methods below.
- Copy `MATRIX_ACCESS_TOKEN` from the output.
- Set `MATRIX_ACCESS_TOKEN`, you can now remove `MATRIX_BOT_PASSWORD`.

*Note*: Doing any of the following can cause issues with encryption later on:

- Using this token with any other client.
- Using an access token extracted via Element.
- Deleting the storage folder.
- Switching between environments (e.g. Docker or no Docker)

# Run

There are multiple ways to run this bot. The easiest way is to run it within docker.

## with Docker

This is the recommended way to run this project. It will use the latest stable release.

```
docker run -it -v storage:/storage --env-file=./.env --name matrix-chatgpt-bot ghcr.io/matrixgpt/matrix-chatgpt-bot:latest
```

or to build locally from the latest unstable release (only do this if you have a good reason):

```
docker build . -t matrix-chatgpt-bot
docker run -it -v storage:/storage --env-file=./.env --name matrix-chatgpt-bot matrix-chatgpt-bot
```

Note: Without -it flags in the command above you won't be able to stop the container using Ctrl-C

Note: In order to see the output of your console you need to run `docker logs matrix-chatgpt-bot`

## with Docker Compose

If you prefer you can use a docker-compose file. Copy the content below and save it in a file named `docker-compose.yml`. Either with a self-build image (run `docker build . -t matrix-chatgpt-bot` from your local git repo location) or with the latest stable pre-built release from this repo (the **recommended** way).

The script will look for the `.env` file in the same folder as the `docker-compose.yml`. The key storage folder `storage` will be created in the same folder as well. Adjust the locations to your needs.

```
  version: '3.7'
  services:
    matrix-chatgpt-bot:
      container_name: matrix-chatgpt-bot 
      image: ghcr.io/matrixgpt/matrix-chatgpt-bot:latest ## change to "matrix-chatgpt-bot" if you want to use your self-build image 
      volumes:
        - ./storage:/storage
      env_file: 
        - ./.env
````

## without Docker

**Important**: It is strongly recommended to run this package under Docker to not need to install various dependencies manually.
Nevertheless, you can also run it by using the package manager yarn (get it via `apt install -y yarn`). You might also need to have a newer version of Node.js and other missing packages. 

- `yarn`
- `yarn build`
- `yarn start`

## in Development

You only need to do this if you want to contribute code to this package.

- Run `yarn`
- Run `yarn build`


# Good to know

- By default "storage"-folder contains all your encryption keys. If you delete it, you will loose access to all your encrypted messages.
- You can use a [Keyv](https://github.com/jaredwray/keyv) storage backend for persistence if you prefer (advanced).
- The bot uses threads by default, to keep the context you should reply to this thread or the bot will think its a new conversation. "Threads" were previously experimental, you may need to activate them in your client's settings (e.g. in Element in the "lab"-section).
- There is support to set the context to work at either the:
  - room level
  - thread level
  - both (threads fork the conversation from the main room)

# FAQ

## I get "[Error: decryption failed because the room key is missing]" 
Encryption works great with this package but can sometimes be a bit sensitive. Following steps can help to solve the "encryption" error

- Don't use a `MATRIX_ACCESS_TOKEN` extracted via Element-App, use the generated token from the bot based on your `MATRIX_BOT_USERNAME` & `MATRIX_BOT_PASSWORD` set in the `env`file. It will be visible in the console at start up if the `MATRIX_ACCESS_TOKEN` is not already set:
1) Remove the `MATRIX_ACCESS_TOKEN` from the `env` file and make sure `MATRIX_BOT_USERNAME` & `MATRIX_BOT_PASSWORD` are set
2) Re-run the bot
3) Copy the token from console output to your `env` file 
4) Restart the bot again.

- If all fails, you can always reset your key storage. It's important to exercise all of the following steps, because any remaining data could lead to the next encryption error. Once everything is working, make sure to not touch the "storage" folder anymore:
1) Stop the bot
2) Delete the "storage" folder
3) Delete all user data for the matrix bot account (e.g. use Synapse-Admin) or create a fresh bot user account (you can then skip step 4)
4) Log into your bot account (e.g. via Element) and log out of all sessions
5) Verify the correctness of your `env` file and then run the bot setup again (e.g. via `docker-compose up` if you use docker-compose).

## I want to chat with the bot without dealing with encryption problems
- Set `MATRIX_ENCRYPTION=false` in your env-file and restart the bot. If it previously was running with encryption switched on, you need to create a new room with the bot as encryption can't be switched off once it was activated.

## I get "{ errcode: 'M_NOT_FOUND', error: 'Event not found.' }"
- So far, its not known to cause issues, you can safely ignore it.

## How do I know that the bot is running succesfully?
Once the bot has started succesfully, it will output the following information to your console. 
-  `[INFO] [index] Starting bot...`
-  `[INFO] [MatrixClientLite] End-to-end encryption enabled`  ## this depends on your setup
-  `[INFO] [index] Bot started!`

## I use Docker but I don't see any console output
You most likely need to view the logs by running `docker logs matrix-chatgpt-bot`

## How to set the temperature
Set the temperature by using CHATGPT_TEMPERATURE in your .env file. The default is 0.8.

Here are some guidelines for setting the temperature:

| Temperature Values | Appropriate Tasks | Examples |
| --- | --- | --- |
| Below 0.5 (low) | Tasks requiring a single correct answer or predictable output | Programming |
| 0.5-0.9 (medium) | Tasks needing somewhat varied and creative content grounded in reality | E-mail response |
| Above 0.9 (high) | Tasks requiring more creative and unpredictable output | Story writing |

## The response I receive is excessively long and gets truncated. Unlike in ChatGPT, my matrix does not have a "Continue generating" button.
You can simply write `continue` to prompt the bot to continue from its previous answer.

# Reporting issues

Please report issues via Github. The chat room is for discussion.

Please use the search on Github and Matrix before asking for support. 

# Discussion

Join [#matrix-chatgpt-bot:matrix.org](https://matrix.to/#/#matrix-chatgpt-bot:matrix.org) with any Matrix chat client or on the web!

If you've never set up a Matrix client before you can follow the prompts to get started.


# License
GNU AGPLv3. See LICENSE
