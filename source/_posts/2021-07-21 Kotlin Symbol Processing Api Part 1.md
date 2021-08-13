---
title: Kotlin Symbol Processing Api Part 1 — Annotation과 KAPT
date: 2021-07-21 01:00
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

## Annotation

`Annotation Processor`이란 용어는 낯설더라도, 아래와 같은 코드들은 개발하면서 종종 보았을 것이다.

```java
@Overide
@NonNull
@Nallable
```

이러한 코드를 **Annotation**이라고 한다. 

**Annotation** 은 소스 코드에 추가 할 수 있는 메타데이터의 한 형태로 컴파일러가 이를 참조 할 수 있도록 한다.

이 참조를 통해 미리 지정된 코드를 생성하기 위한 용도로 사용된다. Android 뿐만 아니라 Spring Framework 등을 개발할때에도 자주 활용한다.

다음의 오라클 문서에서 발췌한 Annotation의 정의를 살펴보자.

> Annotations, a form of metadata, provide data about a program that is not part of the program itself. Annotations have no direct effect on the operation of the code they annotate.

문서의 내용을 해석하면 

> Annotation은 일종의 메타데이터 형태이며, 프로그램의 일부가 아니라 프로그램에 대한 정보를 제공한다.
> Annotation 자체로는 실행되는 코드에 직접적인 영향을 미치지 않는다.

Annotation은 Java 5부터 지원하고 있으며, 주로 다음과 같은 용도로 사용된다.

**1. Information for the compiler**
컴파일러가 에러를 검출하거나, 경고를 표시하지 않도록 사전에 정보를 전달한다.

**2. Compile-time and deployment-time processing**
코드나 xml 파일 등을 컴파일 타임에 생성할 수 있도록 처리한다.

**3. Runtime processing**
몇몇 annotation들은 런타임에도 검사를 수행하도록 처리해준다.

