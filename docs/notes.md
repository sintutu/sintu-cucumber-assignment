# Notes

## 1. Starting a new project

The [10-minute-tutorial](https://cucumber.io/docs/guides/10-minute-tutorial#create-an-empty-cucumber-project (Create an empty Cucumber project)) gives a commmand to create an empty Cucumber project.

```bash
mvn archetype:generate -DarchetypeGroupId=io.cucumber -DarchetypeArtifactId=cucumber-archetype -DarchetypeVersion=7.34.3 -DgroupId=com.sintutu -DartifactId=sintu-cucumber-assignment -Dpackage=com.sintu.cucumberassignment -Dversion=1.0.0-SNAPSHOT -DinteractiveMode=false      
```

What this does is generate a new maven project using the `cucumber-archetype` (version `7.34.3`) found in the `io.cucumber` archetype group. 

This goes into my own group, called `com.sintutu`. I call this artifact `sintu-cucumber-assignment`. The package inside the assignment needs to follow the package naming rules, so I name it `com.sintu.cucumberassignment`. I create the very first version, `1.0.0-SNAPSHOT`. 

I want this to run without any further interaction.

NOTE: `-Dpackage` *should* start with the name of the group by convention. It then continues with the name of the package, SEPARATED BY `.`. It's a java thing. `com.sintutu.cucumberassignment` then defines the package and define the folder com/sintu/cucumberassignment. Package names and folder structures need to match exactly. 

> I see in hindsight I chose `sintu` instead of `sintutu` by accident.

## 2. What's in the box?

The generated artifact is a folder `sintu-cucumber-assignment/` containing the structure

```
sintu-cucumber-assignment
├── pom.xml
└── src
    └── test
        ├── java
        │   └── com
        │       └── sintu
        │           └── cucumberassignment
        │               ├── RunCucumberTest.java
        │               └── StepDefinitions.java
        └── resources
            └── com
                └── sintu
                    └── cucumberassignment
                        └── example.feature
```

To see if it works, I run `mvn test` from the shell.

This does a few maven things and eventually hits the test runner that comes out the box, junit with some cucumber skin on it. This allows Cucumber annotations to extend the junit test annotations for test discovery. So `@Given`, `@When`, and `@Then` work. Also, these annotations bind the steps and step definitions together.

See the pom.xml at starting state:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xmlns="http://maven.apache.org/POM/4.0.0"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.sintutu</groupId>
    <artifactId>sintu-cucumber-assignment</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>jar</packaging>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <maven.compiler.release>17</maven.compiler.release>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>io.cucumber</groupId>
                <artifactId>cucumber-bom</artifactId>
                <version>7.34.3</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.junit</groupId>
                <artifactId>junit-bom</artifactId>
                <version>5.14.2</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
            <dependency>
                <groupId>org.assertj</groupId>
                <artifactId>assertj-bom</artifactId>
                <version>3.27.7</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-java</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>io.cucumber</groupId>
            <artifactId>cucumber-junit-platform-engine</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.junit.platform</groupId>
            <artifactId>junit-platform-suite</artifactId>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.assertj</groupId>
            <artifactId>assertj-core</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.14.1</version>
            </plugin>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>3.5.4</version>
                <configuration>
                    <properties>
                        <!-- Work around. Surefire does not include enough
                             information to disambiguate between different
                             examples and scenarios. -->
                        <configurationParameters>
                            cucumber.junit-platform.naming-strategy=long
                        </configurationParameters>
                    </properties>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

## 3. RunCucumberTest.java

The runner that comes out of `cucumber-archetype` is pre-configured for the generated package using the junit annotations from artifact `junit-platform-suite`.

```java
package com.sintu.cucumberassignment;

import org.junit.platform.suite.api.ConfigurationParameter;
import org.junit.platform.suite.api.IncludeEngines;
import org.junit.platform.suite.api.SelectPackages;
import org.junit.platform.suite.api.Suite;

import static io.cucumber.junit.platform.engine.Constants.GLUE_PROPERTY_NAME;
import static io.cucumber.junit.platform.engine.Constants.PLUGIN_PROPERTY_NAME;

@Suite
@IncludeEngines("cucumber")
@SelectPackages("com.sintu.cucumberassignment")
@ConfigurationParameter(key = PLUGIN_PROPERTY_NAME, value = "pretty")
@ConfigurationParameter(key = GLUE_PROPERTY_NAME, value = "com.sintu.cucumberassignment")
public class RunCucumberTest {
}
```

What seems intuitive here is that:
  1. I name the package I want to run with `@SelectPackages` where the packages my tests go into are sitting, `com.sintu.cucumberassignment`.
  2. I specify the engine to use with `@IncludeEngines` where I pick `cucumber-junit-platform-engine`.
  3. I specify how to glue the features using the `@ConfigurationParameter` annotation, matching key to where `StepDefinitions` sits.

  I might want to read up more on https://github.com/cucumber/cucumber-jvm/blob/main/cucumber-junit-platform-engine/README.md.