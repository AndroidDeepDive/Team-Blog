---
title: Kotlin Symbol Processing Api Part 2 — What is it?
date: 2021-07-21 02:00
cover: https://androiddeepdive.github.io/Team-Blog/images/cover_ksp.png
categories:
- KSP
tags:
- KSP
- Kotlin
- Symbol
- Processing
- KAPT
- Annotation
---

<article class="message message-immersive is-primary">
    <div class="message-body">
        <i class="fas fa-pen-fancy mr-2"></i>Writers<br>
        by 곽욱현 @Knowre<br>
        by 김남훈 @Naver<br>
        by 송시영 @SmartStudy<br>
        by 옥수환 @Naver<br>
        by 이기정 @BankSalad<br>
        by 정세희 @BankSalad
    </div>
</article>


<!-- more -->

## KSP란 무엇인가?

**KSP(Kotlin Symbol Processing)** 은 2021년 2월 10일 구글이 발표한 코틀린에서 경량화된 컴파일러 플러그인을 개발할 수 있는 API다. 

학습곡선을 최소한으로 줄이고, 코틀린의 기능을 활용할 수 있는 단순화된 API를 제공한다. 여러 플랫폼에 호환성을 염두하고 만들어졌으며, 코틀린 1.4.30 버전 이상부터 호환된다. 

KSP는 코틀린 언어가 갖는 특징인 확장 함수, 로컬 함수 같은 기능을 이해한다. 

또한 KSP는 타입을 명시적으로 다루고, 기본적인 타입 검사와 동등성 검사를 지원한다. 

API는 코틀린 문법에 따라 symbol 수준에서 코틀린 프로그램 구조를 모델링 한다. 

KSP 기반 플러그인이 소스 프로그램을 처리할 때 클래스, 클래스 멤버, 함수 및 관련 매개 변수와 같은 구성은 프로세서에서 쉽게 접근할 수 있기 때문에 코틀린 개발자들에게 편리하다. 

개념적으로 KSP는 Kotlin 리플렉션의 KType과 유사하다. API 를 사용하면 프로세서가 클래스 선언에서 특정 타입 인자가 있는 해당 타입, 또는 그 반대로 탐색할 수 있다.

