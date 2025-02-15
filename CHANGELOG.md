# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](http://keepachangelog.com/en/1.0.0/)
and this project adheres to [Semantic Versioning](http://semver.org/spec/v2.0.0.html).

## [0.4.0] 2021-11-12

### Changed

- Updated kotlin to 1.5.31
- Updated ksp to 1.5.31-1.0.1
- Several improvements to code generation which often means less code is generated.

### Added

- Multiple rounds handling: This includes support for using types generated by other ksp processors. As a side effect
  there is better error reporting for unresolved types.
- Support for multiplatform/native. Check out
  the [sample project](https://github.com/evant/kotlin-inject-samples/tree/main/multiplatform/echo).

  Note: components are thread-safe, however you will run into issues actually using them from other threads unless you
  enable the [new memory model](https://blog.jetbrains.com/kotlin/2021/08/try-the-new-kotlin-native-memory-manager-development-preview/).
- Added support for default args when injecting. If the type is present in the graph, it'll be injected, otherwise the
  default will be used.

  ```kotlin
  @Inject class MyClass(val dep: Dep = Dep("default"))

  @Component abstract ComponentWithDep {
      abstract val myClass: MyClass
      @Provides fun dep(): Dep = Dep("injected")
  }
  @Component abstract ComponentWithoutDep {
      abstract val myClass: MyClass
  }

  ComponentWithDep::class.create().myClass.dep // Dep("injected")
  ComponentWithoutDep::class.create().myClass.dep // Dep("default")
  ```

## [0.3.7-RC] 2021-10-29

### Changed

- Updated kotlin to 1.6.0-RC
- Updated ksp to 1.6.0-RC-1.0.1-RC
- Several improvements to code generation which often means less code is generated.

### Added

- Multiple rounds handling: This includes support for using types generated by other ksp processors. As a side effect
  there is better error reporting for unresolved types.
- Support for multiplatform/native. Check out
  the [sample project](https://github.com/evant/kotlin-inject-samples/tree/main/multiplatform/echo).

## [0.3.6] - 2021-07-16

### Changed

- Updated kotlin to 1.5.20
- Experimental kotlin js support

### Fixed

- Fix generated code for @Inject functions with a receiver ex: `@Inject fun Foo.bar() = ...`
- Fix not using the typealias for function return types

## [0.3.5] - 2021-06-02

### Changed

- Updated kotlin to 1.5.10
- Updated ksp to beta01

## [0.3.4] - 2021-05-30

### Fixed

- Fix metata parsing issue with kapt on kotlin 1.5.0
- Fix declaring function injection in another module in ksp

### Changed

- Updated kotlin to 1.5.0
- Updated ksp to alpha10

## [0.3.3] - 2021-04-20

### Added

- **Allow cycles when there is delayed construction**

  You can now break cycles by using `Lazy` or a function. For example,
  ```kotlin
  @Inject class Foo(bar: Bar)
  @Inject class Bar(foo: Foo)
  ```
  will fail with a cycle error, but you can fix it by doing
  ```kotlin
  @Inject class Foo(bar: Lazy<Bar>)
  @Inject class Bar(foo: Foo)
  ```
  or
  ```kotlin
  @Inject class Foo(bar: () -> Bar)
  @Inject class Bar(foo: Foo)
  ```
  This uses `lateinit` under the hood. You will get a runtime exception if you try to use the dependency before
  construction completes.
- Added option `me.tatarka.inject.dumpGraph` to print the dependency graph while building. This can be useful for
  debugging issues.
- **Allow type-alias usage with `@IntoMap`.**

  You can now do
  ```kotlin
  typealias Entry = Pair<String, MyValue>
  
  @Component {
      @Provides @IntoMap
      fun entry1(): Entry = "1" to MyValue(1)
      @Provides @IntoMap
      fun entry2(): Entry = "2" to MyValue(2)
  }
  ```

### Changed

- Code-gen optimization to reduce code size
- ksp performance improvements
- **Made handling of nullable and platform types consistent on the different backends.**

  It is now an error to return a platform type from a `@Provides` methods, you must declare the return type explicitly.

### Fixed

- Fix using `@Qualifier` on scoped dependencies
- Fix declaring components as an inner class
- Fix annotating java class constructors with `@Inject`
- Fix `@Inject` on a companion object

## [0.3.2] - 2021-04-05

### Changed

- Updated ksp to [1.4.30-1.0.0-alpha05](https://github.com/google/ksp/releases/tag/1.4.30-1.0.0-alpha05)

## [0.3.1] - 2021-02-25

### Changed

- Updated ksp to [1.4.30-1.0.0-alpha03](https://github.com/google/ksp/releases/tag/1.4.30-1.0.0-alpha03)
- Minimum supported kotlin version is now 1.4.30

## [0.3.0] - 2021-01-14

### Changed

- **Updated ksp
  to [1.4.20-dev-experimental-20210111](https://github.com/google/ksp/releases/tag/1.4.20-dev-experimental-20210111)**

  Key changes:
    - You no longer have to define `resolutionStrategy` in your `settings.gradle`.
    - The plugin id has changed from `symbol-processing` to `com.google.devtools.ksp`.
- Minimum supported kotlin version is now 1.4.20

### Added

- **Support injecting suspend functions**

  You can now define `suspend` component and provides methods
  ```kotlin
  @Component abstract class MyComponent {
    abstract val foo: suspend () -> IFoo

    val providesFoo: suspend () -> IFoo
        @Provides get() = { Foo() }
  }
  ```
- **Support default args in component constructors**

  If you define any default args, you will get an overload `create` function that provides default values. Due to
  processor limitations, you only get a single overload (i.e. you cannot pass defaults from some args but not other).
  This can be useful for more conveniently defining parent components.
  ```kotlin
  @Component abstract class MyComponent(@Component val parent: ParentComponent = ParentComponent()) {
    ...
  }
  val component = MyComponent::class.create()
  ```
- **Support annotating constructors with `@Inject`**

  Sometimes you don't want to use the primary constructor to construct an object. You can now annotate a more specific
  constructor instead.
  ```kotlin
  class MyClass {
    @Inject constructor(arg: String) // use this one for injection
    constructor(arg: Int)
  }
  ```
- **Support injecting objects**

  While you can use an object directly, it may be useful to inject it so that you can switch it to an instance at a
  later point without updating the consuming code.
  ```kotlin
  @Inject object Foo { ... }
  @Inject MyClass(dep: Foo) { ... }
  ```

### Fixed

- Respect component's class visibility
- Fix generating incorrect code for fun providers

## [0.2.0] - 2020-09-28

### Changed

- [Migrate](https://github.com/google/ksp/blob/master/old-ksp-release.md) ksp
- Improve kapt error messaging
- Build performance improvements

### Added

- Allow annotating interfaces with `@Component`
- Support `javax.inject.Qualifer`

### Fixed

- Fixed companion generation (`me.tatarka.inject.generateCompanionExtensions=true`) for ksp
- Throw error when parent component is missing val instead of generating incorrect code.

## [0.1.0] - 2020-09-17

- Initial Release
