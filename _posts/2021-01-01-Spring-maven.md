---
title: Spring Maven
author: Jiny
date: 2021-01-01 11:13:00 +0800
categories: [Java, Spring]
tags: [java, maven]
toc: false
---

# Maven 설정
___

## 자바 소스 설정

```java
<build>
    <sourceDirectory>src/main/java</sourceDirectory>
</build>
```

## 자바소스 여러 개  설정

```xml
<plugin>
    <groupId>org.codehaus.mojo</groupId>
    <artifactId>build-helper-maven-plugin</artifactId>
    <version>1.2</version>
    <executions>
        <execution>
            <id>add-source-dir</id>
            <phase>generate-sources</phase>
            <goals>
                <goal>add-source</goal>
            </goals>
            <configuration>
                <sources>
                    <source>another/src/main/java</source>
                    <source>others/src</source>
                </sources>
            </configuration>
        </execution>
    </executions>
</plugin>
```

## 저장소 추가

```xml
 <repositories>
    <repository>
        <id>저장소 ID</id>
        <url>저장소 URL</url>
    </repository>
</repositories> 
```

- 개인적으로만 저장소를 쓴다면 .m2\settings.xml에서
profiles > profile > repositories 밑에 repository 설정을 넣으면 됨.

## 컴파일러 소스 및 타겟 버전 설정, UTF-8 인코딩 지정 방법

```xml
<build>
    <plugins>
        <plugin>
            <artifactId>maven-compiler-plugin</artifactId>
            <version>2.0.2</version>
            <configuration>
                <source>1.6</source>
                <target>1.6</target>
                <encoding>UTF-8</encoding>
            </configuration>
        </plugin>
    </plugins>
</build> 
```

## 클래스패스 설정과 -jar 옵션으로 시작할 때 사용할 메인 클래스 지정

```xml
	<build>
		<plugins>
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-jar-plugin</artifactId>
                <version>2.2</version>
                <configuration>
                    <archive>
                        <manifest>
                            <addClasspath>true</addClasspath>
                        </manifest>
                        <manifestEntries>
                            <Main-Class>org.krakenapps.main.Kraken</Main-Class>
                        </manifestEntries>
                    </archive>
                </configuration>
            </plugin>
        </plugins>
	</build>
```

## 의존하는 라이브러리를 포함하여 하나의 JAR 파일로 패키징

```xml
<build>
        <plugins>
            <plugin>
                <artifactId>maven-assembly-plugin</artifactId>
                <configuration>
                    <archive>
                        <manifest>
                            <Main-Class>org.krakenapps.main.Kraken</Main-Class>
                        </manifest>
                    </archive>
                    <descriptorRefs>
                        <descriptorRef>jar-with-dependencies</descriptorRef>
                    </descriptorRefs>
                </configuration>
            </plugin>
        </plugins>
    </build>
 
<!--//-->
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
       <artifactId>maven-war-plugin</artifactId>
       <configuration>
        <warSourceDirectory>src/main/webapp</warSourceDirectory>    
          <!--<warSourceExcludes>common/**</warSourceExcludes> warSourceExcludes는 warSourceDirectory를 기준 특정 폴더 제외-->
       </configuration>
</plugin> 
```

## 기본 디렉토리 변경
```xml
<!-- 변경전 --> 
<?xml version="1.0" encoding="UTF-8"?>
<project-modules id="moduleCoreId" project-version="1.5.0">
    <wb-module deploy-name="questionbox">
        <wb-resource deploy-path="/" source-path="/src/main/webapp" />
        <wb-resource deploy-path="/WEB-INF/classes" source-path="/src/main/resources" />
        <wb-resource deploy-path="/WEB-INF/classes" source-path="/src/main/java" />
        <wb-resource deploy-path="/WEB-INF/classes" source-path="/src/test/java" />
        <property name="context-root" value="questionbox" />
        <property name="java-output-path" />
    </wb-module>
</project-modules>
```

```xml
<!-- 변경후 -->
<?xml version="1.0" encoding="UTF-8"?>
<project-modules id="moduleCoreId" project-version="1.5.0">
    <wb-module deploy-name="questionbox">
        <wb-resource deploy-path="/" source-path="webapp" />
        <wb-resource deploy-path="/WEB-INF/classes" source-path="/src" />
        <property name="context-root" value="questionbox" />
        <property name="java-output-path" />
    </wb-module>
</project-modules>
```

## 메이븐 /src/main/java에 java 외에 파일도 target/classes로 이동

