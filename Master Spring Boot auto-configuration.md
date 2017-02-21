# Master Spring Boot auto-configuration
## 1. Auto-configuration class
+ Regular `@Configuration` class
+ Referenced in `META-INF/spring.factories`
+ Enabled (Guarded) via certain conditions
+ Can be triggered before or after another auto-configuration
  * @AutoConfigureBefore
  * @AutoConfigureAfter
+ Displayed in the auto-configuration report
 * Logs when debug is enabled (·—debug·)
 * ·/autoconfig· on a web app with the actuator
---
## 2. Spring 4 @Conditional
```java
@Configuration
@Conditional(CustomCondition.class)
public class AppConfiguration {
// @Bean methods
}

public class CustomCondition implements Condition {
  public boolean matches(ConditionContext context,
      AnnotatedTypeMetadata metadata) {
  ...
  }
}
```
---
## 3. Core conditions
+ @ConditionalOn[Missing]Class
+ @ConditionalOn[Missing]Bean
+ @ConditionalOnProperty
+ @ConditionalOnResource
+ @ConditionalOnExpression (SpEL)
+ @ConditionalOn[Not]WebApplication
+ @ConditionalOnJava - @ConditionalOnJndi
+ AnyNestedCondition
+ [your condition here]
---
## 4. Let’s build our own auto-config+
+ `xyz-spring-boot-autoconfigure`
  * Auto-configuration classes and related code
+ `xyz-spring-boot-starter`
 * Necessary dependencies to use feature xyz
+ Can be combined in most cases
+ Example based on HornetQ
 * Actually implemented in Spring Boot
 * Real world example with interesting configuration options
---
## 5. @ConfigurationProperties
+ Complete infrastructure to customize your auto-configuration
+ Just add ·@EnableConfigurationProperties(MyProperties.class)·
  * Add that class as a bean in the context (i.e. you can inject it)
  * Inject any key in the Environment that match the prefix + property name
    * Works with system property, command-line switch, key in configuration file, etc.
+ Add the annotation processor to your build
  * Generate the relevant meta-data so that the IDE can pick that up
  * Description (field javadoc), default value, type
+ Make that an isolated (configuration-only) thing
---
## 6. Configuration to the rescue. Again.
+  An auto-configuration can work in several ways (modes)
  *  Ideally extra conditions should discriminate them
  *  And a configuration key should offer a way to ‘force’ the mode explicitly
+  Not always a sunny day
  *  Achieving a good result for the end user may mean low-level checks
  *  Usually more practical to custom conditions
---
## 7. Advanced customizations
+ Try to put you in the shoes of the user
+ ·@ConditionalOnMissingBean· to replace auto-configuration partially
  * JMSConfiguration
  * EmbeddedJMS
+ Callback interface to customize things
  * HornetQConfigurationCustomizer
---
## 8. Your own Condition
+ Many use cases already covered by Spring Boot
+ Reuse `SpringBootCondition` base class
  *  Provide sensible logging
  *  Consistent error management
+ Implement ConfigurationCondition to tune the registration phase
  *  PARSE_CONFIGURATION: the whole @Configuration class isn’t processed if the
  condition does not match
  *  REGISTER_BEAN: evaluated when the bean factory has more information
+ Specify order wisely so that cheap conditions are evaluated first
---
## 9. CLI customizations
```java
package org.test

@Grab('hornetq-jms-client')
@EnableJms
@Log
class ThisWillActuallyRun {

  @JmsListener(destination = 'testQueue')
  def processMsg(String msg) {
    log.info("Received $msg")
  }

}
```
---
## 10. CLI customizations
+ Parse your Groovy script
+ Grab extra dependencies
+ Auto-magically add imports
 *  Extend from CompilerAutoConfiguration
+ `matches(ClassNode node)` determines if the customisation should apply
+ 'applyDependencies(DependencyCustomizer customizer)' add extra
dependencies
+ `applyImports(ImportCustomizer customizer)` add extra imports

## 11. Wrapping Up
+  Implementing your own auto-configuration is easy
  *  Dealing with all the particularities of what you want to configure is the hard part
+  All the infrastructure that Boot uses internally is available
  *  We have a good range of “core” conditions already
+  Creating custom conditions is cheap
  *  Prefer that to über complex SpEL expressions
+  Use configuration keys to expose configuration options
  *  And make sure to generate the relevant meta-data for IDEs support
+  (More advanced) use callback interface for fine-grained customizations
