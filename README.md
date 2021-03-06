gradle-override-plugin
==============

Plugin for overriding properties in a project from the command line. The main use case for this functionality is to avoid
the introduction of "magic" system/project properties for providing user-driven property values as part of build logic.
Property names are provided as String and are looked up by dot notation. If no matching property can be resolved, the build fails.

## The world without the plugin

A typical scenario without the plugin looks as such:

    extensions.create('example', MyExtension)

    class MyExtension {
        String myProp
        Integer otherProp
        Boolean flag
    }

    example {
        myProp = System.getProperty('my.prop') ?: 'hello'
        otherProp = System.getProperty('other.prop') ? System.getProperty('other.prop').toInteger() : 1
        flag = System.getProperty('flag') ? System.getProperty('other.prop').toBoolean() : true
    }

In this scenario, the build script user can override the default value assigned to the extension property `myProp` by
providing the system property `my.prop` on the command line. This works. However, the build script author has to implement
this functionality for every property that is supposed to be overriden from the command line. With an increasing amount of
code this becomes very tedious. Furthermore, the build script user has to know the name of system property.

## Usage

### Applying the Plugin

To include, add the following to your build.gradle

    buildscript {
        repositories { jcenter() }

        dependencies {
            classpath 'com.netflix.nebula:gradle-override-plugin:1.12.+'
        }
    }

    apply plugin: 'nebula-override'

### Overriding properties with the plugin

Overriding properties is not limited to specific Gradle domain objects. You can override all writeable properties. Some
examples:

* Project properties e.g. `buildDir`
* Task properties e.g. `javadoc.maxMemory`
* Extension properties on any nested level

You can override properties through the following methods:

* By providing an environment variable with the prefix `OVERRIDE_`.
* By providing a system property with the prefix `override.`.

Override properties provided through system properties take precedence over the ones provided by environment variables.

Let's consider the following, modified build script code:

    apply plugin: 'nebula-override'

    class MyExtension {
        String myProp
    }

    example {
        myProp = 'hello'
    }

The plugin defers overriding existing properties until the project is parsed during the configuration phase to allow for
changing values of all properties. Property values are automatically converted to the correct type. From the command line
the build script user can override the default value `hello` of myProp by providing `-Doverride.example.myProp=newValue`.

### Future improvements

* Introduce a plugin-specific argument that doesn't require a "prefix" similar to project properties e.g.
`-Nexample.myProp=newValue`. This will require a change in Gradle core as the functionality to add new command line handlers
(see `AbstractPropertiesCommandLineConverter`) is not exposed.