**레포팅 관련**
```xml
<reporting>
	<plugins>    
	<!-- FindBugs 리포트 생성 플러그인 -->
    	<plugin>
        	<groupId>org.codehaus.mojo</groupId>
            <artifactId>findbugs-maven-plugin</artifactId>
            <version>2.1</version>
            <configuration>
            	<forceEncoding>UTF-8</forceEncoding>
                <findbugsXmlOutput>true</findbugsXmlOutput>
                <findbugsXmlWithMessages>true</findbugsXmlWithMessages>
                    <xmlOutput>true</xmlOutput>
                    <xmlOutputDirectory>${basedir}/target/site</xmlOutputDirectory>
            </configuration>
		</plugin>
     	<plugin>
        	<groupId>org.codehaus.mojo</groupId>
            <artifactId>javancss-maven-plugin</artifactId>
            <version>2.0</version>
            <configuration>
            	<forceEncoding>UTF-8</forceEncoding>
                <lineThreshold>30</lineThreshold>
                <xmlOutputDirectory>${basedir}/target/site</xmlOutputDirectory>
                <failOnViolation>true</failOnViolation>
                <ccnLimit>10</ccnLimit>
                <ncssLimit>100</ncssLimit>
            </configuration>
        </plugin>
        <plugin>
        	<groupId>org.codehaus.mojo</groupId>
            <artifactId>jdepend-maven-plugin</artifactId>
            <version>2.0-beta-2</version>
        </plugin>
		<!-- PMD 리포트 생성 플러그인 -->
        <plugin>
    		<artifactId>maven-pmd-plugin</artifactId>
        	<configuration>
            	<rulesets>
                	<ruleset>/rulesets/basic.xml</ruleset>
                 	<ruleset>/rulesets/unusedcode.xml</ruleset>
             	</rulesets>
             	<sourceEncoding>UTF-8</sourceEncoding>
             	<targetJdk>1.5</targetJdk>
        	</configuration>
   		</plugin>
   		<plugin>
        	<groupId>org.codehaus.mojo</groupId>
            <artifactId>cobertura-maven-plugin</artifactId>
            <version>2.3</version>
    	</plugin>
	</plugins>
</reporting>
```

## 	maven-jar-plugin 으로 배포시 특정파일 제외하기

```xml
<build>
	<plugins>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-jar-plugin</artifactId>
			<version>2.3.1</version>
			<configuration>
				<excludes>
					<exclude>*.properties</exclude>
				</excludes>
			</configuration>
		</plugin>
	</plugins>
</build>
```

## source도 depoly하기

```xml
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>2.1.2</version>
            <executions>
                <execution>
                    <id>attach-sources</id>
                    <goals>
                        <goal>jar</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

## 의존관계이 있는 jar파일을 특정 폴더로 복사

> 메이븐에서는 war이 아닌 jar 패키징일 경우 의존하는 라이브러리는 함께 패키징 되지 않는다.

> 이럴 경우 maven-dependency-plugin을 사용하여 의존 관계에 있는 jar 파일을 복사해준다.

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-dependency-plugin</artifactId>
    <executions>
        <execution>
            <id>copy-dependencies</id>
            <phase>package</phase>
            <goals>
                <goal>copy-dependencies</goal>
            </goals>
            <configuration>
                <outputDirectory>${project.build.directory}/lib</outputDirectory>
                <overWriteIfNewer>true</overWriteIfNewer>
            </configuration>
        </execution>
    </executions>
</plugin>
```

## MANIFEST.MF 파일을 만들때 사용

> MANIFEST.MF 파일을 만들고 싶을 때 사용하는 플러그으로 jar 실행 파일을 만들고 싶다면 간단하게 maven-jar-plugin으로 만들수가 있다. 

> 현재 경로에서의 lib 폴더의 jar파일들을 classpath로 추가 시키며, test.Main 클래스는 test 패키지의 Main 클래스 이다.

- 실행은 java -jar jar명.jar 을 할 수 있다 

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-jar-plugin</artifactId>
    <configuration>
        <archive>
            <manifest>
                <addClasspath>true</addClasspath>
                <classpathPrefix>lib/</classpathPrefix>
                <mainClass>test.Main</mainClass>
            </manifest>
        </archive>
    </configuration>
</plugin>
```

## 특정 경로를 리소스를 지정된 경로로 복사

```xml
<plugin>
    <executions>
        <execution>
            <id>copy-resources</id>
            <phase>validate</phase>
            <goals>
                <goal>copy-resources</goal>
            </goals>
            <configuration>
                <outputDirectory>${project.basedir}/target/classes/resources</outputDirectory>
                <resources>
                    <resource>
                        <directory>${project.basedir}/resources</directory>
                        <filtering>false</filtering>
                        <includes>
                            <include>map/**</include>
                            <include>properties/**</include>
                        </includes>
                    </resource>
                </resources>
            </configuration>
        </execution>
    </executions>
</plugin>
```