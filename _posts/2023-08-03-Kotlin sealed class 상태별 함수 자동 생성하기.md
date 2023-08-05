---
layout: post
title: Kotlin sealed class/interface 각각에 대하여 상태 함수 자동 생성하기
subtitle: Kotlin Symbol Processing을 사용해서, 바인딩 파일을 자동 생성시키는 방법
published: true
categories: Kotlin
tags: [Kotlin]
---

## sealed interface/class 를 사용하는 경우
---

> 아래의 코드 처럼 데이터의 상태에 따라 다른 동작을 수행해야 하는 경우가 있습니다.

```kotlin
sealed interface UiState<out T> {
  data class Success<out T>(val data: T) : UiState<T>
  data class Error(val exception: Throwable) : UiState<Nothing>
  object Loading : UiState<Nothing>
}
```

> 이러한 경우, `if` 또는 `when` 문을 사용해서 각 상태에 따른 동작을 수행하게 됩니다.

```kotlin
when (uiState) {
  is UiState.Success -> {
    // uiState.data
  }
  is UiState.Error -> {
    // uiState.exception
  }
  is UiState.Loading -> {
    // 로딩 처리
  }
}
```

when 처리를 매번 하기는 여간 귀찮고 불편한 일이 아닙니다.

`when` 분기를 처리하는 대신에 각 클래스에 따른 함수를 만들어서, 상태에 따른 동작을 수행하도록 하는 방법도 있습니다.

```kotlin
inline fun <T> UiState<T>.onError(block: (Throwable) -> Unit): UiState<T> {
  if (this is UiState.Error)
    block(exception)
  return this
}

inline fun <T> UiState<T>.onLoading(block: () -> Unit): UiState<T> {
  if (this is UiState.Loading)
    block()
  return this
}

inline fun <T> UiState<T>.onSuccess(block: (T) -> Unit): UiState<T> {
  if (this is UiState.Success)
    block(data)
  return this
}
```

이렇게 하면, `when` 분기를 처리하는 대신에, 각 상태에 따른 함수를 호출되면서 동작이 수행됩니다.

```kotlin
uiState
  .onSuccess { data ->
    // data
  }
  .onError { exception ->
    // exception
  }
  .onLoading {
    // 로딩 처리
  }
```

하지만, `UiState` 클래스 내용이 변경되면, 각 상태에 따른 함수도 변경해야 합니다.

이런 불편함을 개선시키고자 KSP(Kotlin Symbol Processing)을 사용해서, `UiState` 클래스에 따른 함수를 자동 생성시키는 방법을 알아보겠습니다.

알려드릴 방법을 사용하면, 컴파일시에 자동으로 sealed interface/class에 대해서 각 상태별로 함수를 만들게 됩니다.


## KSP(Kotlin Symbol Processing) 란?
---
> 코틀린 컴파일러의 확장 기능으로, 컴파일 시간에 코드를 분석하고 생성하는 기능을 제공합니다. KSP를 사용하면, 컴파일러가 코드를 분석하여 새로운 코드를 생성할 수 있습니다. 이를 통해, 코드 생성을 자동화하고, 코드의 반복 작성을 줄일 수 있습니다.

KSP가 아닌 KAPT(Kotlin Annotation Processing Tool)도 있습니다. 이것도 같은 역할을 수행하지만 다음과 같은 차이점이 있습니다.

- KAPT
  - Kotlin코드를 Java Bytecode로 변환한 후, Java Annotation Processing Tool을 사용해서, 코드를 처리합니다.
- KSP
  - Kotlin 코드를 Java로 변환하지 않고 바로 처리하기 때문에, KAPT보다 성능이 더 좋습니다.
  - Kotlin에 최적화 되어 있습니다.


## KSP를 사용하기 위한 사전설정

**project build.gradle**

version은 현재 사용중인 Kotlin version에 맞게 설정합니다.

```
plugins {
    id("com.google.devtools.ksp") version "1.8.22-1.0.11" apply false
    id("org.jetbrains.kotlin.jvm") version "1.8.22" apply false
}
```

## sealed interface/class에 대한 함수 자동 생성하는 과정

### 1. Annotation, Compiler Module 생성
---
> Annotation, Compiler 두 개의 모듈을 만들어야 합니다.

**Annotation Module**  
**build.gradle**

```
plugins {
    id("org.jetbrains.kotlin.jvm")
    id("com.google.devtools.ksp")
}
```

**Compiler Module**  
**build.gradle**

```
plugins {
    id("com.google.devtools.ksp")
}

dependencies {
    implementation("com.google.devtools.ksp:symbol-processing-api:1.8.22-1.0.11")
}
```

