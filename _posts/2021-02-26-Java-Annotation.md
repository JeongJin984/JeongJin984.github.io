---
title: Java Annotation
author: Jiny
date: 2021-01-22 22:40:00 +0800
categories: [Java, Jasic]
tags: [java, basic]
toc: false
---

# Java Annotation
___

## 🔘 Annotation

1. 어노테이션의 역할도 주석과 크게 다르지 않는다.
2. 일반주석과 큰 차이점은 코드를 작성할 수 있다는 것이 다르다.
3. 코드를 작성할 수 있다는 뜻은 어노테이션으로 뭔가를 할 수 있다는 뜻이 된다.
4. 어노테이션도 enum과 마찬가지로 1.5에 등장했다고 한다.

___
## 🔘 Annotation 종류

1. @Retention
   - 자바 컴파일러가 어노테이션을 다루는 방법을 기술하며, 특정 시점까지 영향을 미치는지를 결정
   - RetentionPolicy.SOURCE: 컴파일 전까지만 유효(컴파일시 사라짐)
   - RetentionPolicy.CLASS: 컴파일러가 클래스를 참조할 때까지 유효(바이트코드에는 존재)
   - RetentionPolicy.RUNTIME: 컴파일 이휴에도 JVM에 의해 계속 참조가 가능(리플렉션 사용 가능)
2. Target
   - 어노테이션이 적용될 위치를 선택
   - ElementType.PACKAGE: 패키지에 선언
   - ElementType.TYPE: 타입 선언
   - ElementType.ANNOTAION_TYPE: 어노테이션 타입 선언
   - ElementType.CONSTRUCTOR: 생성자 선언
   - ElementType.FIELD: 멤버 변수 선언
   - ElementType.LOCAL_VARIBALE: 지역변수 선언
   - ElementType.METHOD: 메서드 선언
   - ElementType.PARAMETER: 전달인자 선언
   - ElementType.TYPE_PARAMETER: 전달인자 타입 선언
   - ElementType.TYPE_USE: 타입 선언
3. Documented
   - 해당 Annotation을 JavaDoc에 포함시킵니다.
4.  Inherited
   - Annotaion 상속을 가능하게 합니다.
5. @Repeatable
   - Java8 부터 지원하며 연속적으로 어노테이션을 선언할 수 있게 해줍니다.

___
## 🔘 Annotation 정의하기

✔️ **정수 값 주입하기**

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface InsertIntData {
    int data() default 0;
}
```

```java
public class AnnotationExam01 {
    @InsertIntData(data = 30)
    private int myAge;

    @InsertIntData
    private int defaultAge;

    public AnnotationExam01() {
        this.myAge = -1;
        this.defaultAge = -1;
    }

    public int getMyAge() {
        return myAge;
    }

    public int getDefaultAge() {
        return defaultAge;
    }
}
```

- myAge에 어노테이션에서는 30으로 값을 주입합니다.
- defaultAge 에서는 값이 없는데 이 경우 어노테이션에서 정한 기본 값인 0으로 값이 주입이 됩니다.
- 생성자의 경우 값이 없을 경우 -1을 기본으로 저장합니다.

___

## 🔘 Annotation Processor

> Annotation Processor를 이용하여 컴파일 단계에서 소스를 조작

- Annotation Processor가 컴파일 단계에서 모든 소스를 조작할 수 있는 것은 아니며, 새로 작성되는 파일에만 소스를 조작할 수 있고 기존에 있던 파일들은 건드릴 수 없다고 한다.
- Lombok은 된다고 한다.

___

## 🔘 Annotation Processsor 사용

모든 프로세서들은 AbstractProcessor를 상속받아야 한다.

✔️ 추상메서드 process()

```java
public abstract boolean process(
 Set<? extends TypeElement> annotations, RoundEnvironment roundEnv
);

public synchronized void init(ProcessingEnvironment processingEnv)

public SourceVersion getSupportedSourceVersion()

public Set<String> getSupportedAnnotationTypes()

public Set<String> getSupportedOptions()

public Iterable<? extends Completion> getCompletions(...)

protected synchronized boolean isInitialized()

private Set<String> arrayToSet(...)
```

- process(): 각 프로세서의 main 메서드와 같은 역할 이 메서드에서 원하는 동작을 코드로 작성
- getSupportedAnnotationTypes(): 처리할 Annotation을 명시 이 메서드는 Annotation을 대체할 수 있다.
- getSupportedSourceVersion(): 특정 자바버전을 명시하는데 사용할 수 있으며 Annotation을 대체 할 수 있다.
  - Annotation에서는 SourceVersion.latestSupported()를 사용하지 못하고 특정 버전을 명시해줘야 한다는 점이 다르다.
- init(ProcessingEnvironment processingEnvironment): 모든 Annotation Processor는 기본 생성자를 가져야 한다. 대신 init 메서드를 사용할 수 있는데 processingEnvironment에서 Util을 제공한다.
  1. Types typeUtils = processingEnvironment.getTypeUtils();
  2. Elements elementUtils = processingEnvironment.getElementUtils();
  3. Filer filer = processingEnvironment.getFiler();
  4. Messager messager = processingEnvironment.getMessager();

___

## 🔘 Annotation Processsor 동작

Annotation Processor는 여러번 반복해서 수행된다.
각 라운드는 컴파일러가 어노테이션을 검색하고, 해당 어노테이션에 맞는 Processor를 선택하는 것 부터 시작된다.

> If any files are generated during this process, another round is started with the generated files as its input. This process continues until no new files are generated during the processing stage.

Annotation Processor 또한 JVM 위에서 돌아가므로 라이브러리를 사용할 수 있다. 이렇게 만든 Processor를 .jar 파일로 만들어야 하는데 이렇게 압축할 때 META-INF/services 안에 있는 Processor를 같이 압축해줘야 한다.(google의 auto-service를 사용하면 알아서 해준다.)

## 🔘 Poet 라이브러리

> JAVA 소스 파일을 생성하기 위한 JAVA API

✔️ Hello World 만들기

```java
MethodSpec main = MethodSpec.methodBuilder("main")
    .addModifiers(Modifier.PUBLIC, Modifier.STATIC)
    .returns(void.class)
    .addParameter(String[].class, "args")
    .addStatement("$T.out.println($S)", System.class, "Hello, JavaPoet!")
    .build();

TypeSpec helloWorld = TypeSpec.classBuilder("HelloWorld")
    .addModifiers(Modifier.PUBLIC, Modifier.FINAL)
    .addMethod(main)
    .build();

JavaFile javaFile = JavaFile.builder("com.example.helloworld", helloWorld)
    .build();

javaFile.writeTo(System.out);
```

```java
public static void main(String[] args) throws IOException {
   System.out.println( "Hello World!" );
   HelloWorldPoet poet = new HelloWorldPoet();
   poet.generatedHelloWorld();
}
```

- writeTo(): 입력된 PATH에 파일이 생성됨()

___

## 🔘 Poet 라이브러리
