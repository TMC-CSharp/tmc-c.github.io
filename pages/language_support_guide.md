---
layout: page
title: Language support guide
permalink: /guide_langsupport/
lang: en
categories:
    - en
---

## Preface

This guide was made by a group of students who were tasked to create a C# language plugin to TMC for the Software Engineering Lab course. We don’t have a deep understanding of the inner workings of TMC, nor do we claim to be expert coders. We created this guide mainly for anyone in a similar position: if you already understand enough tmc-langs and know how to make a test runner for your language, reading this might be a bit redundant. This guide will use our C# language plugin (and some other plugins) as reference, but keep in mind that our solutions to problems are far from perfect. This is very much a ‘how we did/interpreted it’ kind of guide.

## Introduction

### TMC-langs

![tmc-langs architecture](https://github.com/TMC-CSharp/tmc-csharp.github.io/blob/master/resources/arch.png)

Test My Code ([TMC](https://testmycode.github.io/)), as you probably already know, is a suite of tools used for teaching programming. TMC has many different moving parts, but the one we need to focus on is called [TMC-langs](https://github.com/testmycode/tmc-langs/). It’s a handy framework for TMC language plugin creation that makes adding support for new languages a lot easier. To develop a new language plugin you should fork the tmc-langs repository and start developing your plugin there, since it contains all the parts you need to change and it compiles into a package that you can swap with existing TMC-langs versions for easy testing & use.

![tmc-langs repository](https://github.com/TMC-CSharp/tmc-csharp.github.io/blob/master/resources/langs_repo.png)

Above you can see the structure of the TMC-langs repository. While the picture doesn’t include our C# plugin you can probably see that to add support for a new language you need to add a new module (written in Java) for your language to the list. More about what it’ll contain later.

### Test runner

![smaller scale architecture](https://github.com/TMC-CSharp/tmc-csharp.github.io/blob/master/resources/arch2.png)

The other part you need to develop in addition to your language plugin in TMC-langs is your test runner. The test runner is the application that actually runs the unit tests that test the student’s program. For this you need to select a unit testing framework for the language you are implementing, which will then be the test framework that the course instructors use to write the acceptance tests for exercises. For example in our project we chose xUnit and the Python plugin uses unittest, but these are of course language specific.

In the above diagram the area around the runner and the unit tests is marked as fuzzy. That is meant to highlight the fact that how your test runner will interact with the unit testing framework you choose can differ between languages and frameworks. The runner will need information about the test results but how the test framework reports those results to the runner depends on the framework.

The runner’s job after the testing has concluded is to parse the test results into a .json file that TMC-langs will read and then forward to other parts of TMC. Note that the only ways TMC-langs and the runner interact are TMC-langs’ invoking of the runner (via command using Java’s ProcessBuilder, nothing fancy) and the runner creating the results .json that TMC-langs then reads.

![json example](https://github.com/TMC-CSharp/tmc-csharp.github.io/blob/master/resources/jsonfile.png)

This also means that you have a greater freedom to implement the runner the way you see fit. While the language plugin has to inherit an abstract plugin class and should adhere to the style of other language plugins, the runner only needs to run the tests on the program (this can include compiling the program if your language is a compiled one) and produce the resulting normalized .json, the ‘how’ is up to you. Of course you should still make the runner simple and well written, especially considering performance and platform compatibility since TMC aims to make programming courses more accessible and scalable.

## Tmc-langs implementation

One of the two main components needed for TMC language support is the language-specific implementation of the tmc-langs -framework. Strictly speaking, only two classes need to be made for the tmc-langs side of language support to function: a Plugin and a StudentFilePolicy. However, it is advisable to also implement two more classes for parsing the two different result files the test runner can create, an ExerciseDescParser and a TestResultParser.

### Plugin

The Plugin class is the main component of any language support implementation. It’s a class that extends the framework’s AbstractLanguagePlugin (abstract) class, which in turn implements the LanguagePlugin interface. The plugin is responsible for all the major tasks expected of a language support implementation, such as recognizing a directory as a valid exercise directory for the language and running tests for a given exercise directory and returning the results. A number of methods have to be implemented in the Plugin class.

#### public boolean isExerciseTypeCorrect(Path path)

A method to check if the given path points to an exercise directory of this plugin’s type. For an example, in a CSharpPlugin the method must return true when the variable path points to a valid C# exercise directory and must return false in all other cases. Thus you must come up with a way to identify the plugin’s specific language’s exercise directories with no chance of falsely identifying other languages as the plugin’s language. 

The way this should be implemented is going to be highly language-specific, but some good methods are using Files.exists to check for specific directory names and Files.walk with a PathMatcher object to look for specific file extensions that are particular to the language. 

#### protected StudentFilePolicy getStudentFilePolicy(Path projectPath)

Returns a new StudentFilePolicy of the language plugin’s specific type. A one-liner method that must be implemented for the plugin to work. This is also the only place where the language-specific StudentFilePolicy class is referenced.

#### public String getPluginName()

Returns the language/plugin name as a string to be printed during execution. Another one-liner method with nothing notable to mention.

#### public Optional<ExerciseDesc> scanExercise(Path path, String exerciseName)

Produces an exercise description of an exercise directory, finding the test cases and the points offered by the exercise. This involves initializing the language-specific test runner into a ProcessRunner object and calling it with a specific command-line command to scan the exercise and write results into a json file. The generated json file must be parsed into a ImmutableList<TestDesc> instance, which is in turn used as a constructor parameter for a new ExerciseDesc object. This parsing of the json file is where you should use a separate language-specific ExerciseDescParser object. The method returns an Optional containing an ExerciseDesc on successful execution, or an Optional absent if the runner execution or json parsing failed.

#### public RunResult runTests(Path path)

Runs tests for an exercise directory and saves the results to a RunResult object. This involves initializing the language-specific test runner into a ProcessRunner object and calling it with a specific command-line command to run tests for the exercise and write results into a json file. The generated json file must be parsed into a RunResult object, or in the case of a failure, a RunResult object specific to that situation must be created. This parsing of the json file is where you should use a separate language-specific TestResultParser object. The method returns either a RunResult object parsed from the test results, or in the case of an exception in calling the runner, compiling the exercise or parsing the results, a specific RunResult object with a status and error logs.

#### public ValidationResult checkCodeStyle(Path path, Locale messageLocale)

A method for running checkstyle or a similar code style checking plugin to a project, if applicable. Returns an object that implements the ValidationResult interface of [tmc-langs-abstraction](https://github.com/testmycode/tmc-langs-abstraction). The method should return a new ValidationResult object with overrides for the getStrategy and getValidationErrors methods containing the results of code style checking. If no code style checking is used, getStrategy can be overridden to return Strategy.DISABLED and getValidationErrors to return a new empty HashMap. The documentation claims it is also fine to return null if no code style checking is used.

#### public void clean(Path path)

Used for cleanup such as deleting unneeded files after executing a task. This could mean the equivalent of executing ‘mvn clean’ for a Java project, or for instance deleting binaries generated by the test runner. With the tmc-langs command-line interface, clean is executed with a separate command ‘clean’.

### StudentFilePolicy

The StudentFilePolicy class is the only class besides the plugin that is truly required. It extends the abstract class ConfigurableStudentFilePolicy from the framework. The StudentFilePolicy class’ purpose is to identify which files are ‘student files’. According to the original documentation: “Student files are any files that are expected to be modified and/or created by the student. That is, any files that should not be overwritten when updating an already downloaded exercise and any files that should be submitted to the server.” In practice, only one method must be implemented in the class.

#### public boolean isStudentSourceFile(Path path, Path projectRootPath)

A method for determining whether the file specified by Path is a program source file. According to the official documentation: "A file should be considered a student source file if it resides in a location the student is expected to create his or her own source files in the general case."

### ExerciseDescParser

An optional but useful class dedicated to parsing the json file outputted by the test runner when called from the plugin’s scanExercise method. Parses the json result file into a list of TestDesc-objects, which are used to create an ExerciseDesc-object in the plugin's scanExercise method.

### TestResultParser

An optional but useful class dedicated to parsing the json file outputted by the test runner when called from the plugin’s runTests method. Parses the json result file into a RunResult object, which is the returned from the plugin's runTests method.
