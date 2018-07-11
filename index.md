---
layout: default
---

## About
BioWDL is a collection of pipelines and workflows usable for a variety of
sequencing related analyses. They are made using
[WDL](https://software.broadinstitute.org/wdl/) and closely related to
[BIOPET](https://biopet.github.io/).

BioWDL is developed at the Leiden University Medical Center by
[the SASC team](http://sasc.lumc.nl/).

## Style guidelines
You can find the style guidelines [here](styleGuidelines.md).

## Contact
For any question related to any BioWDL workflows, please use the relevant
github issue tracker or contact the SASC team directly at: sasc@lumc.nl.

## Pipelines and other repositories

| Name | Category | Description |
|-|-|-|
{% for repo in site.github.public_repositories -%}
{% if repo.has_pages -%}
{% assign category = repo.description | split: "Category:" -%}
| [{{ repo.name }}](/{{repo.name}})  ([repo]({{repo.html_url}}))| {{ category | last }} | {{ category | first }} |
{% endif -%}
{% endfor -%}
