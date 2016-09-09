---
layout: post
title:  "Build Spring Boot Sample Code"
date:   2015-12-23 21:28:31 +0900
categories: tech
---

### Build Spring Boot Sample Code

* For build and test the sample code in https://spring.io/guides

git clone the guide project, say "Validating Form Input" https://spring.io/guides/gs/validating-form-input/

Go to the folder `complete` and build:
`gradle build`

Will out to build/libs/program.jar

And then run it:
`java -jar build/libs/program.jar`

* For generate empty gradle project:

Generate sample spring project use: `http://start.spring.io/`

Select `Gradle` to generate the project.
