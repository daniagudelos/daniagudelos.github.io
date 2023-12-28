---
title: "Signing Android flavors with different keys"
date: 2023-03-12T11:06:47+06:00
draft: false

# post thumb
image: "images/engineering/signing_android_flavors.jpg"
from: "yuri.samoilov.online"

# meta description
description: "A example on how to sign different flavors with different keys"

# taxonomies
categories: 
  - "Engineering"
tags:
  - "code"
  - "english"

# post type
type: "featured"
---

Do you have different android flavors and want to assign a different signing key to each one? This post is for you.

# An example problem

Imagine you have a mobile application which displays different shapes and color schemes according to the client. Each combination of shape and color scheme requires different signing keys.

If you are not familiar with flavoring, please read the
[Android Documentation](https://developer.android.com/studio/build/build-variants#product-flavors) first, so you can
understand the code.

<hr>

# Solution

1. Define the shapes and colors as `ArrayList<String>`
2. Define the flavor dimensions: `shapes` and `colors`
3. Create the shape flavors, iterating the list of shapes
4. Create the color flavors, iterating the list of colors
5. Create the signing configuration for each shape-color flavor combination
6. Assign a signing configuraation to each flavor. 

As an additional note, line shows how to filter flavor combinations out.


```go {linenos=table,style=syntax.css,linenostart=1}
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
}

def shapes = ["circle", "square", "rectangle"]
def colors = ["yellow", "blue", "green"]

android {
    namespace 'com.example.flavors'
    compileSdk 33

    defaultConfig {
        applicationId "com.example.flavors"
        minSdk 28
        targetSdk 33
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }

    flavorDimensions "shape", "color"

    shapes.forEach { shape ->
        productFlavors.create(shape).setDimension("shape")
    }
    colors.forEach { color ->
        productFlavors.create(color).setDimension("color")
    }

    buildTypes {
        release {
            shapes.forEach{shape ->
                colors.forEach{color ->
                    signingConfigs.create("$shape${color.capitalize()}${name.capitalize()}"){
                        storeFile file(System.getProperty("build.configRoot") + "/keystores/myApp_${shape}_${color}_$name.p12")
                        storePassword "password"
                        keyAlias "myApp_${shape}_${color}"
                        keyPassword "keyPassword"
                    }
                }
            }
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
        debug {
            shapes.forEach{shape ->
                colors.forEach{color ->
                    signingConfigs.create("$shape${color.capitalize()}${name.capitalize()}"){
                        storeFile file(System.getProperty("build.configRoot") + "/keystores/myApp_${shape}_${color}_$name.p12")
                        storePassword "password"
                        keyAlias "myApp_${shape}_${color}"
                        keyPassword "keyPassword"
                    }
                }
            }
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }

    variantFilter { variant ->
        def names = variant.flavors*.name
        if(name.contains("Yellow") && name.contains("circle")){
            setIgnore(true)
        } else {
            android.defaultConfig.signingConfig signingConfigs.getByName("${variant.name}")
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = '1.8'
    }
}

dependencies {

    implementation 'androidx.core:core-ktx:1.7.0'
    implementation 'androidx.appcompat:appcompat:1.4.1'
    implementation 'com.google.android.material:material:1.5.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.3'
    implementation project(path: ':shared')
    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.ext:junit:1.1.3'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.4.0'
}
```
 
<hr>

I hope that's helpful! :) 

