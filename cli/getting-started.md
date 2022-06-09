---
pos: 1
title: Getting Started with Saleor CLI
description:
next:
  path: /api/creating-apps/
quizQuestions:
  - question: Saleor CLI enables you to easily create your organisation, project and environment.
    answerType: radio
    answerOptions:
      - answer: Yes
      - answer: No
        isCorrect: true
    feedback: "You cannot create your organisation using CLI."
---

# Getting Started with Saleor CLI

SALEOR VERSION
3.4.2

Saleor provides you with a quick way to set up the platform for your client. Instead of forking the repository or resorting to the web version you may use the command-line interface (CLI), which gives you a quick access to many actions, like registering an account, logging in or handling your projects and environments from the level of a terminal.

This page guides you through a list of the Saleor CLI commands, alongside their optional parameters for additional behavior. It will get you up and running with your Saleor Cloud instance in no time.

## Prerequisites

1. In order to utilise the features of Saleor CLI, you need to be aware of the main concepts that Saleor Cloud is built on.

![Saleor Cloud Setup Structure Diagram](./StructureOrganization.png)

#### Organisation

The place where you will store data about billing, staff members and other general configuration details.

#### Project

It aggregates your environments. You can have many projects in an organisation.

#### Environment

One particular instance of your e-commerce data. It is equipped with a GraphQL-powered database and accessible through a dedicated dashboard. Every newly created environment defaults to a _sandbox_ mode, you can then stage it for _production_. You can have many environments in a project.

2. You need to register your account in [Saleor Cloud](https://saleor.io/) and establish your organisation.

## Installation

To download and install Saleor CLI, run the following command:
`npm i -g saleor-cli`

## Getting -help

Typing `saleor` in the terminal will display a list of possible commands and their descriptions. Typing `saleor <command>` will display additional comands list for this particular command. This way, we can navigate over the interface to trigger specific actions.

## Logging in

Firstly, you need to login into an existing Saleor Cloud account or create a new one. In order to log in let's type:

`saleor login`

command, which will redirect you to a webpage with authentication. There, you need to pass in your login credentials. After a successful login, you can come back to the CLI.

## Listing your organisation(s)

Once logged in, you can use CLI to check upon your organisation(s). To list them simply type in:

`saleor org list`

You will be provided with the basic information about your active organisations, such as a slug, a name or a company email. If you need to display more details, type in:

` saleor org show`

## Creating a new Project

After setting up the organisation, you are ready to create your first project. In the terminal, type in:

`saleor project create`

After selecting the organisation, typing in the name and choosing the region, you'll be set up.

## Creating a new Environment

Last thing to set up is the environment. The setup flow is going to be quite the same as with the previous iterations. In the terminal use:

`saleor env create`

In the process of establishing the environment, you will be prompted for some details, like the project name, database template or sandbox version. You will also have the possibility to opt in for a dashboard access and basic authorisation.
And that's it! Your client is now ready to use the features of the Salor Cloud.
