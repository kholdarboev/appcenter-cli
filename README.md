# Sonoma CLI - command line for Sonoma
(Kind of obvious from the name, really)

Sonoma CLI is a command line interface to interact with the Sonoma services. It's intended
for use by mobile developers and people building scripts that need to interact with Sonoma (for example,
local CI configurations).

## Technologies Used

sonoma cli is written using Node.js version 6 and [Typescript](http://typescriptlang.org). Wrappers over the Bifrost HTTP API are
generated using the [AutoRest][https://github.com/Azure/autorest] code generator. And the usual
plethora of npm modules.

We use [mocha](https://http://mochajs.org/) for a test runner / framework. [Chai](http://http://chaijs.com/) is
the assertion library. [Sinon](http://sinonjs.org) is the general mocking library, and [nock](https://github.com/node-nock/nock)
will be used to record and playback mock http traffic. [Note: this isn't set up yet.]

# Setting Up

## Prerequisites

Install the latese version of Node 6 from [here](https://nodejs.org). If you are on a Mac, we recommend
a 64-bit version.

Also have a working git installation. You will need access to this [repo](https://github.com/Microsoft/sonoma-cli).
At this time, repo access is managed manually. Everyone in the sonoma group should have read access. If not, please
contact Chris Tavares (ctavares@microsoft.com or @ctavares-ms on Xamarin slack)
or Matt Gibbs (matt.gibbs@microsoft.com or @mattgi on Xamarin slack) to get added to the repo.

### Optional Tools

The repo is set up so that once you've got node and the repo, everything you need to do will work. However, it is
convenient to install some tools globally in addition.

#### Node Version Management

If you need multiple versions of node on your machine at the same time, consider a node version manager.
For Mac or Linux, try [nvm](https://github.com/creationix/nvm). For Windows machines, [nodist](https://github.com/marcelklehr/nodist)
works well.

#### Typescript compiler and Typings

The typescript compilation can be run via the `npm run build` command, but if you want the Typescript compiler available directly,
install it on you machine by doing `npm install -g typescript`.

The Typings tool is useful if you're bringing in a new exteral Javascript library and want to get access to type definitions
for that library. The typings files are checked into the repo, but if you want to download and add new ones, you'll need to
install typings: `npm install -g typings`.

#### gulp

gulp is used as a task runner to get everything built and in the right place. Everything is hooked up through NPM scripts, but if you
want to save some characters at the command line, install `npm install -g gulp-cli`.

#### ts-node

By default, to run the CLI you need to compile the typescript explicitly first. The `ts-node` command line tool will let you run
`.ts` files directly from the command line. Install it with `npm install -g ts-node`.

#### mono

If you want to regenerate the HTTP client code from a non-Windows machine, you'll need mono installed and on your path.
see the [Mono download page](http://www.mono-project.com/download/) for downloads and installation instructions.

## Troubleshooting

If you are running on a Mac and have to use `sudo` to install global npm modules (for example, `npm install -g typescript`),
the please check out [this tutorial](https://docs.npmjs.com/getting-started/fixing-npm-permissions).

# Building

After installing node and cloning the repo, do:

 1. `npm install`
 2. `npm run build`

To run the test suite, do:
 3. `npm test`

# Running the cli in development

If you're using node directly, do:

 1. `npm run build`
 2. `node dist/index.js <command...> <args...>`

If you've installed `ts-node` as mentioned above, you can skip the build step and do:

 1. `ts-node src/index.ts <command...> <args...>`

# Scripts

There are a bunch of scripts in package.json file. Here's what they are and what they do:

| Script command | What it does |
|----------------|------------- |
| `npm run build` | Compiles the typescript into javascript, creates `dist` directory |
| `npm run test` | Runs the test suite. Can also be run with `npm test` |
| `npm run watch-test` | Runs a watcher on the test file that will rerun tests automatically on save |
| `npm run clean` | Cleans up any compilation output |
| `npm run autorest` | Regenerate the HTTP client libraries. Downloads required tools as part of this process |

There will be more over time.

## Gulp targets

The gulpfile.js file contains the following targets that can be called manually if you desire

| Target | npm script |What it does |
|--------|------------|-------------|
| `clean`  | `clean` | Deletes the dist folder |
| `build-ts` | | Runs typesscript compiler, using settings in tsconfig.json |
| `copy-assets` | | Copies .txt files from src to dist (category descriptions) |
| `copy-generated-client` | | Copies the generated HTTP client code to dist |
| `build` | `build` | Runs the build (build-ts, copy-assets, copy-generated-clients) |
| `clean-autorest` | | Deleted all generated code from src directory |
| `autorest` | `autorest` | Regenerate client code from swagger/bifrost.swagger.json file |
| `default` | | Runs the build task |

# Touring the codebase

## General design principles

Use promises for async operations. Use `async`/`await` in Typescript for handling typical promises.

If you've got a bunch of related code, export the library as a whole in a single import using an `index.ts` file.
For example, the `profile` library includes files `environment.ts` and `profile.ts`, but users of the module
just needs to do `import { stuff } from "../util/profile"`

Don't overuse objects or inheritance. In many cases global functions or modules can do just as well and be easier to consume.

### Directory structure

#### dist

Created by the `npm run build` command, contains the compiled-to-javascript code.

#### src

This is where the source code for the CLI lives.

#### src/commands

The implementation of each command is in this directory. Each category (distribute, build, app, etc) will be a subdirectory of this directory. Each command lives in an individual source file with the same name as the command.

For example:

| Command | Source File |
| ------- | ----------- |
| `sonoma login` | src/commands/login.ts |
| `sonoma profile configure` | src/commands/profile/configure.ts |
| `sonoma apps list` | src/commands/apps/list.ts |

The command line parser and dispatcher uses the directory structure and file names to determine which code to run, so the naming conventions are important.

In addition, place a `category.txt` file in your category directory. The contents of
this file will be displayed by the help system when getting help for the category.

If you have shared code across commands in your category, you can add a directory named `lib` in your category's directory and put that code there. The command line dispatcher will explicitly ignore this directory and not try to accidentally run your utility code from the command line.

#### src/util

This contains framework and utility code used across all the commands. See readme files in each directory for specific details of each one. (Working on these.)

#### src/util/apis

Http client wrappers and utility code for handling HTTP communication with Bifrost.

#### src/util/apis/generated

Autorest-generated client code for talking to Bifrost.

#### src/util/commandline

The command line parser and dispatching code, along with base class and decorators for implementing commands.

#### src/util/interaction

Central point for all user I/O done by commands. Use `interaction.prompt` to get input from a user, and
`interaction.out` to output various forms of results.

Commands should use these rather than directly using `console.log` because the interaction library handles output formats (the `--format` switch) and the `--quiet` switch transparently to the command author.

#### src/util/profile

Code for storing and retrieving information about the current logged in user.

#### scripts

Support files for build and packaging.

#### test

Test code lives here. For new tests create a subdirectory structure corresponding to the `src` folder. Test code will be automatically run if you name the file `<testname>-test.ts` or `<testname>-test.js`. We recommend using Typescript for you tests to keep things consistent across the entire codebase.

#### typings

Stores type definitions for the external Javascript libraries used. These are checked in rather than dynamically downloaded in case we need to edit them.

# Development Processes

We follow the standard GitHub flow. Each person working on the cli should create their own fork of the repo. Work in your own repo (preferably on a feature branch). When ready, send a pull request to the master Microsoft/sonoma-cli repo against the master branch. After review, the pull request will be merged.

Issue tracking will be done on [VSO](https://mseng.visualstudio.com/Mobile%20DevOps/Command%20Line%20Interface/).

# Building Installers

TBD. We'll need builds for a Mac installer, Windows MSI, and at least one format of Linux package, plus be able to push to NPM.

