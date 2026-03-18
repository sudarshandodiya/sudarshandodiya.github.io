## Glossary

Before we dive into the topic, let's first understand some of the terminology
that we will be using throughout this post.

- **Gradle** is a build automation tool that is used to build, test, and package
  software written in Java, Kotlin, Groovy, Scala, or any other JVM language. In
  layman's terms, just think of it as a wrapper around the language compiler
  that adds some extra functionality to make it easier to build and test your
  software.
- **Dependencies** are the libraries that your project depends on. Think of
  these as a set of useful code bundled up together that your project can use,
  instead of rewriting it everytime it is required.

## Introduction to Version Catalogs

Generally, in a Gradle project, when you want to use a dependency, you need to
define the following things:

1. The group of the dependency
2. The name of the dependency
3. The version of the dependency

For example, if you want to use the OkHttp library, you would define the
following:

```kotlin
dependencies {
    implementation("com.squareup.okhttp3:okhttp:4.9.3")
}
```

Now, if you want to use the same library in another module of your project, you
will need to make sure that you are using the same version of the library, or
you might face compatibility issues.

This can still be resolved by defining the version of the dependency in your
project's `gradle.properties` file, like this:

```properties
okhttpVersion=4.9.3
```

and then referencing it in your `build.gradle.kts` file:

```kotlin
dependencies {
    private val okhttpVersion: String by project
    implementation("com.squareup.okhttp3:okhttp:$okhttpVersion")
}
```

This is a good solution, and does solve our issue to a certain extent, but once
you have to manage multiple projects, it becomes quite cumbersome to make sure
that the versions are consistent across all the projects. Let's take a scenario
where:

1. You have a set of functions that provide useful abstractions over the OkHttp
   library, and you use them in multiple projects.
2. To make it easier to manage these sets of functions, you decide to create a
   separate project that contains the functions that are common across all the
   projects, and you name the library `common`. This library uses OkHttp version
   4.9.3 as its dependency.
3. In your project, you import the `common` library, but instead of using the
   version 4.9.3 that you are using in the `common` library, you use the version
   4.10.0.

The given scenario might lead to the following issues:

- The `common` library might not be compatible with the version of OkHttp that
  you are using in your project, which results in either a compilation error or
  a runtime error.
- You might not be aware of the OkHttp version that you are using in your
  `common` library, and assume it's using the same version as your project,
  which might not be the case. This can lead to you not realizing that you are
  using an outdated version of OkHttp, which can result in security
  vulnerabilities in your code.
- Every time you update the OkHttp version in your `common` library, you need to
  update it in all the projects that use the library, which can be a tedious and
  time-consuming process.

Here comes **Version Catalogs** to the rescue.

> A version catalog is a selected list of dependencies that can be referenced in
> a Gradle build script in a type-safe manner.

To explain it in developer terms, think of it as a file where you mention all
the strings required by the project, so that instead of mentioning the same
string again and again, you can just reference it from that file, with
auto-completion support as icing on the cake. It also helps that every time you
need to update that string, you only need to do it in one place, instead of
updating it in multiple places.

## Using Version Catalogs in Gradle

Version catalog is defined in a file called `libs.versions.toml` in the `gradle`
directory of your project. The `libs.versions.toml` file has four sections:

- **[versions]**: Declare versions of the dependencies. If you have a set of
  libraries which belong to the same group, instead of mentioning versions for
  them separately, you can define them in a single place, and reference it for
  all the libraries for which it is applicable.
- **[libraries]**: Here you can define the group, artifact, and version of the
  libraries that you are using in your project. For version, you can either
  mention the version directly, or you can reference a version from the
  `[versions]` section.
- **[bundles]**: If you have a set of libraries that are commonly used together,
  you can define them as a bundle, and import the bundle in your project. Gradle
  will check which libraries belong to that bundle, and import them all.
- **[plugins]**: Define plugins that you are using in your project. This is
  different from the `[libraries]` section, because plugins are not libraries,
  and instead of consisting of a group, artifact, and version, they consist of
  an id and a version.

An example `libs.versions.toml` file would look like this:

