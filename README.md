[ ![Gradle Plugin Portal](https://img.shields.io/gradle-plugin-portal/v/edu.sc.seis.launch4j)](https://plugins.gradle.org/plugin/edu.sc.seis.launch4j)

**Build status**:
[ ![Build status master](https://ci.appveyor.com/api/projects/status/xscd7594tneg721r/branch/master?svg=true&passingText=master%20-%20OK&failingText=master%20-%20Fails&pendingText=master%20-%20pending)](https://ci.appveyor.com/project/TheBoegl/gradle-launch4j/branch/master)
[ ![Build status develop](https://ci.appveyor.com/api/projects/status/xscd7594tneg721r/branch/develop?svg=true&passingText=develop%20-%20OK&failingText=develop%20-%20Fails&pendingText=develop%20-%20pending)](https://ci.appveyor.com/project/TheBoegl/gradle-launch4j/branch/develop)

**Table of contents**
* [Introduction](#introduction)
* [Tasks](#tasks)
* [Configuration](#configuration)
* [Launch4jLibraryTask](#launch4jlibrarytask)
* [Launch4jExternalTask](#launch4jexternaltask)
* [Kotlin](#kotlin)
* [Contributors](#contributors) (see [Contributors](graphs/contributors))
* [Version](#version) (see [VERSION.md](VERSION.md))

# Introduction

The gradle-launch4j plugin uses [launch4j](http://launch4j.sourceforge.net/) [3.50](src/main/groovy/edu/sc/seis/launch4j/Launch4jPlugin.groovy) to create windows .exe files for java applications.
This plugin is compatible with the Gradle versions 4.10 and later.
If you still rely on an outdated gradle version `[2-4.10[`, use the plugin version [2.5.4](https://github.com/TheBoegl/gradle-launch4j/tree/v2.5.4).

Since **version 2.5** this plugin requires **Java 8*, as launch4j in version 3.14 and later requires that as well.
If you are still forced to work with Java 6, use the plugin version [2.4.9](https://github.com/TheBoegl/gradle-launch4j/tree/v2.4.9).

# Tasks

There are 2 tasks:

* **createExe** - Backward compatible task to generate an .exe file. *Execute this task to generate an executable.* With default settings this creates the executable under `${project.buildDir}/launch4j` and puts all runtime libraries into the lib subfolder. 
* createAllExecutables - Helper task to run all tasks of the `Launch4jExternalTask` and `Launch4jLibraryTask` type.

Launch4j no longer needs to be installed separately, but if you want, you can still use it from the *PATH*. Since version 2.0 use the [Launch4jExternalTask](#launch4jexternaltask) to create your executable.

# Configuration

The configuration follows the structure of the launch4j xml file. The gradle-launch4j plugin tries to pick sensible defaults based on the project. The only required
value is the `mainClassName`.

## How to include
An example configuration within your `build.gradle` for use in all Gradle versions might look like:

    plugins {
      id 'java'
      id 'edu.sc.seis.launch4j' version '3.0.5'
    }

    launch4j {
      mainClassName = 'com.example.myapp.Start'
      icon = "${projectDir}/icons/myApp.ico"
    }

The same script snippet for using [legacy plugin application](https://docs.gradle.org/current/userguide/plugins.html#sec:old_plugin_application):

    buildscript {
      repositories {
        gradlePluginPortal()
      }
      dependencies {
        classpath 'edu.sc.seis.launch4j:launch4j:3.0.5'
      }
    }

    repositories {
      mavenCentral()
    }

    apply plugin: 'java'
    apply plugin: 'edu.sc.seis.launch4j'

    launch4j {
      mainClassName = 'com.example.myapp.Start'
      icon = "${projectDir}/icons/myApp.ico"
    }

If no repository is configured before applying this plugin the *Maven central* repository will be added to the project.

See the [Gradle User guide](http://gradle.org/docs/current/userguide/custom_plugins.html#customPluginStandalone) for more information on how to use a custom plugin and the [plugin page](https://plugins.gradle.org/plugin/edu.sc.seis.launch4j) for the above settings.

## How to configure

The values configurable within the launch4j extension along with their defaults are:

| Property Name | Default Value | Comment |
|---------------|---------------|---------|
| String outputDir | "launch4j" | This is the plugin's working path relative to `$buildDir`. Use the distribution plugin or a custom implementation to copy necessary files to an output location instead of adjusting this property.|
| String libraryDir | "lib" | The library directory next to the created executable, where all dependencies, i.e. libraries, will be copied into. |
| Object copyConfigurable | null | User-defined set of files to copy to`libraryDir` (if not set, the default copy logic is used) |
| Set&lt;String&gt; classpath| [] | User-defined classpath property (if not set, the default logic based on the set of files copied to `libraryDir` is used) |
| String xmlFileName | "launch4j.xml" | |
| String mainClassName | | |
| boolean dontWrapJar | false | |
| String headerType | "gui" | |
| Task jarTask | tasks[jar], if the JavaPlugin is loaded | The jar producing task. See [here](#configurable-input-configuration) how to use this for the shadow plugin. |
| String outfile | project.name+'.exe' | |
| String errTitle | "" | |
| String cmdLine | "" | |
| String chdir | '.' | |
| String priority | 'normal' | |
| String downloadUrl | "http://java.com/download" | |
| String supportUrl | "" | |
| boolean stayAlive | false | |
| boolean restartOnCrash | false | |
| String manifest | "" | |
| String icon | "" | A relative path from the outfile or an absolute path to the icon file. If you are unsure, use "${projectDir}/path/to/icon.ico" |
| String version | project.version | |
| String textVersion | project.version | |
| String copyright | "unknown" | |
| String companyName | "" | |
| String fileDescription | project.name | |
| String productName | project.name | |
| String internalName | project.name | |
| String trademarks | | |
| String language | "ENGLISH_US" | |
| Set&lt;String&gt; jvmOptions | [ ] | |
| String bundledJrePath | | |
| boolean requires64Bit | false | |
| String jreMinVersion | project.targetCompatibility or<br> the current java version,<br> if the property is not set | |
| String jreMaxVersion | | |
| Set&lt;String&gt; variables | [ ] | |
| String mutexName | | |
| String windowTitle | | |
| String messagesStartupError | | |
| String messagesJreNotFoundError | | |
| String messagesJreVersionError | | |
| String messagesLauncherError | | |
| String messagesInstanceAlreadyExists | | |
| Integer initialHeapSize | | |
| Integer initialHeapPercent | | |
| Integer maxHeapSize | | |
| Integer maxHeapPercent | | |
| String splashFileName | | A relative path from the outfile or an absolute path to the bmp splash file. |
| boolean splashWaitForWindows | true | |
| Integer splashTimeout | 60 | |
| boolean splashTimeoutError | true | |
| DuplicatesStrategy duplicatesStrategy | DuplicatesStrategy.EXCLUDE | The duplication strategy to use when duplicates are found. See also [here](https://docs.gradle.org/current/javadoc/org/gradle/api/file/DuplicatesStrategy.html).|

| Removed properties                 | Default Value | Description                          |
|------------------------------------|---------------|--------------------------------------|
| ~~boolean bundledJreAsFallback~~   | false         |                                      |
| ~~boolean bundledJre64Bit~~        | false         | use requires64Bit instead            |"
| ~~jar~~                            | "lib/"+project.tasks[jar].archiveName<br/>or<br/>"", if the JavaPlugin is not loaded |    |
| ~~String jdkPreference~~           | "preferJre"   | use requiresJdk instead              |
| ~~String jreRuntimeBits~~          | "64/32"       | use requires64Bit instead            |
| ~~String messagesBundledJreError~~ |               | use messagesJreNotFoundError instead |

### Configurable input configuration

In order to configure the input of the *copyL4jLib* task set the `copyConfigurable` property.
The following example shows how to use this plugin hand in hand with the shadow plugin:

    launch4j {
        outfile = 'TestMain.exe'
        mainClassName = project.mainClassName
        copyConfigurable = []
        jarTask = project.tasks.shadowJar
    }

If you use the outdated fatJar plugin the following configuration correctly wires the execution graph:

    fatJar {
        classifier 'fat'
        with jar
        manifest {
            attributes 'Main-Class': project.mainClassName
        }
    }
    
    fatJarPrepareFiles.dependsOn jar
    
    launch4j {
        outfile = 'TestMain.exe'
        mainClassName = project.mainClassName
        copyConfigurable = []
        jarTask = project.tasks.fatJar
    }

# Launch4jLibraryTask
This task type can be used to build multiple executables with Launch4j. The default launch4j configuration from [how to configure](#how-to-configure) is used for the default values but can be adjusted.
To avoid replacing the resulting xml file or executable on each invocation, `xmlFileName` and `outfile` are set to the task name (`name.xml` and `name.exe` respectively).

Creating three executables is as easy as:

    launch4j {
        outfile = 'MyApp.exe'
        mainClassName = 'com.example.myapp.Start'
        icon = "${projectDir}/icons/myApp.ico"
        productName = 'My App'
    }
    
    tasks.register('createFastStart', edu.sc.seis.launch4j.tasks.Launch4jLibraryTask) {
        outfile = 'FastMyApp.exe'
        mainClassName = 'com.example.myapp.FastStart'
        icon = "${projectDir}/icons/myAppFast.ico"
        fileDescription = 'The lightning fast implementation'
    }
    
    tasks.register('MyApp-memory', edu.sc.seis.launch4j.tasks.Launch4jLibraryTask) {
        fileDescription = 'The default implementation with increased heap size'
        maxHeapPercent = 50
    }

Running the `createAllExecutables` task will create the following executables in the launch4j folder located in the buildDir:
* MyApp.exe
* FastMyApp.exe
* MyApp-memory.exe

# Launch4jExternalTask
The [section from above](#launch4jlibrarytask) applies to this task, too.
This task type has the following additional property:

* String launch4jCmd = "launch4j"

In order to use a launch4j instance named 'launch4j-test' located in the PATH create a task like the following:

    launch4j {
        mainClassName = 'com.example.myapp.Start'
    }
    
    task createMyApp(type: edu.sc.seis.launch4j.tasks.Launch4jExternalTask) {
        launch4jCmd = 'launch4j-test'
        outfile = 'MyApp.exe'
    }
# Using another launch4j binary
To use a different launch4j binary instead of the default one, you can provide it as follows. Use your desired version instead of 3.50.

    dependencies {
        launch4jBin 'net.sf.launch4j:launch4j:3.50:workdir-win32'
    }
# Kotlin

To get started using this plugin from a kotlin build script the above example from [the section Launch4jLibraryTask](#launch4jlibrarytask) would be written as:
```kotlin

tasks.withType<edu.sc.seis.launch4j.tasks.DefaultLaunch4jTask> {
    outfile.set("${appName}.exe")
    mainClassName.set(mainClass)
    icon.set("$projectDir/icons/myApp.ico")
    productName.set("My App")
}

tasks.register<edu.sc.seis.launch4j.tasks.Launch4jLibraryTask>("createFastStart") {
    outfile.set("FastMyApp.exe")
    mainClassName.set("com.example.myapp.FastStart")
    icon.set("$projectDir/icons/myAppFast.ico")
    fileDescription.set("The lightning fast implementation")
}
tasks.register<edu.sc.seis.launch4j.tasks.Launch4jLibraryTask>("MyApp-memory") {
    fileDescription.set("The default implementation with increased heap size")
    maxHeapPercent.set(50)
}
```

# Debugging
To get insight into the launch4j creation process start a launch4j task, e.g. `createExe`, `createExecutables` or your custom task, with the script parameter `-Pl4j-debug`. This will copy the created xml into `${buildDir}/tmp/${task.name}`.

In order to debug the created executable call it with the command line argument `--l4j-debug`. This will create the log file `launch4j.log` next to the executable.

# Using `SNAPSHOT` versions

When you report a bug and it got fixed, you will have access to some `-SNAPSHOT` version.
Adjust your buildscript to use the artifactory OSS repo:
```gradle
buildscript {
  repositories {
    maven { url 'https://boegl.jfrog.io/artifactory/snapshots-gradle-dev-local/' }
  }
  dependencies {
    classpath 'edu.sc.seis.launch4j:launch4j:latest.integration'
  }
}

apply plugin: 'edu.sc.seis.launch4j'
```

# Contributors

* [Sebastian Bögl](https://github.com/TheBoegl) (Maintainer)
* [Philip Crotwell](https://github.com/crotwell) (Creator)

See [contributors](graphs/contributors) for a complete list. 

# Version

See [VERSION.md](VERSION.md) for more information.
