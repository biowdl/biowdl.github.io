---
layout: default
title: Functional Testing
---

All pipelines and workflows which are a part of BioWDL should have functional
testing to ensure nothing breaks during updates. The functional testing for
BioWDL is written in scala.

## Source file structure
The scala code used for testing the pipelines is located in the
`src/test/scala/biowdl/test` folder of the corresponding repository.
Should there be multiple workflows in the same repository then the scala code
should be in a folder per workflow. There are three scala files required per
workflow: `*.scala`, `*Success.scala` and `*Test.scala`. These files should be
named after the workflow they test, eg. for a workflow "foo.wdl" the test files
should be called "Foo.scala", "FooSuccess.scala" and "FooTest.scala".

In addition, in the root of the repository a `build.sbt`, `environment.yml` and
`Jenkinsfile` should be present.

**Multiple workflows:**
```
./
+-- ...
+-- src/
|   +-- test/
|       +-- scala/
|           +-- biowdl/
|               +-- test/
|                   +-- workflow1/
|                   |   +-- Workflow1.scala
|                   |   +-- Workflow1Success.scala
|                   |   +-- Workflow1Test.scala
|                   +-- workflow2/
|                       +-- Workflow2.scala
|                       +-- Workflow2Success.scala
|                       +-- Workflow2Test.scala
+-- build.sbt
+-- Jenkinsfile
+-- environment.yml
+-- ...
```

**Singular workflow:**
```
./
+-- ...
+-- src\
|   +-- test\
|       +-- scala\
|           +-- biowdl\
|               +-- test\
|                   +-- Workflow.scala
|                   +-- WorkflowSuccess.scala
|                   +-- WorkflowTest.scala
+-- build.sbt
+-- Jenkinsfile
+-- environment.yml
+-- ...
```

## Testing source files
The functional tests are currently defined in three files: `*.scala`,
`*Success.scala`, `*Test.scala`. All of these files should part of the
`biowdl.test` package.

The scala code should be formatted using the `scalafmt` command and a header
should be added using the `headerCreate` command in sbt.

### \*.scala
This file contains a `trait` of the same name as the workflow or pipeline
being tested. It is used to provide the inputs and other values needed to
run the pipeline.

This trait should extend either the `nl.biopet.utils.biowdl.Pipeline` trait
or the `nl.biopet.utils.biowdl.multisample.MultisamplePipeline` trait if it is a
multi-sample pipeline.

It should also extend the `nl.biopet.utils.biowdl.references.Reference` trait
if the pipeline requires a (fasta) reference input and the
`nl.biopet.utils.biowdl.annotations.Annotation` trait if it requires a GTF or
Refflat annotation input.

The trait defined in this file should define the inputs to be given to the
pipeline. These inputs should be defined by overriding a `Map` called `inputs`.
Be sure to append the original Map (`super.inputs`) to the new one. It should
also define the actual WDL file to be run under the `startFile` variable and
initialize variables to represent the inputs for the pipeline. These variables
should only be initialized and not defined (this will happen in the
\*Test.scala file), unless they should be inferred from another input
(eg. BAM indexes).

> Note that the output directory is already defined under the `outputDir`
variable in the `Pipeline` trait. It does still have to be added to the
inputs Map, if applicable for the pipeline in question!

> In the case of multi-sample pipelines the `sampleConfig` input is already
defined as well and ***is*** already included in the inputs Map!

**example:**
```scala
package biowdl.test

import java.io.File

import nl.biopet.utils.biowdl.multisample.MultisamplePipeline
import nl.biopet.utils.biowdl.references.Reference

trait Foo extends MultisamplePipeline with Reference {

  def dbsnpFile: File // initialize input

  // Map representing the input JSON
  override def inputs: Map[String, Any] =
    super.inputs ++
      Map(
        "Foo.outputDir" -> outputDir.getAbsolutePath,
        "Foo.dbsnp" -> dbsnpFile.getAbsolutePath
      )

  // The WDL file to be tested.
  def startFile: File = new File("foo.wdl")
}
```

