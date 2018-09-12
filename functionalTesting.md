---
layout: default
title: Functional Testing
---

All pipelines and workflows which are a part of BioWDL should have functional
testing to ensure nothing breaks during updates. The functional testing for
BioWDL is written in scala.

## Source file structure
The scala code used for testing the workflows is located in the
`src/test/scala/biowdl/test` folder of the corresponding repository. This
folder contains the `biowdl.test` package. For each workflow in a repository
a sub-package should be created, containing the scala code for testing that
workflow. This sub-package should be named after the workflow it tests. There
are three scala files required per workflow: `*.scala`, `*Success.scala` and
`*Test.scala`. These files should be named after the workflow they test as
well. For example, for a workflow "foo.wdl" the test files should be called
"Foo.scala", "FooSuccess.scala" and "FooTest.scala", and they should be part of
the "biowdl.test.foo" package.

In addition, in the root of the repository a `build.sbt`, `environment.yml` and
`Jenkinsfile` should be present.

```
./
+-- ...
+-- src/
|   +-- test/
|       +-- scala/
|           +-- biowdl/
|               +-- test/
|                   +-- foo/
|                   |   +-- Foo.scala
|                   |   +-- FooSuccess.scala
|                   |   +-- FooTest.scala
|                   +-- bar/
|                       +-- Bar.scala
|                       +-- BarSuccess.scala
|                       +-- BarTest.scala
+-- build.sbt
+-- Jenkinsfile
+-- environment.yml
+-- ...
```

## Testing source files
All scala code should be formatted using the `scalafmt` command and a header
should be added to all scala files, using the `headerCreate` command in sbt.

The functional tests are defined in three files: `*.scala`, `*Success.scala`
and `*Test.scala`. All of these files should part of the `biowdl.test.*`
package. `*` should be replaced by the name of the workflow being tested.

