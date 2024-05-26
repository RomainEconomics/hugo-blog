---
title: "Installation"
date: 2020-08-16
draft: true
description: "How to install the Blowfish theme."
slug: "installation"
tags: ["installation", "docs"]
series: ["Documentation"]
series_order: 2
---

Simply follow the standard Hugo [Quick Start](https://gohugo.io/getting-started/quick-start/) procedure to get up and running quickly.

Detailed installation instructions can be found below. Instructions for [updating the theme](#installing-updates) are also available.

## Installation

These instructions will get you up and running using Hugo and Blowfish from a completely blank state. Most of the dependencies mentioned in this guide can be installed using the package manager of choice for your platform.

### Install Hugo

If you haven't used Hugo before, you will need to [install it onto your local machine](https://gohugo.io/getting-started/installing). You can check if it's already installed by running the command `hugo version`.

{{< alert >}}
Make sure you are using **Hugo version 0.87.0** or later as the theme takes advantage of some of the latest Hugo features.
{{< /alert >}}

You can find detailed installation instructions for your platform in the [Hugo docs](https://gohugo.io/getting-started/installing).

### Blowfish Tools (recommended)

We just launched a new CLI tool to help you get started with Blowfish. It will create a new Hugo project, install the theme and set up the theme configuration files for you. It's still in beta so please [report any issues you find](https://github.com/nunocoracao/blowfish-tools).

Install the CLI tool globally using npm (or other package manager):

```shell
npx blowfish-tools
```

or

```shell
npm i -g blowfish-tools
```

Then run the command `blowfish-tools` to start an interactive run which will guide you through creation and configuration use-cases.

```shell
blowfish-tools
```

You can also run the command `blowfish-tools new` to create a new Hugo project and install the theme in one go. Check the CLI help for more information.

```shell
blowfish-tools new mynewsite
```

Here's a quick video of how fast it is to get started with Blowfish using the CLI tool:

<iframe width="100%" height="350" src="https://www.youtube.com/embed/SgXhGb-7QbU?si=ce44baicuQ6zMeXz" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
