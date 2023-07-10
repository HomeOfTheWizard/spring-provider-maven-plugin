﻿# spring-provider-maven-plugin
A plugin that helps other plugins to use spring libraries.  
Maven uses [Sisu](https://eclipse.github.io/sisu.inject/) as dependency injection mechanism and this plugin allows you to have a bridge between the two.  
You just add this plugin in your project, configure it, and you will be able to simply inject your beans in your mojo using `@Inject` and start using them.   
  
# how it works ?
You need to add this plugin in the plugin section of your initial plugin's pom.xml like so:  
```
<dependencies>
    <dependency>
        <groupId>com.homeofthewizard</groupId>
        <artifactId>provider-generator-maven-plugin</artifactId>
        <version>1.0-SNAPSHOT</version>
    </dependency>
</dependencies>

...

<build>
    <plugins>
        <plugin>
            <groupId>com.homeofthewizard</groupId>
            <artifactId>spring-provider-maven-plugin</artifactId>
            <version>1.0-SNAPSHOT</version>
            <dependencies>
                <dependency>
                    <groupId>org.springframework.somelib</groupId>
                    <artifactId>spring-some-lib</artifactId>
                    <version>x.x.x</version>
                </dependency>
            </dependencies>
            <executions>
                <execution>
                    <id>generate</id>
                    <goals>
                        <goal>generate</goal>
                    </goals>
                </execution>
                <execution>
                    <id>clean</id>
                    <goals>
                        <goal>clean</goal>
                    </goals>
                </execution>
            </executions>
            <configuration>
                <generatedProvidersSourcePath>${project.build.sourceDirectory}</generatedProvidersSourcePath>
                <contextConfigClasses>
                    <contextConfigClass>org.springframework.somelib.config.SomeBeansConfiguration</contextConfigClass>
                </contextConfigClasses>
                <applicationPropertiesFile>src\main\resources\application.properties</applicationPropertiesFile>
            </configuration>
        </plugin>
    </plugins>
</build>
```
  
⚠️ First pay attention to the fact that we have to provide the plugin in both `dependencies` and `plugin` sections.  
This is because your plugin will also needs some dependencies to run spring, and this plugin has them for you.  
  
The plugin section allows this project to generate code for your, that will be picked up by maven injection mechanism.  
So you will have to build your initial plugin so that sisu pickes up the generated classes in its injection mechanism.  
  
See that in the plugin section, we provide the library that we want to use as a dependency to this project.  
Then in the `<configuration>` section of the plugin we tell what to do with this library (spring configuration class, and application properties to use, ect..).
This is an example provides a full list of configuration parameters.  
Some of them have default values, so you can omit them.  
`generatedProvidersSourcePath` and `applicationPropertiesFile` are optional if you want to use the values given in the example above.  
⚠️ Be careful where you want this plugin to generate the source files because sisu will need to be able to scan them to add them in the injection mechanism.  
see next section for more details.

# Under the hood
This plugin allows you to use JSR330 annotations to inject your spring beans.
JSR330 is made available in maven with the arrival of sisu.  
  
Maven uses Sisu since version 3.1.0, so this plugin works from Maven 3.1.x and on. It requires Java 11.  
See this [doc](https://maven.apache.org/maven-jsr330.html) for mo﻿re details on how to use JSR330 in maven plugins and about maven's history of DI mechanism. 

The plugin allows injecting the beans via generating code of JSR330 providers.  
The plugin is configurer to run during the `sources` phase of the maven execution,  
so that the sources will be compiled later on the `compile` phase by the maven compile plugin,  
and then the classes will be scanned and indexed by the sisu maven plugin.  
See again [here](https://maven.apache.org/maven-jsr330.html) for the sisu plugin usage

Even though sisu is based on Guice and in guice API we can create custom bindings, maven unforthunately does not expose the Api to plugins. The documentation is unfortunately [wrong].(https://github.com/eclipse/sisu.plexus/issues/35)  
So we have two options to bypass this, one is this plugin, and the second is to build an extension, like demonstrated [here](https://github.com/scratches/plugin-demo)