**src/main**

- 폴더 생성
  - resources/META-INF/services
    - com.google.devtools.ksp.processing.SymbolProcessorProvider 파일 생성

**com.google.devtools.ksp.processing.SymbolProcessorProvider** 파일의 내용으로

```
io.github.pknujsp.core.compiler.BindFuncProcessorProvider
```

를 작성합니다.


### 2. Annotation Module에 Annotation 생성
---
Annotation Module에 Annotation.kt를 생성하고, 아래의 내용을 작성합니다.

```kotlin
@Target(AnnotationTarget.CLASS)
@Inherited
annotation class KBindFunc
```

@KBindFunc로 사용할 수 있게 되며, `sealed interface/class`에 붙여서 사용합니다.

### 3. Compiler Module에 Processor 생성
---

Compiler Module에 두 개의 파일을 만들어야 합니다.

1. BindFuncProcessorProvider.kt
2. BindFuncKspProcessor.kt


**BindFuncProcessorProvider.kt**

- `SymbolProcessorProvider`는 KSP에서 Annotation 처리를 위한 사용자 정의 `SymbolProcessor`를 생성하는 방법을 정의하는 인터페이스입니다.
- `BindFuncProcessorProvider`의 `create` 메서드는 이 인터페이스의 구현부로서, 이 메서드를 호출할 때마다 새로운 `BindFuncKspProcessor` 인스턴스를 생성합니다.
- `environment.codeGenerator`, `environment.logger`, `environment.options`는 각각 코드 생성, 로깅, 옵션 처리를 담당하는 객체입니다.

```kotlin
class BindFuncProcessorProvider : SymbolProcessorProvider {
  override fun create(environment: SymbolProcessorEnvironment): SymbolProcessor = BindFuncKspProcessor(
    codeGenerator = environment.codeGenerator,
    logger = environment.logger,
    options = environment.options,
  )
}
```

**BindFuncKspProcessor.kt**