### \*Success.scala
This file contains a `trait` which defines the requirements for a pipeline run
to be considered successful. It should extend the trait defined in the
\*.scala file, as well as the `nl.biopet.utils.biowdl.PipelineSuccess`
trait. `PipelineSuccess` defines a basic test to check whether the pipeline run
was considered successful by Cromwell.
Within the trait's body additional tests should be defined, to be perform once
the pipeline is done running. Some basic wrappers exist for commonly used
tests, such as the `addMustHaveFile`-method which adds a test to check whether
a file exists.

Custom tests should be added as methods within the trait with the `@Test`
annotation, be sure to import `org.testng.annotations.Test`.

```scala
package biowdl.test

import nl.biopet.utils.biowdl.PipelineSuccess
import org.testng.annotations.Test

trait FooSuccess extends Foo with PipelineSuccess {
  addMustHaveFile("output.txt") //Test if the expected output exists

  @Test
  def testContents(): Unit = {
    // Do some fancy testing here
  }
}
```

### \*Test.scala
This file contains the actual test cases. For each test case a class is added
to define the appropriate inputs, using the variables initialized in the
`*.scala` file. These classes should extend the `*Success` trait defined in the
`*Success.scala` file.

If the trait defined in the `*.scala` file extended `Reference` or `Annotation`
the classes here should extend the
`nl.biopet.utils.biowdl.references.TestReference` and
`nl.biopet.utils.biowdl.annotations.TestAnnotation` traits as appropriate.

In the case of multi-sample pipelines, the various predefined samples
can be added by extending a trait from the `nl.biopet.utils.biowdl.samples`
package.

```scala
package biowdl.test

import java.io.File

import nl.biopet.utils.biowdl.fixtureFile
import nl.biopet.utils.biowdl.references.TestReference
import nl.biopet.utils.biowdl.samples.{
  Wgs1SingleEnd,
  Wgs1PairedEnd,
  Wgs2SingleEnd,
  Wgs2PairedEnd
}

class FooTestPairedEnd
    extends FooSuccess
    with TestReference
    with Wgs1PairedEnd
    with Wgs2PairedEnd {
  def dbsnpFile: File = fixtureFile("samples", "wgs2", "wgs2.vcf.gz")
}

class FooTestSingleEnd
    extends FooSuccess
    with TestReference
    with Wgs1SingleEnd
    with Wgs2SingleEnd {
  def dbsnpFile: File = fixtureFile("samples", "wgs2", "wgs2.vcf.gz")
}
```

## build.sbt
This is the SBT build file. Please use the template below and replace the
values between the `<>`:
```
organization := "com.github.biopet"
organizationName := "Biowdl"
name := <repository name>

biopetUrlName := <repository name>

biopetIsTool := false

concurrentRestrictions := Seq(
  Tags.limitAll(
    Option(System.getProperty("biowdl.threads")).map(_.toInt).getOrElse(1)),
  Tags.limit(Tags.Compile, java.lang.Runtime.getRuntime.availableProcessors())
)

developers += Developer(id = <github username>,
                        name = <name>,
                        email = <email address>,
                        url = <url>)

scalaVersion := "2.11.12"

libraryDependencies += "com.github.biopet" %% "biowdl-test-utils" % "0.1-SNAPSHOT" % Test changing ()
```

## environment.yml
This is a conda environment file. Please make sure all tools used in a pipeline
are mentioned here, including the specific versions of the tools.

