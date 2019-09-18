---
layout: default
title: Home
---

## About
BioWDL is a collection of pipelines and workflows usable for a variety of
sequencing related analyses. They are made using
[WDL](http://www.openwdl.org/) and are developed at the
[Leiden University Medical Center](https://www.lumc.nl/) by 
the SASC team.

## Pipelines and other repositories
The following are the pipelines and repositories from BioWDL. The repositories
are divided into the following categories:
- **Single-Sample**: Pipelines or workflows which can only be run for a single
sample at a time. These are production ready.
- **Multi-Sample**: Pipelines or workflows which can be run for a multiple
samples at the same time and perform inter-sample operations. These are
production ready.
- **Single-Sample InDevelopment**: Single-Sample pipelines and workflows which
are still in development.
- **Multi-Sample InDevelopment**: Multi-Sample pipelines and workflows which
are still in development.
- **Other**: Repositories which contain something other than workflows or
pipelines.

| Name | Category | Description |
|-|-|-|
{% for repo in site.github.public_repositories -%}
{% if repo.has_pages -%}
{% if repo.description contains "Category:" -%}
{% assign category = repo.description | split: "Category:" -%}
| [{{ repo.name }}](/{{repo.name}})  ([repo]({{repo.html_url}}))| {{ category | last }} | {{ category | first }} |
{% endif -%}
{% endif -%}
{% endfor %}

## Contributing
Please feel free to make a pull requests if you have something you would like
to add or improve. Below you can find a link to the style guidelines these
pipelines should adhere to, as well as some notes on adding tests.

### Style guidelines
You can find the style guidelines [here](styleGuidelines.md). The BioWDL
pipelines should adhere to these guidelines as much as possible. If something
is unclear or you feel something is missing, please don't hesitate to make an
[issue](https://github.com/biowdl/biowdl.github.io/issues) or
[pull request](https://github.com/biowdl/biowdl.github.io/pulls).

### Functional tests
You can find documentation on the automated testing used in BioWDL
[here](functionalTesting.md).

## Contact
For any question related to any BioWDL workflows, please use the relevant
github issue tracker or contact the SASC team directly at: sasc@lumc.nl.