> **참고** [Oracle JavaDoc#Annotations](https://docs.oracle.com/javase/tutorial/java/annotations/index.html)

### 이미 정의된 Annotation 살펴보기

`java.lang` 패키지에는 자바에서 제공하는 Annotation들이 있다. 대표적인 것 두 가지만 가지 살펴보자.

- `@Deprecated` : deprecated 됐음을 의미하며 더 이상 쓰지 말 것을 권장할 때 사용한다. 이 Annotation이 달린 코드를 사용하면 컴파일러는 경고를 내뱉는다.

- `@SuppressWarnings` : 이 Annotation은 컴파일러가 생성할 경고를 억제하도록 지시한다.

  ```java
  @SuppressWarnings("deprecation")
  void useDeprecatedMethod() {
    // deprecation warning
    // - suppressed
    objectOne.deprecatedMethod();
  }
  ```

`java.lang` 하위의 `annotation` 패키지 즉 `java.lang.annotation` 패키지에서는 **Meta Annotation** 이라고 불리는 것들이 존재한다.

- `@Retention` : Retention Annotation은 표기된 Annotation이 저장되는 방법을 지정한다.
  - `RetentionPolicy.SOURCE` : 소스 코드에서만 유지되며, 컴파일러에서는 무시된다.
  - `RetentionPolicy.CLASS` : 컴파일 타임에 컴파일러에 의해 유지되지만, JVM에서는 무시된다.
  - `RetentionPolicy.RUNTIME` : JVM에 의해서 유지되므로 런타임에서 사용가능하다.
- `@Documented` : Annotation이 사용될 때 마다 해당 element가 Javadoc에 문서화 되어야 함을 나타낸다.
- `@Target` : Annotation을 적용할 수 있는 Java Elements 종류를 제한한다.
- `@Inherited` : super class 로부터 상속될 수 있는 Annotation 타입이다. class 선언시에만 적용.

이 Meta Annotation의 특징은 다른 Annotation에 적용이 가능한 Annotation이라는 점이다.

Android 앱 개발시에는 `java.lang` 대신 `androidx.annotation` 패키지를 참조하면 다양한 Annotation을 활용할 수 있다.

`androidx.annotation` 패키지에 속한 annotation 목록은 아래 링크를 참고하자.

> **참고** [Android Developers#androidx.annotation](https://developer.android.com/reference/androidx/annotation/package-summary)


### Annotation Processor

앞서 소개한 Annotation의 용도를 사용하기 위해서 **Annotation Processor**가 필요하다. 

Annotation Processor는 Java 컴파일러 플러그인의 일종으로, 컴파일러에게 어떠한 요소(클래스, 메서드, 필드 등)에 annotation이 추가 되어있는지 확인하도록 한다.

컴파일러는 컴파일 타임에 코드베이스를 검사하거나 확인된 정보를 통해 새로운 코드를 생성하는 식으로 동작하며, 주로 개발자가 정의한 태스크를 자동화하거나 보일러 플레이트 작성을 줄이는 용도로 사용된다.

Annotation Processor의 특징을 정리하자면 다음과 같다.

- 컴파일 타임에 특정 작업을 수행한다.
- 리플렉션없이 프로그램의 의미 및 구조를 파악할 수 있게 된다.
- 자동으로 보일러 플레이트를 생성할 수 있게 된다.

#### Annotation processor 실행 순서

Annotation processor는 여러 라운드에 걸쳐 수행된다.

{% asset_img processing_rounds.png [Processing rounds] %}

실행 순서를 간단히 정리하자면 다음과 같다.

1. 등록된 Annotation processor들과 함께 컴파일러가 시작된다.
2. Annotation processor들이 작성된 Annotation을 기반으로 코드 검사 및 생성을 수행한다.
3. 컴파일러가 모든 Annotation processor의 작업이 끝났는지 확인하고, 그렇지 않다면 2번을 반복한다.
4. 모든 처리가 끝난다면 전체 코드에 대한 컴파일을 시작한다. (이후 프로세스는 기존과 동일)

#### Android에서 Annotation Processor를 사용하는 라이브러리들

**Room** 

Android에서 SQLite에 대한 추상화를 제공하는 Room 라이브러리에도 Annotation Processor가 적용되어 있다.

아래는 대표적인 예제인 User 관련 코드이다.

```kotlin
// User.kt
@Entity
data class User(
    @PrimaryKey val uid: Int,
    @ColumnInfo(name = "first_name") val firstName: String?,
    @ColumnInfo(name = "last_name") val lastName: String?
)
```

```kotlin
// UserDao.kt
@Dao
interface UserDao {
    @Query("SELECT * FROM user")
    fun getAll(): List<User>

    @Query("SELECT * FROM user WHERE uid IN (:userIds)")
    fun loadAllByIds(userIds: IntArray): List<User>

    @Query("SELECT * FROM user WHERE first_name LIKE :first AND " +
            "last_name LIKE :last LIMIT 1")
    fun findByName(first: String, last: String): User

    @Insert
    fun insertAll(vararg users: User)

    @Delete
    fun delete(user: User)
}
```

```kotlin
// AppDatabase.kt
@Database(entities = arrayOf(User::class), version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}
```

위의 에졔에서 쓰인 Annotation들은 `@Entity`, `@PrimaryKey`, `@ColumnInfo`, `@Dao`, `@Query`, `@Insert`, `@Delete`이다.

이 Annotation들의 구현체를 확인해보자.

```java
// Entity.java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.CLASS)
public @interface Entity {
    String tableName() default "";
    Index[] indices() default {};
    boolean inheritSuperIndices() default false;
    String[] primaryKeys() default {};
    ForeignKey[] foreignKeys() default {};
    String[] ignoredColumns() default {};
}
```

```java
// PrimaryKey.java
@Target({ElementType.FIELD, ElementType.METHOD})
@Retention(RetentionPolicy.CLASS)
public @interface PrimaryKey {
    boolean autoGenerate() default false;
}
```

```java
// ColumnInfo.java
@Target({ElementType.FIELD, ElementType.METHOD})
@Retention(RetentionPolicy.CLASS)
public @interface ColumnInfo {
    String name() default INHERIT_FIELD_NAME;
    @SuppressWarnings("unused") @SQLiteTypeAffinity int typeAffinity() default UNDEFINED;
    boolean index() default false;
    String defaultValue() default VALUE_UNSPECIFIED;
    String INHERIT_FIELD_NAME = "[field-name]";


    int UNDEFINED = 1;
    int TEXT = 2;
    int INTEGER = 3;
    int REAL = 4;
    int BLOB = 5;
    @IntDef({UNDEFINED, TEXT, INTEGER, REAL, BLOB})
    @Retention(RetentionPolicy.CLASS)
    @interface SQLiteTypeAffinity {
    }

    int UNSPECIFIED = 1;
    int BINARY = 2;
    int NOCASE = 3;
    int RTRIM = 4;
    @RequiresApi(21)
    int LOCALIZED = 5;
    @RequiresApi(21)
    int UNICODE = 6;
    @IntDef({UNSPECIFIED, BINARY, NOCASE, RTRIM, LOCALIZED, UNICODE})
    @Retention(RetentionPolicy.CLASS)
    @interface Collate {
    }
    String VALUE_UNSPECIFIED = "[value-unspecified]";
}
```

이 외에도 `@Dao`, `@Query`, `@Insert`, `@Delete`과 같은 Annotation들은 각자 인터페이스, 구현체 값들을 이미 가지고 있다.

 **Room** 뿐만 아니라 범용적으로 사용되는 **Dagger**, **Glide**와 같은 라이브러리들도 Annotation(Processor)을 기반으로 동작한다. 

<br>
<br>
<br>

## KAPT (Kotlin Annotation Processing Tool)

코틀린 프로젝트를 컴파일 할 때는 javac가 아닌 kotlinc로 컴파일을 하기 때문에 Java로 작성한 애노테이션 프로세서(AbstractProcessor)가 동작하지 않는다. 

따라서 코틀린에서는 이러한 애노테이션 처리기를 위해 [KAPT(Kotlin Annotation Processing Tool)](https://kotlinlang.org/docs/kapt.html)를 제공한다. 

KAPT를 사용하기 위해 모듈의 build.gradle에 다음과 같은 코드를 추가한다.

```groovy
// Groovy DSL
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-kapt'
```

이 때 kotlin-android 설정을 먼저 해줘야 kotlin-kapt를 쓸 수 있다.

Annotation processor가 포함된 라이브러리를 추가한다면 다음과 같이 의존성을 추가한다.

```groovy
dependencies {
 // 기존 annotationProcessor 대신 kapt로 대체
 // annotationProcessor 'groupId:artifactId:version'
    kapt 'groupId:artifactId:version'
}
```

hilt로 예시를 바꿔보면 아래와 같다.

```groovy
dependencies{
  //annotationProcessor "com.google.dagger:hilt-android-compiler:$hilt_version"
  kapt "com.google.dagger:hilt-android-compiler:$hilt_version"
}
```

### Pluggable Annotation Processing API (JSR#269)

자바 이외의 언어에서 어노테이션을 지원하기 위해서는 몇가지 옵션이 있다.

자바의 컴파일러와 어노테이션 프로세스를 위한 플러그인 대상 API가 필요한데, 이를 정리한 스펙 문서가 JSR 269이다.

> **참고** [JSR 269 : Pluggable Annotation Processing API in JCP](https://jcp.org/en/jsr/detail?id=269)

이 플러그인 API를 사용하면 특정 어노테이션이 정의되었을때, 컴파일러에게 어노테이션에 작성된 클래스, 메서드, 필드 등의 구성 요소를 질의하고

컴파일러는 해당 구성 요소를 나타내는 객체의 컬렉션을 반환하게 된다.

이후 프로세서가 이 컬렉션을 검증하고, 새로운 코드(=Stub)을 생성하게 된다.

코틀린의 경우 빌드한 바이너리가 자바이기 때문에, 코틀린 컴파일러의 실행후 자바 컴파일러가 바이너리 파일인 `*.class`를 인식한다.

이때 컴파일러는 코틀린과 자바에서 생성된 각 바이너리에 대해서 구별할 수는 없다.

다만, 코틀린은 언어의 특성상 Processor가 생성한 선언을 참조할 수 없고, 바이너리에는 주석이 포함되지않기때문에 이를 해결하기 위해 **KAPT**를 사용하는 것이다.

**KAPT**를 사용하면 APT와 똑같이 Stub을 생성하고 자바의 의존성을 가지는 대신 구현이 상대적으로 쉽다는 장점이 있다.

자바와 코틀린간의 간극을 없애주는 **KAPT**의 대표적인 예시로 Android의 DI를 위해 사용하는 Dagger나 Databinding을 코틀린에서도 사용할 수 있는 점을 들 수 있다.

하지만 **KAPT** 도 APT와 마찬가지로 결국 Stub을 생성하기 위해 많은 컴파일 및 빌트 타임을 소모하게 되는 문제점은 그대로 남아있게 된다.

이번 포스팅의 주제이자, 위의 문제를 해결하기 위해 나온 **KSP**를 다음 파트에서 자세히 알아보자.