## Jenkinsfile
The contents of the Jenkinsfile should be as follows in order for tests to be
run:
```
pipeline {
    agent {
        node {
            label 'local'
        }
    }
    tools {
        jdk 'JDK 8u162'
    }
    environment {
        CROMWELL_JAR      = credentials('cromwell-jar')
        CROMWELL_CONFIG   = credentials('cromwell-config')
        CROMWELL_BACKEND  = credentials('cromwell-backend')
        FIXTURE_DIR       = credentials('fixture-dir')
        CONDA_PREFIX      = credentials('conda-prefix')
        THREADS           = credentials('threads')
        OUTPUT_DIR        = credentials('output-dir')
        FUNCTIONAL_TESTS  = credentials('functional-tests')
        INTEGRATION_TESTS = credentials('integration-tests')
    }
    stages {
        stage('Init') {
            steps {
                sh 'java -version'
                checkout scm
                sh 'git submodule update --init --recursive'
                script {
                    def sbtHome = tool 'sbt 1.0.4'
                    env.outputDir= "${OUTPUT_DIR}/${JOB_NAME}/${BUILD_NUMBER}"
                    env.condaEnv= "${outputDir}/conda_env"
                    env.sbt= "${sbtHome}/bin/sbt -Dbiowdl.functionalTests=${FUNCTIONAL_TESTS} -Dbiowdl.integrationTests=${INTEGRATION_TESTS} -Dbiowdl.outputDir=${outputDir} -Dcromwell.jar=${CROMWELL_JAR} -Dcromwell.config=${CROMWELL_CONFIG} -Dcromwell.extraOptions=-Dbackend.providers.${CROMWELL_BACKEND}.config.root=${outputDir}/cromwell-executions -Dbiowdl.fixtureDir=${FIXTURE_DIR} -Dbiowdl.threads=${THREADS} -no-colors -batch"
                    env.activateEnv= "source ${CONDA_PREFIX}/activate \$(readlink -f ${condaEnv})"
                    env.createEnv= "${CONDA_PREFIX}/conda-env create -f environment.yml -p ${condaEnv}"
                }
                sh "rm -rf ${outputDir}"
                sh "mkdir -p ${outputDir}"

                sh "#!/bin/bash\n" +
                        "set -e -v -o pipefail\n" +
                        "${sbt} clean evicted scalafmt headerCreate | tee sbt.log"
                sh 'n=`grep -ce "\\* com.github.biopet" sbt.log || true`; if [ "$n" -ne \"0\" ]; then echo "ERROR: Found conflicting dependencies inside biopet"; exit 1; fi'
                sh "git diff --exit-code || (echo \"ERROR: Git changes detected, please regenerate the readme, create license headers and run scalafmt: sbt biopetGenerateReadme headerCreate scalafmt\" && exit 1)"
            }
        }

        stage('Submodules develop') {
            when {
                branch 'develop'
            }
            steps {
                sh 'git submodule foreach --recursive git checkout develop'
                sh 'git submodule foreach --recursive git pull'
            }
        }

        stage('Create conda environment') {
            steps {
                sh "#!/bin/bash\n" +
                    "set -e -v -o pipefail\n" +
                    "${createEnv}\n"
            }
        }

        stage('Build & Test') {
            steps {
                sh "#!/bin/bash\n" +
                        "set -e -v\n" +
                        "${activateEnv}\n" +
                        "${sbt} test"
            }
        }
    }
    post {
        always {
            junit '**/test-output/junitreports/*.xml'
        }
        failure {
            slackSend(color: '#FF0000', message: "Failure: Job '${env.JOB_NAME} #${env.BUILD_NUMBER}' (<${env.BUILD_URL}|Open>)", channel: '#biopet-bot', teamDomain: 'lumc', tokenCredentialId: 'lumc')
        }
        unstable {
            slackSend(color: '#FFCC00', message: "Unstable: Job '${env.JOB_NAME} #${env.BUILD_NUMBER}' (<${env.BUILD_URL}|Open>)", channel: '#biopet-bot', teamDomain: 'lumc', tokenCredentialId: 'lumc')
        }
        aborted {
            slackSend(color: '#7f7f7f', message: "Aborted: Job '${env.JOB_NAME} #${env.BUILD_NUMBER}' (<${env.BUILD_URL}|Open>)", channel: '#biopet-bot', teamDomain: 'lumc', tokenCredentialId: 'lumc')
        }
        success {
            slackSend(color: '#00FF00', message: "Success: Job '${env.JOB_NAME} #${env.BUILD_NUMBER}' (<${env.BUILD_URL}|Open>)", channel: '#biopet-bot', teamDomain: 'lumc', tokenCredentialId: 'lumc')
        }
    }
}
```
