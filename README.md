# Habushu #
[![Maven Central](https://img.shields.io/maven-central/v/org.bitbucket.cpointe.habushu/root.svg)](https://search.maven.org/#search%7Cgav%7C1%7Cg%3A%22org.bitbucket.cpointe.habushu%22%20AND%20a%3A%22habushu%22)
[![License](https://img.shields.io/github/license/mashape/apistatus.svg)](https://opensource.org/licenses/mit)

In Okinawa, habushu (pronounced HA-BU-SHU) is a sake that is made with venomous snake. The alcohol in the snake assists in dissolving the snake's venom and making it non-poinsonous. In Maven, habushu allows Venv-based python projects to be included as part a Maven build. This brings some order and consistency to what can otherwise be haphazardly structured projects.

## Approach ##
Habushu is implemented as a series of [Maven](https://maven.apache.org/) plugins that tie together existing tooling in an opinionated fashion similar to how Maven structures Java projects.  By taking care of manual steps and bringing a predictable order of execution and core naming conventions, habushu increases time spent on impactful work rather than every developer or data scientist building out scripts that meet their personal preferences.  It's likely that no one person will agree with all the opinions habushu brings forth, but the value in being able to run entire builds from a single `mvn clean install` command regardless of your prior experience with the projects adds substantial value - both locally and in DevSecOps scenarios.

## Using Habushu ##

### Adding to your Maven project
Adding habushu to your project is easy.  

#### Update Packaging ####
Change the packaging of your module's pom to use habushu:
```
	<packaging>habushu</packaging>
```

#### Add the Habuahu Plugin ####
Add the following plugin to your module's pom's build section:
```
	<plugin>
		<groupId>org.bitbucket.cpointe.habushu</groupId>
		<artifactId>habushu-maven-plugin</artifactId>
		<version>${project.version}</version>
		<extensions>true</extensions>
	</plugin>
```

#### Resulting Build Licefycle ####
After performing the steps above, your module will leverage the habushu lifecycle rather than the default lifecycle. It consists of the following stages. 

Configuration for options described below can be accomplished as described in the Test Configuration Options section below.

##### clean #####
Leverages the standard [`maven-clean-plugin`](https://maven.apache.org/plugins/maven-clean-plugin/index.html) to clear out the entire `target`directory when clean is passed to the build.  All configurations options are listed in the plugin's documentation.

##### resources #####
Habushu has extended the default [`maven-resources-plugin`](https://maven.apache.org/plugins/maven-resources-plugin/) to copy anything in `src/main/python`, `src/main/resources` into the `target/staging` directory. The project's `pom.xml` and Venv dependency file are also included. These copies can then be used for testing and will be included in the zip file produced later in the lifecycle. All configurations options are listed in the plugin's documentation.  The following configuration options can be specified via standard Maven configuration for plugins:

* _venvDependencyFile:_ The location of your Venv dependency file.  By default, this will point to `dependencies.txt` directly within the root of the module.
* _pythonSourceDirectory:_ The directory in which your source code should be placed.  By default, `src/main/python`.  It is highly discouraged to change this value.
* _resourcesDirectory:_ The directory in which your resources should be placed.  BY default, `src/main/resources`.  It is highly discouraged to change this value.

##### configure-environment #####
Create or update your Venv virtual environment. This ensures it is valid and that any tests are run in the versioned controlled environment. The following configuration options can be specified via standard Maven confguration for plugins:  

* _venvDirectory:_ The root location of your virtual environment.  By default, this will point to the /virtualenvs/ folder directly under the project build directory (usually the target folder).
* _workingDirectory:_ The location in which any venv commands will be run.  By default, this is `target`.
* _pythonSourceDirectory:_ The directory in which your source code should be placed.  By default, `src/main/python`.  It is highly discouraged to change this value.
* _environmentName:_ The name of your virtual environment.  By default, this will be the `artifactId` of your Maven build.
* _pathToVirtualEnvironment:_ The path to your specified virtual environment.  By default, this will simply add your environmentName onto the file path of your `venvDirectory`.

##### test #####
Run behave if any files exist within the `src/test/python/features` directory. More information on authoring and running tests can be found later in this document, including configuration options.

##### zip #####
Habushu has extended the [`maven-assembly-plugin`](http://maven.apache.org/plugins/maven-assembly-plugin/) to create a zip file from all the files in the `target/staging` directory. All configurations options are listed in the plugin's documentation.

##### install #####
Moves the assembly creates in the zip lifecycle to the local machine's `.m2/respository` per the standard Maven lifecycle. This includes information for versioning SNAPSHOT (development) and released versions via GAV (groupId, artifactId, version) naming conventions and allows the artifact to be pulled by GAV with any local Maven-compliant dependency fetching algorithm.  If needed, more information is available on the [`maven-install-plugin`](https://maven.apache.org/plugins/maven-install-plugin/) page.

##### deploy #####
The same as install, but deploys the artifact and associated GAV files to a server to reuse across environments via Maven-compliant dependency fetching algorithms. This will often include digital signature files to ensure the integrity of deployed files (automatic within the standard Maven process for released versions). If needed, more information is available on the [`maven-deploy-plugin`](https://maven.apache.org/plugins/maven-deploy-plugin/) page.

### Adding Tests ###
Habushu leverages [behave](https://behave.readthedocs.io/en/stable/index.html) to run Cucumber tests quiskly and easily.

#### Add Feature Files ####
You can add your [Gherkin feature files](https://cucumber.io/docs/gherkin/) to the `src/test/python/features` directory.  You can change the location of where test code is located via the `pythonTestDirectory` configuration value using [standard Maven configuration procedures](https://maven.apache.org/guides/mini/guide-configuring-plugins.html), though this is highely discouraged.
 
#### Run Build to Generate Step Stubs ####
As is customary with Cucumber testing, once you add your feature, stubbed methods can be created for each step by running the build via `mvn clean test`.  Any step that is missing will be listed at the end of the build, allowing you to copy that snippet and then add in the implementation.  For example:
```
[INFO] Failing scenarios:
[INFO]   ../../src/test/python/features/example.feature:30  This is an unimplemented test for testing purposes

You can implement step definitions for undefined steps with these snippets:

@given(u'any SSL leveraged within the System')
def step_impl(context):
    raise NotImplementedError(u'STEP: Given any SSL leveraged within the System')
```

#### Author Step Implementations ####
Following the standard behave convention, add your Step classes to the `src/test/python/features/steps` folder.  The stubbed methods from the prior step will be matched at runtime.

#### Test Configuration Options ####
Several configuration options are described below.  Please feel free to request additional configuration options via a [JIRA ticket](https://fermenter.atlassian.net/browse/HAB).

Per standard Maven conventions, these confgurations can be set in your POM or via the command line.

*POM Example*
```
<plugin>
	<groupId>org.bitbucket.cpointe.habushu</groupId>
	<artifactId>habushu-maven-plugin</artifactId>
	<version>LATEST VERSION GOES HERE</version>
	<configuration>
		<cucumber.options>--tags @justThisTag</cucumber.options>
	</configuration>
</plugin>
```

*Command Line Example*

`mvn clean install -Dcucumber.options="--tags @justThisTag"`

##### Include @manual Tagged Features/Scenarios #####
By default, any feature or scenario tagged with `@manual` will be automatically skipped.  You can disable this behavior by setting the `excludeManualTag` confugration options. No need to specify at all unless you want to change the default.

##### Specify a Behave / Cucumber Command Line Option #####
Behave supports a [number of command line options](https://behave.readthedocs.io/en/stable/behave.html#command-line-arguments).  One or more can bee added via the `cucumber.options` configuration.  Often, this is used to specify a specific tag or tags you are actively iterating on to speed that process.

##### Skip Tests Entirely #####
This is almost always a bad idea, but occasionally is very useful.  The standard Maven `skipTests` configuration can be added (without specifying `=true`, though that works too).

## Building Habushu ##
If you are working on habushu, please be aware of some nuances in working with a plugin that defines a lifecycle and packaging. We use `habushu-mixology` to immediately test our plugin. However, if the `habushu-maven-plugin` has not been previously built (or was built with critical errors), you need to manually build the `habushu-maven-plugin` first.  To assist, there are two profiles available in the build:

* `mvn clean install -Pbootstrap`: will build the `habushu-maven-plugin` so `habushu-mixology` can be executed in subsequent default builds.  Note that this also means that mixology lifecycle changes require two builds to test - one to build the lifecycle, then a second to use that updated lifecycle.  Code changes within the existing lifecycle work via normal builds without the need for a second pass.
* `mvn clean install -Pdefault`: (ACTIVE BY DEFAULT - `-Pdefault` does not need to be specified) builds all modules.

## Common Issues ##

### Plugin Does Not Yet Exist ###
Of you encounter the following error, please see the Building Habushu section for details on how to use the bootstrap profile to get around this issue.
```
[WARNING] The POM for org.bitbucket.cpointe.habushu:habushu-maven-plugin:jar:0.0.1-SNAPSHOT is missing, no dependency information available
[ERROR] [ERROR] Some problems were encountered while processing the POMs:
[ERROR] Unresolveable build extension: Plugin org.bitbucket.cpointe.habushu:habushu-maven-plugin:0.0.1-SNAPSHOT or one of its dependencies could not be resolved: Could not find artifact org.bitbucket.cpointe.habushu:habushu-maven-plugin:jar:0.0.1-SNAPSHOT @ 
[ERROR] Unknown packaging: habushu @ line 15, column 13
``` 