KAPT와 비교했을 때 KSP를 사용하는 [Annotation Processor](https://www.charlezz.com/?p=1167)는 최대 **2배** 더 빠르게 실행할 수 있다. 

자세한 내용은 [Google KSP repository](https://github.com/google/ksp)에서 소스코드 및 문서를 확인할 수 있다. 

### 왜 KSP를 사용해야 할까?

컴파일러 플러그인은 코드 작성 방법을 크게 향상시킬 수 있는 강력한 메타프로그래밍 도구이다. 

컴파일러 플러그인은 컴파일러를 라이브러리로 직접 호출하여 입력 프로그램을 분석하고 편집하여 다양한 용도로 쓰일 수 있는 산출물들을 생성한다.. 

예를 들어, Boiler Plate를 생성하거나 Parcelble과 같은 특별히 마크된 프로그램 요소에 대한 전체 구현을 생성할 수도 있다. 

플러그인은 다양한 용도로 사용되며 언어로 직접 제공되지 않는 기능을 구현하고 미세 조정하는 데 사용할 수도 있다. 

다만 컴파일러 플러그인은 컴파일러에 대한 배경 지식 및 특정 컴파일러의 구현 세부 사항에 대한 숙련도를 어느정도 필요로 하므로 어느 정도의 진입 장벽이 요구된다.

이 진입 장벽을 최대한 낮추고자 KSP 는 컴파일러의 변경사항을 은닉하도록 설계되어 최소한의 유지 보수로 플러그인을 개발할 수 있게 해준다.

또한 JVM에 종속되지않도록 설계되었으므로, 다른 플랫폼에 적용하는 것도 용이하다.

무엇보다 KSP는 빌드시간을 최소화하는 것이 가장 큰 장점이다.

Glide 와 같은 일부 Processor의 경우 KSP는 KAPT와 비교할 때 컴파일 시간을 25%까지 줄이는 것으로 확인되었다.

#### **kotlinc** 컴파일러 플러그인과의 비교

`kotlinc` 컴파일러 플러그인의 경우 강력한 기능을 제공하는 것은 맞지만, 그만큼 컴파일러에 대한 의존성이 크기때문에 유지보수에 대한 용이성이 떨어진다. 반면 KSP는 대부분의 컴파일러 변경사항을 은닉하여 api를 통해 접근할 수 있도록 해준다. 물론 한 단계를 더 건너야하는 만큼 `kotlinc`의 모든 기능을 지원하지는 않지만, 기술 부채를 고려하였을 때 합리적인 선택이 될 것이다.

#### **kotlin.reflect** 와의 비교

KSP는 `kotlin.reflect`와 유사하게 생겼지만, KSP는 타입에 대한 참조를 명시적으로 지정해주어야 한다.

### KSP의 한계점

KSP는 기존의 Annotation Processor에 비해 상대적으로 간단한 방법론을 제공하기 위해 몇 가지 절충한 부분이 존재한다.

따라서 아래의 기능들은 KSP에서 제공하고자 하는 대상이 아니다.

1. 소스 코드의 표현 수준 정보를 조사하기
2. 소스 코드 수정하기
3. Java Annotation Processing API와 100% 호환하기
4. IDE와 통합하기 (현재는 IDE가 생성 된 코드를 읽지 못함)

특히 4번 항목때문에 Android Studio에서 KSP를 개발하여 사용하고자 하는 경우엔 아래와 같은 경로를 명시해야 한다.

```kotlin
build/generated/ksp/debug/kotlin
```

build.gradle.kts 예시

```kotlin
android {
    buildTypes {
        getByName("debug") {
            sourceSets {
                getByName("main") {
                    java.srcDir(File("build/generated/ksp/debug/kotlin"))
                }
            }
        }
    }
}
```

### KSP 내부 살펴보기

{% asset_img ClassDiagram.svg [ClassDiagram] %}

위의 그림은 매우 복잡하고 사이즈가 크니, 클릭해서 크게 보는 것을 권장한다.

> **참고** [KSP API definition](https://github.com/google/ksp/blob/main/api/src/main/kotlin/com/google/devtools/ksp)

> **참고** [KSP Symbol definition](https://github.com/google/ksp/blob/main/api/src/main/kotlin/com/google/devtools/ksp/symbol)

KSP 모델에 대한 딥다이브를 해보자.

먼저 KSP의 전체 구조는 아래와 같다.

#### KSP가 파일을 파싱하는 구조

KSP를 사용하기 위해 작업하는 경우 아래와 같은 구조로 작성해야 한다.

복잡해보이지만 `KS`는 prefix일 뿐이고, kotlin으로 작성한 파일 구조를 추종한다는 것을 알 수 있다.


```kotlin
KSFile
    /**
     * File의 정보
     * - Package Name / File Name / 적용된 Anootation 리스트
     * - simpleName, qualifiedName 등을 포함한 선언 정보
     */
    packageName: KSName
    fileName: String
    annotations: List<KSAnnotation>  (File annotations)
    declarations: List<KSDeclaration>
   
    /**
     * Class / Interface / Object
     */
    KSClassDeclaration
        simpleName: KSName
        qualifiedName: KSName
        containingFile: String
        typeParameters: KSTypeParameter
        parentDeclaration: KSDeclaration
        classKind: ClassKind
        primaryConstructor: KSFunctionDeclaration
        superTypes: List<KSTypeReference>
        declarations: List<KSDeclaration> // contains inner classes, member functions, properties, etc.

    /**
     * Top-Level Funtion
     */
    KSFunctionDeclaration
        simpleName: KSName
        qualifiedName: KSName
        containingFile: String
        typeParameters: KSTypeParameter
        parentDeclaration: KSDeclaration
        functionKind: FunctionKind
        extensionReceiver: KSTypeReference?
        returnType: KSTypeReference
        parameters: List<KSValueParameter>
        declarations: List<KSDeclaration> // contains local classes, local functions, local variables, etc.

    /*
     * Global Variable
     */
    KSPropertyDeclaration 
        simpleName: KSName
    qualifiedName: KSName
    containingFile: String
    typeParameters: KSTypeParameter
    parentDeclaration: KSDeclaration
    extensionReceiver: KSTypeReference?
    type: KSTypeReference
    getter: KSPropertyGetter
        returnType: KSTypeReference
    setter: KSPropertySetter
        parameter: KSValueParameter
```

위의 파일 구조를 실제 구조에 대입하여 파악해보자.

`AppcompatActivity`를 상속한 `SampleActivity`가 있다고 할때, 

Annotation이 붙여진 `어떠한 것`이 변수인지, 함수인지, 클래스인지를 알고 싶다면, 위 구성을 이해하면 된다. 아래와 같이 Activity를 구성했다고 가정해보자.

``` kotlin
// KSFile
package com.jshme.kspsample

import androidx.appcompat.app.AppCompatActivity
import android.os.Bundle

@TestAnnotation //KClassDeclaration 
class SampleActivity : AppCompatActivity() {

    @Test //KSPropertyDeclaration
    var number: Int = 0

    @Test //KSPropertyDeclaration
    var str: String = ""

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_sample)
        
        number = 2
        str = "Test Sample"
    }
}
```

`@TestAnnotation` 이 달린 곳은 클래스 타입이므로 KClassDeclaration으로 구성될 것이고, 

@Test` 가 달린 곳은 KSPropertyDeclaration으로 구성될 것이다. 파일 내부를 돌면서 클래스, 함수, 변수타입을 구별해내 `KSDeclaration` 으로 정의하는 것이다. 

이 구조도와 예제를 통해 `KSClassDeclaration`, `KSFunctionDeclaration`, `KSPropertyDeclaration` 

세 가지 타입의 경우 `KSDeclaration`를 상속받고 있기때문에 타입 캐스팅을 사용할 수 있기때문에, 리스트 형태로 관리할 수 있음을 확인할 수 있다.

#### KSP의 타입 참조 프로세스

KSP에서 타입에 대한 참조는 몇 가지 예외를 제외하면 명시적으로 지정하도록 되어있다.

`KSFunctionDeclaration.returnType` 혹은 `KSAnnotation.annotationType`과 같이 타입을 참조하는 경우, 

타입은 항상 annotation과 modifier가 포함된 `KSReferenceElement` 기반의 `KSTypeReference`이다.

```kotlin
interface KSFunctionDeclaration : ... {
    val returnType: KSTypeReference?
    ...
}

interface KSTypeReference : KSAnnotated, KSModifierListOwner {
    val type: KSReferenceElement
}
```

`KSTypeReference`는 Kotlin의 타입 시스템의 `KSType`으로 `resolve()`할 수 있고, Kotlin 문법과 일치하는 `KSReferenceElement`를 가지고 있다.

이번엔 `KSReferenceElement`다.


```kotlin
interface KSReferenceElement : KSNode {
    val typeArguments: List<KSTypeArgument>
}
```

`KSReferenceElement`는 유용한 정보를 많이 포함하고 있는 `KSClassifierReference` 혹은 `KSCallableReference`가 될 수 있다.

```kotlin
interface KSClassifierReference : KSReferenceElement {
    val qualifier: KSClassifierReference?
    fun referencedName(): String

    override fun <D, R> accept(visitor: KSVisitor<D, R>, data: D): R {
        return visitor.visitClassifierReference(this, data)
    }
}
```

예를 들어 `KSClassifierReference`는 `referencedName`라는 속성을 가지고 있으며,

```kotlin
interface KSCallableReference : KSReferenceElement {
    val receiverType: KSTypeReference?
    val functionParameters: List<KSValueParameter>
    val returnType: KSTypeReference

    override fun <D, R> accept(visitor: KSVisitor<D, R>, data: D): R {
        return visitor.visitCallableReference(this, data)
    }
}
```

`KSCallableReference`는 `receiverType`과 `functionArguments` 그리고 `returnType`을 가지고 있다.

`KSTypeReference`에서 참조되는 타입의 선언이 필요한 경우 아래와 같은 순서로 접근한다.

```kotlin
KSTypeReference -> .resolve() -> KSType -> .declaration -> KSDeclaration
```

`resolve()`를 통해 `KSType`으로 접근하고, `declaration` 속성을 통해 `KSDeclaration` 객체를 획득한다.

#### Java Annotation Processing에 대응하는 KSP 레퍼런스

기존에 Annotation processor를 작성해 본 경험이 있다면 아래의 내용을 참조하면 좋다. 내용이 방대하여 링크로 대체한다.

> **참고** [Github ksp#references](https://github.com/google/ksp/blob/main/docs/reference.md)

<br>
<br>
<br>

## KSP 개발 프로세스

KSP repository에도 playground가 있지만, 공식 repo와 문서가 자세하거나 친절하게 작성되어있지않아 임의로 개발 프로세스를 예제 기반으로 명세해보았다.

개략적인 내용은 ksp repository의 QuickStart 문서에 나와있다.

> **참고** [Github ksp#quickstart](https://github.com/google/ksp/blob/main/docs/quickstart.md)

KSP 개발을 위한 환경은 갖추어져 있다고 가정한다.

**Step 1**
You'll need to implement `com.google.devtools.ksp.processing.SymbolProcessor` and `com.google.devtools.ksp.processing.SymbolProcessorProvider`. 

Your implementation of `SymbolProcessorProvider` will be loaded as a service to instantiate the `SymbolProcessor` you implement.

먼저 `SymbolProcessor`를 상속받은 구현체를 작성한다.

```kotlin
// BuilderProcessor.kt
class BuilderProcessor(
    val codeGenerator: CodeGenerator,
    val logger: KSPLogger
) : SymbolProcessor {
    // ...
}
```

그리고 `SymbolProcessorProvider`를 상속받은 구현체도 선언한다.

```kotlin
// BuilderProcessorProvider.kt
class BuilderProcessorProvider : SymbolProcessorProvider {
    // ...
}
```

**Step 2**
Implement `SymbolProcessorProvider.create()` to create a `SymbolProcessor`. 

Dependencies your processor needs (e.g. `CodeGenerator`, processor options) are passed through the parameters of `SymbolProcessorProvider.create()`.

`BuilderProcessorProvider` 클래스에 `SymbolProcessorProvider.create()` 메소드를 구현한다.

```kotlin
// BuilderProcessorProvider.kt
class BuilderProcessorProvider : SymbolProcessorProvider {
    override fun create(environment: SymbolProcessorEnvironment): SymbolProcessor {
        return BuilderProcessor(environment.codeGenerator, environment.logger)
    }
}
```

**Step 3**
Your main logic should be in the SymbolProcessor.process() method.

`BuilderProcessor`의 핵심 로직인 `SymbolProcessor.process()` 메서드를 오버라이딩 한다.

```kotlin
class BuilderProcessor : SymbolProcessor {

    override fun process(resolver: Resolver): List<KSAnnotated> {
        // ...
    }
    // ...
}
```

**Step 4**
Use resolver.getSymbolsWithAnnotation() to get the symbols you want to process, given the fully-qualified name of an annotation.

Step 3에서 선언한 `BuilderProcessor` 클래스의 `process()`에 파라미터로 주어지는 `resolver`에 `getSymbolsWithAnnotation()` 메소드를 호출하여 KSP를 명시한다.

```kotlin
class BuilderProcessor(
    val codeGenerator: CodeGenerator,
    val logger: KSPLogger
) : SymbolProcessor {

    override fun process(resolver: Resolver): List<KSAnnotated> {
        val symbols = resolver.getSymbolsWithAnnotation("android.deepdive.ksp.builder.Builder")
        val ret = symbols.filter { !it.validate() }.toList()

        symbols
            .filter { it is KSClassDeclaration && it.validate() }
            .forEach { it.accept(BuilderVisitor(), Unit) }
        return ret
    }
    // ...
}
```

**Step 5**
A common use case for KSP is to implement a customized visitor (interface com.google.devtools.ksp.symbol.KSVisitor) for operating on symbols. A simple template visitor is com.google.devtools.ksp.symbol.KSDefaultVisitor.

`com.google.devtools.ksp.symbol.KSDefaultVisitor`를 참조하여 Visiter 클래스를 작성한다.

```kotlin
class BuilderProcessor(
    val codeGenerator: CodeGenerator,
    val logger: KSPLogger
) : SymbolProcessor {
    // ...
    inner class BuilderVisitor : KSVisitorVoid() {
        override fun visitClassDeclaration(classDeclaration: KSClassDeclaration, data: Unit) {
            classDeclaration.primaryConstructor!!.accept(this, data)
        }

        override fun visitFunctionDeclaration(function: KSFunctionDeclaration, data: Unit) {
            val parent = function.parentDeclaration as KSClassDeclaration
            val packageName = parent.containingFile!!.packageName.asString()
            val className = "${parent.simpleName.asString()}Builder"
            val file = codeGenerator.createNewFile(Dependencies(true, function.containingFile!!), packageName, className)

            file.appendText("package $packageName\n\n")
            file.appendText("import HELLO\n\n")
            file.appendText("class $className{\n")

            function.parameters.forEach {
                val name = it.name!!.asString()
                val typeName = StringBuilder(it.type.resolve().declaration.qualifiedName?.asString() ?: "<ERROR>")
                val typeArgs = it.type.element!!.typeArguments

                if (it.type.element!!.typeArguments.isNotEmpty()) {
                    typeName.append("<")
                    typeName.append(
                        typeArgs.map {
                            val type = it.type?.resolve()
                            "${it.variance.label} ${type?.declaration?.qualifiedName?.asString() ?: "ERROR"}" +
                                    if (type?.nullability == Nullability.NULLABLE) "?" else ""
                        }.joinToString(", ")
                    )
                    typeName.append(">")
                }
                file.appendText("    private var $name: $typeName? = null\n")
                file.appendText("    internal fun with${name.capitalize()}($name: $typeName): $className {\n")
                file.appendText("        this.$name = $name\n")
                file.appendText("        return this\n")
                file.appendText("    }\n\n")
            }

            file.appendText("    internal fun build(): ${parent.qualifiedName!!.asString()} {\n")
            file.appendText("        return ${parent.qualifiedName!!.asString()}(")
            file.appendText(
                function.parameters.map {
                    "${it.name!!.asString()}!!"
                }.joinToString(", ")
            )
            file.appendText(")\n")
            file.appendText("    }\n")
            file.appendText("}\n")
            file.close()
        }
    }
    // ...
}
```

**Step 6**
For sample implementations of the SymbolProcessorProvider and SymbolProcessor interfaces, see the following files in the sample project.
- src/main/kotlin/BuilderProcessor.kt
- src/main/kotlin/TestProcessor.kt

작성한 클래스는 아래와 같다.

- src/main/java/android/deepdive/ksp/builder/BuilderProcessor.kt
- src/main/java/android/deepdive/ksp/builder/BuilderProcessorProvider.kt

**Step 7**
After writing your own processor, register your processor provider to the package by including its fully-qualified name in resources/META-INF/services/com.google.devtools.ksp.processing.SymbolProcessorProvider.

`META-INF`에 작성한 `BuilderProcessorProvider`의 path를 명시한다.

```
// path : src/main/resources/META-INF/services/com.google.devtools.ksp.processing.SymbolProcessorProvider
android.deepdive.ksp.builder.BuilderProcessorProvider
```

실제 스터디를 진행하면서 만들어본 KSP 예제는 아래 링크를 참조하면 된다.

- [Android Deep Dive#Kotlin-Symbol-Processing-Api](https://github.com/AndroidDeepDive/ADD-Kotlin-Symbol-Processing-Api)
- [Charlezz#IntentBuilderSample](https://github.com/Charlezz/IntentBuilderSample)
- [SODA1127#KSPImplementationExample](https://github.com/SODA1127/KSPImplementationExample)
- [jsh-me#KSPWebViewLoader](https://github.com/jsh-me/KSPWebViewLoader)