```kotlin
class BindFuncKspProcessor(
  private val codeGenerator: CodeGenerator,
  private val logger: KSPLogger,
  private val options: Map<String, String>,
) : SymbolProcessor {

  private companion object {
    val ANNOTATION_TYPE: String = KBindFunc::class.java.canonicalName
    val PREFIX_OUTPUT_FILE_NAME = ANNOTATION_TYPE
  }

  // KSP에서 핵심 함수
  override fun process(resolver: Resolver): List<KSAnnotated> {
    val declarations = resolver.getSymbolsWithAnnotation(ANNOTATION_TYPE).filterIsInstance<KSClassDeclaration>().toList()

    val files = declarations.map { declaration ->
      val typeParams = declaration.typeParameters.mapTo(mutableListOf()) { parameter ->
        "$parameter, ${parameter.variance.name}, ${parameter.isReified}, ${parameter.bounds.map { it.element }.toList()}"
      }
      val properties = declaration.getDeclaredProperties().mapTo(mutableListOf()) { property ->
        "${property.simpleName.asString()} : ${property.type.resolve()}"
      }
      val impls = declaration.getSealedSubclasses().mapTo(mutableListOf()) { impl ->
        "${impl.simpleName.asString()}"
      }

      declaration
    }
    files.forEach { createBindingFile(it) }
    return declarations.toList()
  }

  // 바인딩 파일을 생성하는 함수
  private fun createBindingFile(declaration: KSClassDeclaration) {
    val removeImports = mutableSetOf<String>()
    val funcSpecs = declaration.getSealedSubclasses().map { createMethods(declaration, it, removeImports) }

    val newFileSpec = FileSpec.builder(declaration.packageName.asString(), "${PREFIX_OUTPUT_FILE_NAME}${declaration.simpleName.asString()}").apply {
      funcSpecs.forEach {
        addFunction(it)
      }
    }.build()

    try {
      codeGenerator.createNewFile(
        dependencies = Dependencies(false, declaration.containingFile!!),
        packageName = declaration.packageName.asString(),
        fileName = "${PREFIX_OUTPUT_FILE_NAME}_${declaration.simpleName.asString()}",
      ).bufferedWriter().use {
        it.write(
          newFileSpec.toString().run {
            removeUnnecessaryImports(this, removeImports)
          },
        )
      }
    } catch (e: Exception) {
    }
  }

  // import T와 같이 불필요한 import문을 제거하는 함수
  private fun removeUnnecessaryImports(content: String, imports: MutableSet<String>): String {
    var content = content
    imports.forEach {
      content = content.replace("import $it\n", "")
    }
    return content
  }

  // sealed interface/class에 대하여 바인딩 함수를 생성하는 함수
  private fun createMethods(parent: KSClassDeclaration, sub: KSClassDeclaration, removes: MutableSet<String>): FunSpec {
    print("${sub}--------------------------")

    val isGeneric = parent.typeParameters.isNotEmpty()
    val typeParameters = if (isGeneric) parent.typeParameters
    else emptyList()
    val properties = sub.getDeclaredProperties().toList()

    logger.info("fields: $properties")

    return FunSpec.builder("on${sub.simpleName.asString()}").run {
      addModifiers(KModifier.PUBLIC)
      addModifiers(KModifier.INLINE)
      receiver(
        ClassName.bestGuess(parent.qualifiedName!!.asString()).run {
          if (isGeneric) parameterizedBy(typeParameters.map { TypeVariableName(it.name.asString()) }.toList())
          else this
        },
      )
      addTypeVariables(
        typeParameters.map { ksTypeParameter ->
          TypeVariableName(
            ksTypeParameter.name.asString(),
            variance = KModifier.values().find { it.name == ksTypeParameter.variance.name.lowercase() },
          ).let { typeVariableName ->
            if (ksTypeParameter.bounds.toList().isNotEmpty() && ksTypeParameter.bounds.any { it.element != null }) {
              typeVariableName.copy(
                bounds = ksTypeParameter.bounds.toList().map {
                  TypeVariableName(it.toString())
                },
              )
            } else {
              typeVariableName
            }
          }
        }.toList(),
      )
      addParameter(
        ParameterSpec.builder(
          "block",
          LambdaTypeName.get(
            parameters = properties.map {
              ParameterSpec.unnamed(
                if (it.type.resolve().declaration.typeParameters.isNotEmpty()) {
                  TypeVariableName(it.type.resolve().declaration.typeParameters.first().name.asString())
                } else {
                  ClassName.bestGuess(
                    it.type.resolve().declaration.qualifiedName!!.asString().run {
                      if (contains(sub.simpleName.getShortName())) {
                        it.type.resolve().declaration.simpleName.getShortName().apply { removes.add(this) }
                      } else this
                    },
                  )
                },
              )

            }.toList(),
            returnType = UNIT,
          ),
        ).build(),
      )
      returns(
        ClassName.bestGuess(parent.qualifiedName!!.asString()).run {
          if (isGeneric) parameterizedBy(typeParameters.map { TypeVariableName(it.name.asString()) }.toList())
          else this
        },
      )
      addStatement("if (this is ${parent.simpleName.asString()}.${sub.simpleName.asString()})")

      val block = if (properties.isNotEmpty()) {
        properties.mapIndexed { i, v ->
          if (i < properties.size - 1) "${v.simpleName.asString()}," else v.simpleName.asString()
        }.joinToString("").run { "block(${this})" }
      } else {
        "block()"
      }

      addStatement(
        """
        |  $block
        """.trimMargin(),
      )
      addStatement("return this")
      build()
    }
  }

}
```

### 3. Annotation 사용법
---

sealed class에 @KBindFunc를 붙여주고 컴파일하면 끝입니다.

```kotlin
@KBindFunc
sealed interface UiState<out T> {
  data class Success<out T>(val data: T) : UiState<T>
  data class Error(val exception: Throwable) : UiState<Nothing>
  object Loading : UiState<Nothing>
}
```

컴파일 결과 아래와 같은 함수를 가진 바인딩 파일이 생성됩니다.

```kotlin
public inline fun <T> UiState<T>.onError(block: (Throwable) -> Unit): UiState<T> {
  if (this is UiState.Error)
    block(exception)
  return this
}

public inline fun <T> UiState<T>.onLoading(block: () -> Unit): UiState<T> {
  if (this is UiState.Loading)
    block()
  return this
}

public inline fun <T> UiState<T>.onSuccess(block: (T) -> Unit): UiState<T> {
  if (this is UiState.Success)
    block(data)
  return this
}
```

빌더 패턴으로 되어있으므로 함수를 붙여서 사용하면 됩니다.

```
uiState.onSuccess {
  // UiState.Success
}.onError {
  // UiState.Error
}.onLoading {
  // UiState.Loading
}
```

## 라이브러리 배포
---

> 사용하기 쉽도록 라이브러리로 배포하였습니다.

아래 처럼 build.gradle에 추가하고 사용하면 됩니다.

```
plugins {
  id("com.google.devtools.ksp")
}

dependencies {
  ksp("io.github.pknujsp:ksealedbinding-compiler:1.0.0")
  implementation("io.github.pknujsp:ksealedbinding-annotation:1.0.0")
}
```