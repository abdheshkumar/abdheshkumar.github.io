**sbt testOnly:**

Running a specific test suite. For example: `DefaultClientIntegrationSpec` is in data-access project, Now I want to run that file only

`sbt "data-access/testOnly **DefaultClientIntegrationSpec"`

If you don't have subproject and spec is in the root project, then you can run like below

`sbt testOnly **DefaultClientIntegrationSpec`

The `testOnly` task also accepts wildcards, which is how I usually specify the suites:
`sbt testOnly **.APISpec **.AT*Spec` It will run all the test files that ends with file name by `APISpec` and file name contains `Spec` after `AT`


Run a single test or a subset of related tests from spec file. For example, Spec file `DefaultClientIntegrationSpec` is in `data-access` subproject, Now I want to run single test or a subset of related tests that contains word `mismatched` 

`sbt "data-access/testOnly **DefaultAdaptexClientIntegrationSpec -- -z mismatched"`

Here (`--` is required to mark the start of parameters passed verbatim to the test runner)