Below the contents of these files is explained for testing single-sample
workflows. For multi-sample workflow testing is largely the same with
[some exceptions](#Multi-Sample-workflows).

### \*.scala
This file contains a `trait` of the same name as the workflow being tested. It
is used to provide the inputs and other values needed to run the workflow.

This trait should extend either the `nl.biopet.utils.biowdl.Pipeline` trait
as well as the `nl.biopet.utils.biowdl.references.Reference` trait if the
workflow requires a (fasta) reference input and the
`nl.biopet.utils.biowdl.annotations.Annotation` trait if it requires a GTF or
Refflat annotation input.

The trait defined in this file should define the inputs to be given to the
workflow. These inputs should be defined by overriding a `Map` called `inputs`.
Be sure to append the original Map (`super.inputs`) to the new one. It should
also define the actual WDL file to be run under the `startFile` variable and
initialize variables to represent the inputs for the workflow. These variables
should only be initialized and not defined (this will happen in the
\*Test.scala file), unless they should be inferred from another input
(eg. BAM indexes).

> Note that the output directory is already defined under the `outputDir`
variable in the `Pipeline` trait. It does still have to be added to the
inputs Map, if applicable for the workflow in question!

**example:**
```scala
package biowdl.test.foo

import java.io.File

import nl.biopet.utils.biowdl.multisample.MultisamplePipeline
import nl.biopet.utils.biowdl.references.Reference

trait Foo extends Pipeline with Reference {

  def dbsnpFile: File // initialize input

  // Map representing the input JSON
  override def inputs: Map[String, Any] =
    super.inputs ++
      Map(
        "Foo.outputDir" -> outputDir.getAbsolutePath,
            //outputDir defined in the Pipeline trait
        "Foo.dbsnp" -> dbsnpFile.getAbsolutePath,
        "Foo.fasta" -> referenceFasta.getAbsolutePath
            //referenceFasta is defined in the Reference trait
      )

  // The WDL file to be tested.
  def startFile: File = new File("foo.wdl")
}
```

### \*Success.scala
This file contains a `trait` which defines the requirements for a workflow run
to be considered successful. It should extend the trait defined in the
\*.scala file, as well as the `nl.biopet.utils.biowdl.PipelineSuccess`
trait. `PipelineSuccess` defines a basic test to check whether the workflow run
was considered successful by Cromwell.  
Within the trait's body additional tests should be defined, to be performed once
the workflow is done running. Some basic wrappers exist for commonly used
tests, such as the `addMustHaveFile`-method which adds a test to check whether
a file exists.

Custom tests should be added as methods within the trait with the `@Test`
annotation, be sure to import `org.testng.annotations.Test`.

**example:**
```scala
package biowdl.test.foo

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
`*Success.scala` file. If the trait defined in the `*.scala` file extended
`Reference` or `Annotation` the classes here should extend the
`nl.biopet.utils.biowdl.references.TestReference` and
`nl.biopet.utils.biowdl.annotations.TestAnnotation` traits as appropriate.

**example:**
```scala
package biowdl.test.foo

import java.io.File

import nl.biopet.utils.biowdl.fixtureFile
import nl.biopet.utils.biowdl.references.TestReference

class FooTestWithoutDbsnp
    extends FooSuccess
    with TestReference

class FooTestWithDbsnp
    extends FooTestWithoutDbsnp {
  def dbsnpFile: File = fixtureFile("references", "dbsnp.vcf.gz")
}
```

## Multi-Sample workflows
Testing multi-sample workflows largely works the same as
[testing single-sample workflows](#Testing-source-files), with a one
exceptions:  
Rather than extending `nl.biopet.utils.biowdl.Pipeline` in the `*.scala file`,
`nl.biopet.utils.biowdl.multisample.MultisamplePipeline` should be extended.
This will automatically generate a sample configuration file and add it to the
inputs Map. Adding samples to test cases can be done by extending various
predefined traits from the `nl.biopet.utils.biowdl.samples`.

**example:**
\*.scala
```scala
package biowdl.test.bar

import java.io.File

import nl.biopet.utils.biowdl.multisample.MultisamplePipeline
import nl.biopet.utils.biowdl.references.Reference

trait Bar extends MultisamplePipeline with Reference {

  def dbsnpFile: File // initialize input

  // Map representing the input JSON
  override def inputs: Map[String, Any] =
    super.inputs ++
      Map(
        "Bar.outputDir" -> outputDir.getAbsolutePath,
            //outputDir defined in the Pipeline trait
        "Bar.dbsnp" -> dbsnpFile.getAbsolutePath,
        "Bar.fasta" -> referenceFasta.getAbsolutePath
            //referenceFasta is defined in the Reference trait
      )

  // The WDL file to be tested.
  def startFile: File = new File("bar.wdl")
}
```

\*Success.scala
```scala
package biowdl.test.bar

import nl.biopet.utils.biowdl.PipelineSuccess
import org.testng.annotations.Test

trait BarSuccess extends Bar with PipelineSuccess {
  addMustHaveFile("output.txt") //Test if the expected output exists

  @Test
  def testContents(): Unit = {
    // Do some fancy testing here
  }
}
```

\*Test.scala
```scala
package biowdl.test.bar

import java.io.File

import nl.biopet.utils.biowdl.fixtureFile
import nl.biopet.utils.biowdl.references.TestReference
import nl.biopet.utils.biowdl.samples.{
  Wgs1SingleEnd,
  Wgs1PairedEnd,
  Wgs2SingleEnd,
  Wgs2PairedEnd
}

class BarTestPairedEnd
    extends BarSuccess
    with TestReference
    with Wgs1PairedEnd
    with Wgs2PairedEnd {
  def dbsnpFile: File = fixtureFile("samples", "wgs2", "wgs2.vcf.gz")
}

class BarTestSingleEnd
    extends BarSuccess
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
This is a conda environment file. Please make sure all tools used in a workflow
are mentioned here, including the specific versions of the tools.

## Jenkinsfile
The contents of the Jenkinsfile should be as follows in order for tests to be
run. An example of such a Jenkinsfile can be found
[here](https://github.com/biowdl/pipeline-template/blob/develop/Jenkinsfile).
