sbt testOnly:

Running a specific test suite. For example: DefaultAdaptexClientIntegrationSpec is in data-access project, Now I want to run that file only

sbt "data-access/testOnly **DefaultAdaptexClientIntegrationSpec"

If you don't have subproject and spec is in the root project then you can run like below

sbt testOnly **DefaultAdaptexClientIntegrationSpec

The testOnly task also accepts wildcards, which is how I usually specify the suites:
sbt testOnly **.APISpec **.AT*Spec //It will run all the test files that ends with file name by APISpec and file name contains 'Spec' after 'AT'


Run single test or a subset of related tests from spec file. For example: Spec file DefaultAdaptexClientIntegrationSpec is in data-access project, Now I want to run single test or a subset of related tests that contains word 'mismatched' 

sbt "data-access/testOnly **DefaultAdaptexClientIntegrationSpec -- -z mismatched"

Here (-- is required to mark the start of parameters passed verbatim to the test runner)
