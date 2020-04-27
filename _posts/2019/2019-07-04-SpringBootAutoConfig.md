---
layout: post
category: Spring Boot
title: Spring Boot 自动配置原理
tagline: by 城南书客
tags: 
  - Spring Boot
published: true
---
## 简介
Spring Boot 是由 Pivotal 团队提供的全新框架，其设计目的是用来简化 Spring 应用的初始搭建以及开发过程。它使用 “习惯优于配置” 的理念可以让你的项目快速搭建、部署以及独立运行。使用 Spring Boot 可以不用或者只需要很少的 Spring 配置。

Spring Boot 核心的功能就是自动配置。它会根据在类路径中的 jar、类自动配置 Bean ,当我们需要配置的 Bean 没有被 Spring Boot 提供支持时,也可以自定义自动配置，来看一下自动配置原理。

## 配置的加载原理
SpringBoot 应用启动的时候，从主方法进行启动
```
@SpringBootApplication
public class XxxApplication {
  public static void main(String[] args) {
    SpringApplication.run(XxxApplication.class, args);
  }
}
```
@SpringBootApplication 是一个组合注解，包含如下注解
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
```
@EnableAutoConfiguration 为开启自动配置，其也是组合注解，包含如下注解
```
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
```
主要功能由 @Import 提供，其导入的 AutoConfigurationImportSelector 的 selectImports 方法
```
public String[] selectImports(AnnotationMetadata annotationMetadata) {
    if (!this.isEnabled(annotationMetadata)) {
        return NO_IMPORTS;
    } else {
        AutoConfigurationMetadata autoConfigurationMetadata = AutoConfigurationMetadataLoader.loadMetadata(this.beanClassLoader);
        AutoConfigurationImportSelector.AutoConfigurationEntry autoConfigurationEntry = this.getAutoConfigurationEntry(autoConfigurationMetadata, annotationMetadata);
        return StringUtils.toStringArray(autoConfigurationEntry.getConfigurations());
    }
}
```
调用 getAutoConfigurationEntry 方法
```
protected AutoConfigurationImportSelector.AutoConfigurationEntry getAutoConfigurationEntry(AutoConfigurationMetadata autoConfigurationMetadata, AnnotationMetadata annotationMetadata) {
  if (!this.isEnabled(annotationMetadata)) {
    return EMPTY_ENTRY;
  } else {
    AnnotationAttributes attributes = this.getAttributes(annotationMetadata);
    List<String> configurations = this.getCandidateConfigurations(annotationMetadata, attributes);
    configurations = this.removeDuplicates(configurations);
    Set<String> exclusions = this.getExclusions(annotationMetadata, attributes);
    this.checkExcludedClasses(configurations, exclusions);
    configurations.removeAll(exclusions);
    configurations = this.filter(configurations, autoConfigurationMetadata);
    this.fireAutoConfigurationImportEvents(configurations, exclusions);
    return new AutoConfigurationImportSelector.AutoConfigurationEntry(configurations, exclusions);
  }
}
```
再调用 getCandidateConfigurations 方法
```
protected List<String> getCandidateConfigurations(AnnotationMetadata metadata, AnnotationAttributes attributes) {
  List<String> configurations = SpringFactoriesLoader.loadFactoryNames(this.getSpringFactoriesLoaderFactoryClass(), this.getBeanClassLoader());
  Assert.notEmpty(configurations, "No auto configuration classes found in META-INF/spring.factories. If you are using a custom packaging, make sure that file is correct.");
  return configurations;
}
```
通过 loadFactoryNames 方法扫描 spring.factories 文件中包含的 JAR 文件，加载到 Spring 容器。
```
public static AutoConfigurationMetadata loadMetadata(ClassLoader classLoader) {
    return loadMetadata(classLoader, "META-INF/spring-autoconfigure-metadata.properties");
}
```
## 配置的生效原理
这些自动配置类都是在某些条件下生效
```
@ConditionalOnBean：当容器里有指定的bean的条件下
@ConditionalOnMissingBean：当容器里不存在指定bean的条件下
@ConditionalOnClass：当类路径下有指定类的条件下
@ConditionalOnMissingClass：当类路径下不存在指定类的条件下
@ConditionalOnProperty：指定的属性是否有指定的值
```