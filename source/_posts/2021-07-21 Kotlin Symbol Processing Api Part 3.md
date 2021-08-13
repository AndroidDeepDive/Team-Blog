---
title: Kotlin Symbol Processing Api Part 3 — KSP vs KAPT
date: 2021-07-21 03:00
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

앞선 두 개의 포스팅에서 **KAPT** 와 **KSP** 에 대해서 다루었다.

이번 포스팅에서는 **KAPT** 와 **KSP** 를 서로 비교해본다.


## Better than KAPT

앞선 포스팅에서 언급했듯 KAPT는 Kotlin 코드를 Java Annotation Processor를 수정하지 않기 위해 컴파일시 Java로 된 Stub을 생성하게 된다. 

Stub을 생성하는 것은 kotlinc의 분석 비용의 3분의 1이나 차지하므로, 빌드시 필연적으로 많은 오버헤드가 발생하게 된다. 

KAPT와 달리 KSP는 Java 관점이 아닌 Kotlin의 관점에서 접근하며, `top-level function`과 같은 Kotlin의 고유 기능에 더 적합하다.

KAPT와 좀 더 자세하게 비교해보면 아래로 정리할 수 있다.

**1. 빠르다**

기존에는 코틀린 전용 애노테이션 프로세서가 없었기 때문에, `javax.lang.model` 패키지에서 제공하는 API를 통해 애노테이션 프로세서를 작성했다. 

이 프로세서를 수행하기 위해 KAPT는 코틀린 코드를 **자바 Stub** 으로 컴파일하게 된다. 

이러한 Stub을 생성하려면 KAPT가 코틀린 프로그램의 모든 기호(symbol)들을 확인해야 한다. Stub 코드를 생성하는 비용은 컴파일 전체의 1/3을 차지한다. 

성능 평가를 위해 KSP에서 Glide의 단순화된 버전을 구현하여 **Tachiyomi** 프로젝트용 코드를 생성했는데 

코틀린 컴파일 시간은 21.55초에서 KAPT가 코드를 생성하는데 8.67초, KSP가 코드를 생성하는데 1.15초가 걸렸다고 한다. 

