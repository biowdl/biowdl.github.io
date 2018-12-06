---
layout: default
title: Home
---

## About
BioWDL is a collection of pipelines and workflows usable for a variety of
sequencing related analyses. They are made using
[WDL](https://software.broadinstitute.org/wdl/) and are developed at the
Leiden University Medical Center by [the SASC team](http://sasc.lumc.nl/).

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
{% assign category = repo.description | split: "Category:" -%}
| [{{ repo.name }}](/{{repo.name}})  ([repo]({{repo.html_url}}))| {{ category | last }} | {{ category | first }} |
{% endif -%}
{% endfor %}

## Inputs
For some general notes regarding the inputs of the above mentioned pipelines
and workflows, please have a look [here](inputs.md).

## Contributing
Please feel free to make a pull requests if you have something you would like
to add or improve. Below you can find a link to the style guidelines these
pipelines should adhere to, as well as instructions on adding functional tests.

### Style guidelines
You can find the style guidelines [here](styleGuidelines.md). The BioWDL
pipelines should adhere to these guidelines as much as possible. If something
is unclear or you feel something is missing, please don't hesitate to make an
[issue](https://github.com/biowdl/biowdl.github.io/issues) or
[pull request](https://github.com/biowdl/biowdl.github.io/pulls).

### Functional tests
You can find instructions on how to add functional tests
[here](functionalTesting.md).

## Contact
For any question related to any BioWDL workflows, please use the relevant
github issue tracker or contact the SASC team directly at: sasc@lumc.nl.
