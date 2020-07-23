# Native image integration tests
Builds Native image executables for small applications based on some Native image capable runtimes such
as Quarkus, Micronaut and Helidon. It also provides a convenient way of building small test apps for Native image backbox testing.

## Prerequisites

The TS expects you run Java 11+ and have ```ps``` program available on your Linux/Mac
and ```wmic``` (by default present) on your Windows system.
Native image builds also require you have the following packages installed:
* glibc-devel
* zlib-devel
* gcc
* libffi-devel

On Fedora/CentOS/RHEL they can be installed with:
```bash
dnf install glibc-devel zlib-devel gcc libffi-devel libstdc++-static
```

On Ubuntu-like systems with:
```bash
apt install gcc zlib1g-dev libffi-dev build-essential
```

## Usage

The basic usage that runs all tests:
```
mvn clean verify -Ptestsuite
```

One can fine-tune excluded test cases or tests with `excludeTags`, e.g. `-DexcludeTags=runtimes`
to exclude all `runtimes` tests or `-DexcludeTags=micronaut` to exclude just one of them. 
You can also exclude everything and include just `reproducers` suite: `-DexcludeTags=all -DincludeTags=reproducers`

## RuntimesSmokeTest

The goal is to build and start applications with some real source code that actually
exercises some rudimentary business logic of selected libraries.

### Collect results:

There are several log files archived in `testsuite/target/archived-logs/` after runtimes test execution. e.g.:

```
org.graalvm.tests.integration.RuntimesSmokeTest/quarkusFullMicroProfile/report.md
org.graalvm.tests.integration.RuntimesSmokeTest/quarkusFullMicroProfile/build-and-run.log
org.graalvm.tests.integration.RuntimesSmokeTest/quarkusFullMicroProfile/measurements.csv
```

`report.md` is human readable description of what commands were executed and how much time and memory was spent to
get the expected output form the runtime's web server. `measurements.csv` contains the same data in a machine friendly form.
Last but not least, `build-and-run.log` journals the whole history of all commands and their output.

There is also an aggregated `testsuite/target/archived-logs/aggregated-report.md` describing all the testsuite did.

## AppReproducersTest

This part of the test suite runs smaller apps that are not expected to offer a web server, so just their stdout/stderr
is checked. No measurements are made as to the used memory at the time of writing, but it could be easily changed.
e.g. a current example:

```
org.graalvm.tests.integration.AppReproducersTest/randomNumbersReinit/report.md
org.graalvm.tests.integration.AppReproducersTest/randomNumbersReinit/build-and-run.log
``` 

## Logs and Whitelist

Logs are checked for error and warning messages. Expected error messages can be whitelisted in [WhitelistLogLines.java](./testsuite/src/it/java/org/graalvm/tests/integration/utils/WhitelistLogLines.java).

## Thresholds

The test suite works with ```threshold.properties```, e.g. `./apps/quarkus-full-microprofile/threshold.properties`: 

```
linux.time.to.first.ok.request.threshold.ms=50
linux.RSS.threshold.kB=120000
windows.time.to.first.ok.request.threshold.ms=80
windows.RSS.threshold.kB=120000
```

**THIS IS NOT A PERFORMANCE TEST** The thresholds are in place only as a sanity check to make sure
an update to Native image did not make the application runtime to run way over the expected benevolent values.

The measured values are simply compared to be less or equal to the set threshold. One can overwrite the `threshold.properties`
by using env variables or system properties (in this order). All letter are capitalized and dot is replaced with underscore, e.g.

```
APPS_QUARKUS_FULL_MICROPROFILE_LINUX_TIME_TO_FIRST_OK_REQUEST_THRESHOLD_MS=35 mvn clean verify -Ptestsuite 
```

### Example failures

With a rather harsh threshold of 5ms:

```
org.opentest4j.AssertionFailedError: 
Application QUARKUS_FULL_MICROPROFILE took 27 ms to get the first OK request, 
which is over 5 ms threshold. ==> expected: <true> but was: <false>
        at org.graalvm.tests.integration.RuntimesSmokeTest.testRuntime(RuntimesSmokeTest.java:131)
        at org.graalvm.tests.integration.RuntimesSmokeTest.quarkusFullMicroProfile(RuntimesSmokeTest.java:147)
```

For logs checking, see an example failure before we whitelisted the particular warning:

```
[ERROR] Failures: 
[ERROR]   RuntimesSmokeTest.micronautHelloWorld:154->testRuntime:91 
  build-and-run.log log should not contain error or warning lines that are not whitelisted.
  See testsuite/target/archived-logs/org.graalvm.tests.integration.RuntimesSmokeTest/micronautHelloWorld/build-and-run.log
  and check these offending lines: 
    [WARNING] Discovered module-info.class. Shading will break its strong encapsulation.
```

Happy testing!