```toml
[versions]
okhttp = "4.9.3"

[libraries]
okhttp-core = { group = "com.squareup.okhttp3", name = "okhttp", version = "4.9.3" } # mentioning the version directly
okhttp-logging = { group = "com.squareup.okhttp3", name = "logging-interceptor", version.ref = "okhttp" } # referencing the version from the okhttp version

[bundles]
okhttp = ["okhttp-core", "okhttp-logging"]

[plugins]
kotlin-jvm = { id = "org.jetbrains.kotlin.jvm" , version = "1.9.20" }
```

To use the version catalog in your project, you need to add the following lines
to your `build.gradle.kts` file:

```kotlin
plugins {
    alias(libs.plugins.kotlin.jvm) // to use the kotlin-jvm plugin
}

dependencies {
    // You can either import the libraries separately
    implementation(libs.okhttp.core)
    implementation(libs.okhttp.logging)
    // Or as a part of a bundle
    implementation(libs.bundles.okhttp)
}
```

## Publishing Version Catalogs

So far, we have seen how to define version catalogs in a `libs.versions.toml`
file, and how to use them in a Gradle project. However, this still does not
solve the inherent problem of managing dependencies across multiple projects.

Fortunately, you can publish your version catalog, just like any other library!
To do that, you need to import the following plugins:

```kotlin
plugins {
    `version-catalog`
    `maven-publish`
}
```

This will allow you to configure your version catalog, and configure
`maven-publish` plugin to publish the catalog, like this:

```kotlin
catalog {
   versionCatalog {
       // Import the version catalog from the libs.versions.toml file
       from(files("${rootProject.projectDir}/gradle/libs.versions.toml"))
   }
}

publishing {
    publications {
        create<MavenPublication>("maven") {
            from(components["versionCatalog"])
            artifactId = "version-catalog"
            group = "com.yourgroup"
            version = "1.0" // or a version from a configuration file
        }
    }
}
```

Now, along with your `common` library, you can import the version catalog in
your projects, and use the same version of the OkHttp library that you are using
in the `common` library.

Add this to your `settings.gradle.kts` file:

```kotlin
dependencyResolutionManagement {
    versionCatalogs {
        create("libs") {
            from("com.yourgroup:version-catalog:1.0")
        }
    }
}
```

Voilà! Your version catalog is ready to be used in your project:

```kotlin
dependencies {
    implementation(libs.okhttp.core)
    implementation(libs.okhttp.logging)
}
```

Now, every time you update the OkHttp version in `common` library, you only need
to update your library version in the project, and the OkHttp version will
automatically be updated!

### But what if you want to use a library, that is not a part of version catalog?

In that case, you can customize the version catalog at the project level to
include the library that you want to use:

```kotlin
dependencyResolutionManagement {
    val log4jToSlf4jVersion: String by settings
    versionCatalogs {
        create("libs") {
            from("com.yourgroup:version-catalog:1.0")
            library("log4jToSlf4j", "org.apache.logging.log4j:log4j-to-slf4j:$log4jToSlf4jVersion") // log4jToSlf4jVersion is defined in gradle.properties
        }
    }
}
```

Later you can add this library to your version catalog if you want this to be
used across projects.

### Can we publish the common library as a part of the version catalog?

Since the `common` library is getting published along with the version catalog,
we would need to make sure that the library version is updated in
`libs.versions.toml` with every release.

To avoid this, you can define the `common` library not as a part of
`libs.versions.toml`, but as a part of the version catalog Gradle plugin, like
this:

```kotlin
catalog {
   versionCatalog {
       // Import the version catalog from the libs.versions.toml file
       from(files("${rootProject.projectDir}/gradle/libs.versions.toml"))
       library("common", "com.yourgroup:common:$version")
   }
}
```

Where you can dictate how the version is defined (environment variable,
property, etc.).

## Conclusion

We have seen how to define version catalogs, and how to use them in a Gradle
project. Instead of spending days updating each and every dependency manually
across projects, you can reduce the effort to just a couple of hours!!

Additionally, to reduce the time even more, you can use a plugin like
`version-catalog-update-plugin`, which will automatically update the version of
your dependencies to the latest version available that it can find. However, be
careful while using such plugins, as sometimes they might update the version to
an unstable one, which might not be desirable for your project.

Hope you found this post helpful!
