# ASAP - Autonomous Shell Assistant Program

Fix bugs and understand issues in your codebase. Powered by GPT-4.

Run something like `asap do "fix my type errors"` and the agent will:

- attempt to understand the issue
- read the relevant code
- figure out the fix
- apply a patch to your files
- checkout a fresh git branch
- and, commit the changes.

You'll be able to step in with assistance or confirmations at various stages to help guide the process.

## Prerequisites

To run `asap`, you'll need:

- a Humanloop account ([sign up here](https://app.humanloop.com/signup))
- an OpenAI API key with access to GPT4-32k (https://platform.openai.com/account/api-keys)

## Setup

To start using `asap`, do the following:

1. Clone this repo
1. `cd` to the project directory
1. Run `npm install` to install dependencies
1. If you want to be able to invoke `asap` from any other directory on your machine (suggested) run `npm install -g .`
1. Run `npm build` (required as the project is written in TypeScript and needs to be compiled)
1. By this point, you should be able to directly invoke `asap help` from the command line.
   - If the `asap` executable is not found, you may need to add your global `npm` binary directory to your `PATH`. To find this directory, run `echo $(npm config get prefix)/bin`
   - You can also run it directly with `npx ts-node src/index.ts help`
1. Next, configure your installation by running `asap init`. You'll be asked to provide your API keys for both Humanloop and OpenAI.
   - The program will then install a project called `asap` in your Humanloop account and deploy the relevant model config to that project.

## Developing

If you want to work on developing `asap`, run `npm run dev` in a new terminal. This will run the TypeScript compiler in watch mode so that next time you run `asap do "..."` you'll get the latest behaviour. To work on improving your prompts, we recommend using the Humanloop editor and [environment deployments](https://docs.humanloop.com/docs/deploy-to-an-environment) to control which model configuration you target.

## How it works

### Tool Calling

`asap` works by including a "tool call" in the prompt which tells GPT-4 that it can request running any command in your terminal to help debug the issues and answer the questions you ask. Tool calls correspond to [function calling](https://openai.com/blog/function-calling-and-other-api-updates#function-calling) for OpenAI models. The basic concept is that you can define certain functions that the model is allowed to call by providing a description and function signature; if the model calls your function, you perform whatever processing is necessary to determine the output and provide this back to the model in the next chat message.

As an example, if you ask `asap` to fix your TypeScript type errors, the model is likely to first ask to run `tsc` (the TypeScript compiler). The `asap` program then actually executes the desired `tsc` command on your machine, and then passes the output from stdout back to the model. This process continues in an infinite while-loop until the model is satisfied that it has resolved your issue.

In addition to the CLI tool call, there are a couple more tool calls exposed to the model. The three tools which are:

- `cli_call`: Run a CLI command with the specified arguments in the project directory.
- `done`: Call this when satisfied that the problem is resolved.
- `patch`: Call this to apply a modification to a file in the project.

You can inspect the exact definition of these tools in the Humanloop Editor (after you have setup with `asap init`).

## Examples

Try running `asap` on the examples in the `/examples` directory in the project repo. For instance:

1. Navigate to `examples/1`
1. Run `asap do "fix the typescript errors"`
1. You should find that `asap` attempts to run `tsc --noEmit`, or something similiar, to determine the errors it needs to fix, and then reads the output of the command before proceeding to explore the relevant files and apply fixes.

## Usage

```
Usage: asap [options] [command]

Autonomous Shell Assistant Program

Options:
  -V, --version               output the version number
  -h, --help                  display help for command

Commands:
  init [options]              set up your `asap` installation
  do [options] <instruction>  ask asap to do something
  help [command]              display help for command
```
