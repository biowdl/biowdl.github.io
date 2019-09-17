---
layout: default
title: Testing
---

All pipelines and workflows which are a part of BioWDL have automated
testing to ensure nothing breaks during updates. These tests are
implemented using the
[pytest-workflow](https://pytest-workflow.readthedocs.io/en/stable/)
framework.

All tests defined in the pytest-workflow configuration should have either
the `integration` or `functional` tag to indicate whether or not the tests
should be run on every PR or just periodically.

## Test file structure
Inside the repository of the workflows a folder named "test" can be found.
This folder contains all the test files, including input JSON files,
test configuration YAML files and any required data and cromwell configurations.

Note that most of these directories and files are optional. Samplesheets
will only be present if a workflow requires a samplesheet input, for example.
```
.
├── cromwell_config
│   └── docker.conf
├── cromwell_options.json
├── data
│   └── <test data>
├── functional
│   └── <input JSON files for any potential functional (large data) tests>
├── integration
│   └── <input JSON files for any potential integration (small data) tests>
├── samplesheets
│   └── <sample sheets for the test samples>
├── <custom pytest scripts for more complex testing>
└── <pytest-workflow YAML files>
```

## Data files
Data files may be anything from VCF files to reference FASTA files.
These files will be referenced in the input JSONs or samplesheets.
Here relative paths should be used in order for the tests to be runnable
on most systems.

Preferably these test files should be small (<1MB), as to not slow down
downloads and git operations. 

## Functional test
By functional test we mean large scale tests on real (or at least 
real-sized) data. The associated data files should therefore not be
included in the repository. These tests are only run on the LUMC's
internal cluster and are only run periodically.

## Integration tests
These are the tests that will be run for every PR that is made. These
should only use small data which is included in the repository.

## Travis and Jenkins files
In order for tests to be run configurations for the used CI systems are
needed. These are [travis](https://travis-ci.org/) and 
[Jenkins](https://stingray.researchlumc.nl/)

Examples of these configuration files can be found in the
[pipeline-template](https://github.com/biowdl/pipeline-template):
- [Jenkinsfile](https://github.com/biowdl/pipeline-template/blob/develop/Jenkinsfile)
- [.travis.yml](https://github.com/biowdl/pipeline-template/blob/develop/.travis.yml)