> **참고** [Tachiyomi repository](https://github.com/tachiyomiorg/tachiyomi)

**2. 쉽다**

KSP는 코틀린 친화적이다. 

KSP는 코틀린만의 고유한 기능들인 확장 함수(extension function), 선언 위치 변환 (Declaration-Site Variance), 지역 함수(local functions) 등을 이해한다. 

또한 타입을 모델링하고 동등성 및 할당호환성(assign-comppatibility)과 같은 기본적인 타입을 검사하는 기능을 제공한다.

KSP를 이용하여 소스코드를 처리할 때 클래스, 클래스 멤버, 함수 및 관련 매개변수와 같은 내용에 쉽게 접근이 가능하다. 

개념적으로는 코틀린 리플렉션의 KType과 유사하기 때문에 커스텀 SymbolProcessor 작성 시 AbstractProcessor와 비교하여 작성이 편하다는 느낌을 받게 된다.

자세한 내용은 Part 2를 참고하자.

**3. 호환성 및 유지보수**

KSP는 JVM에 종속되지 않도록 설계되었기 때문에 향후 다른 플랫폼에 보다 쉽게 적용할 수 있다. 

또한 컴파일러 변경 사항을 숨기도록 설계되어 이를 사용하는 프로세서의 유지 관리 노력을 최소화 한다. 

<br>
<br>
<br>

### KAPT vs KSP 실제로 빌드해보기 (Feat_ Room)

Rooom 라이브러리를 이용해 간단하게 테스트를 수행해보았다.

테스트 조건은 아래와 같다.

1. 프로젝트로 작성된 간단한 프로젝트 
2. `RoomDatabase`를 상속받은 클래스 하나
3. 3개의 DAO, 3개의 Entity 클래스

#### KAPT 프로젝트

빌드 스크립트에 `kotlin-kapt`를 추가한다.

```kotlin
// build.gradle (app)
plugins {
  ...
  id 'kotlin-kapt'
  ...
}
```

의존성으로는 테스트할 Room 라이브러리를 추가하고, kapt에서 room을 사용하도록 처리한다.

아래의 의존성으로 인해 Room 라이브러리는 Kotlin 어노테이션을 파싱한 뒤 SQLite를 사용하기 위한 자바 기반의 Stub 파일을 생성하게 된다.

```kotlin
dependencies {
  // ...
  implementation "androidx.room:room-runtime:2.3.0"
  kapt "androidx.room:room-compiler:2.3.0"
  implementation "androidx.room:room-ktx:2.3.0"
  // ...
}
```

이제 `RoomDatabase`를 상속한 `ApplicationDatabase` 클래스를 작성한다.

```kotlin
@Database(
  entities = [AEntity::class, BEntity::class, CEntity::class],
  version = 1,
  exportSchema = false
)
abstract class ApplicationDatabase: RoomDatabase() {

  companion object {
    const val DB_NAME = "ApplicationDataBase.db"
  }

  abstract fun ADao(): ADao

  abstract fun BDao(): BDao

  abstract fun CDao(): CDao

}
```

Kotlin 코드는 다음과 같은 Java Stub 파일로 생성되게 된다.

```java
// ...
  
@SuppressWarnings({"unchecked", "deprecation"})
public final class ApplicationDatabase_Impl extends ApplicationDatabase {
  private volatile ADao _aDao;

  private volatile BDao _bDao;

  private volatile CDao _cDao;

  @Override
  protected SupportSQLiteOpenHelper createOpenHelper(DatabaseConfiguration configuration) {
    final SupportSQLiteOpenHelper.Callback _openCallback = new RoomOpenHelper(configuration, new RoomOpenHelper.Delegate(1) {
      @Override
      public void createAllTables(SupportSQLiteDatabase _db) {
        // ...
      }
    }                              
// ...                                                                              
```

**빌드 타임 측정**

{% asset_img rcWvgv0.png [rcWvgv0] %}

프로젝트 클린 후

kaptGenerateStubsDebugKotlin으로 빌드시 **16.6s** 가 소요되었다.

#### KSP 프로젝트

위에서 작성한 KAPT 프로젝트를 KSP로 변경해보자.

프로젝트 단위의 build.gradle 설정에서는 아래와 같이 의존성을 하나 더 추가해준다.

```kotlin
// build.gradle (root)
buildscript {
  ext.kotlin_version = "1.4.21"
  repositories {
    google()
  }
  dependencies {
    ...
    classpath("com.google.devtools.ksp:com.google.devtools.ksp.gradle.plugin:1.5.20-1.0.0-beta04")
  }
}
```

app의 build.gradle에 `com.google.devtools.ksp` 플러그인을 추가한다.

```kotlin
plugins {
  ...
  id 'kotlin-kapt'
  id 'com.google.devtools.ksp'
  ...
}
```

이후 기존 kapt로 추가했던 `room-compiler` 의존성을 ksp로 교체한다.

```kotlin
dependencies {
  // ...
  implementation "androidx.room:room-runtime:2.3.0"
  // kapt "androidx.room:room-compiler:2.3.0"
  ksp "androidx.room:room-compiler:2.3.0"
  implementation "androidx.room:room-ktx:2.3.0"
  // ...
}
```

**빌드 타임 측정**

{% asset_img HyjEy1l.png [HyjEy1l] %}

프로젝트 클린 후 빌드하면

kaptGenerateStubsDebugKotlin이 **12.54s**

kspDebugKotlin이 **5.49s** 만큼 시간이 소요되어 총합 **18.03s** 만큼 시간이 소요되었다.

KSP를 써서 더 빨라질 것을 기대했는데 왜 더 걸린 것일까?

Android Developer의 공식 블로그에 그 이유가 공개되어 있다.

> **참고** [Announcing Kotlin Symbol Processing (KSP) Alpha](https://android-developers.googleblog.com/2021/02/announcing-kotlin-symbol-processing-ksp.html)

위의 문서를 보면 아래와 같은 문구가 있다.

That said, using KAPT and KSP in the same module will likely slow down your build initially, so during this alpha period, it is best to use KSP and KAPT in separate modules.

동일한 모듈 내에서 kapt와 ksp를 동시에 사용하면 초기 빌드 속도가 더 소요된다고 명시되어있고, 

이에 대한 해결책으로(ksp가 alpha인 동안은) kapt와 ksp를 별도의 모듈에서 사용하는 것을 권장하고 있다.

현재 프로젝트에서 KAPT를 사용하는 의존성을 전부 게거한 뒤, Room에 대한 KAPT, KSP를 비교하면 아래와 같은 결과를 얻을 수 있다.

**Compile Time By KAPT only using Room**
{% asset_img NjWXyw6.png [Compile Time By KAPT only using Room] %}

8.40s

**Compile Time By KSP only using Room**

{% asset_img 0HScWJk.png [Compile Time By KSP only using Room] %}

0.71s

한정적인 테스트지만 빌드 시간이 현격히 감소한 것을 확인할 수 있다.

<br>
<br>
<br>

## References

### Members of Study
- [찰스의 안드로이드 - [Android] Annotation Processor 만들기](https://www.charlezz.com/?p=1167)
- [Namhoon Kims'Coding Log - KSP Api Part 1](https://namhoon.kim/2021/06/25/android_deep_dive/020_Kotlin%20Symbol%20Processing%20API%20Part%201/)
- [Namhoon Kims'Coding Log - KSP Api Part 2](https://namhoon.kim/2021/06/27/android_deep_dive/021_Kotlin%20Symbol%20Processing%20API%20Part%202/)
- [소다의 개발 블로그 - Introduce Kotlin Symbol Processing](https://soda1127.github.io/introduce-kotlin-symbol-processing/)
- [소다의 개발 블로그 - Implement Kotlin Symbol Processing](https://soda1127.github.io/implement-kotlin-symbol-processing-example/)
- [jsh-me.log - KSP(Kotlin Symbol Processor) 톺아보기](https://velog.io/@jshme/KSP-Kotlin-Symbol-Processor-%ED%86%BA%EC%95%84%EB%B3%B4%EA%B8%B0)

### Official
- [Android Developers#androidx.annotation](https://developer.android.com/reference/androidx/annotation/package-summary)
- [Google KSP Repository](https://github.com/google/ksp)
- [KSP API definition](https://github.com/google/ksp/blob/main/api/src/main/kotlin/com/google/devtools/ksp)
- [KSP Symbol definition](https://github.com/google/ksp/blob/main/api/src/main/kotlin/com/google/devtools/ksp/symbol)
- [KAPT(Kotlin Annotation Processing Tool)](https://kotlinlang.org/docs/kapt.html)
- [Announcing Kotlin Symbol Processing (KSP) Alpha](https://android-developers.googleblog.com/2021/02/announcing-kotlin-symbol-processing-ksp.html)
- [Oracle JavaDoc#Annotations](https://docs.oracle.com/javase/tutorial/java/annotations/index.html)
- [kapt: Annotation Processing for Kotlin](https://blog.jetbrains.com/kotlin/2015/05/kapt-annotation-processing-for-kotlin)
- [Using kapt](https://kotlinlang.org/docs/kapt.html#using-in-gradle)

### Etc
- [Tachiyomi](https://github.com/tachiyomiorg/tachiyomi)
- [My first Kotlin Symbol Processing Tool for Android](https://jsuch2362.medium.com/my-first-kotlin-symbol-processing-tool-for-android-4eb3a2cfd600)
- [KSP: Fact or kapt?](https://proandroiddev.com/ksp-fact-or-kapt-7c7e9218c575)
- [Annotation Processing 101](https://hannesdorfmann.com/annotation-processing/annotationprocessing101/)
- [Kotlin : Kotlin plugin should be enabled before ‘kotlin-kapt’](https://imstudio.medium.com/kotlin-kotlin-plugin-should-be-enabled-before-kotlin-kapt-d5879f45f09d)
- [Pushing the limits of Kotlin annotation processing](https://medium.com/@workingkills/pushing-the-limits-of-kotlin-annotation-processing-8611027b6711)
