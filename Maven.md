# How to use Retryer with Maven #

Retryer have not published in public maven repositories, but maintains it's own small repository in it's own svn. You'll need to add following to your pom project/repositories:
```
<repository>
  <id>java-retryer.googlecode.com</id>
  <name>java-Retryer Repository for Maven</name>
  <url>http://java-retryer.googlecode.com/svn/repo/</url>
</repository>
```

and the following to dependencies:

```
<dependency>
  <groupId>org.retryer</groupId>
  <artifactId>retryer</artifactId>
  <version>0.2</version>
</dependency>
```