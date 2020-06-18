---
layout: page
title: Guide for teachers
permalink: /guide_teachers/
lang: en
categories:
    - en
---

## TMC account & organization

To create courses to [TMC](https://tmc.mooc.fi/) you need an account and an organization under which you’ll create your course. Register an account in TMC and follow [this guide](http://testmycode-usermanual.github.io/usermanual/teachers.html) to make a new organization if your school doesn’t already have one. If an organization under your school’s name already exists, ask a teacher who has access to it to give you access as well. You can see all the organizations on the TMC front page.

## Creating a course

Currently the only straightforward way for a non-admin to create a course is by using existing templates. If you just want to clone an existing course to teach it in your school, this is what you're looking for. To create a course from a template follow [this handy guide](http://testmycode-usermanual.github.io/usermanual/teachers.html#creating_a_course).
If you want to create your own exercises you need to create a custom course from a [Git](https://en.wikipedia.org/wiki/Git) repository. There is a [guide](http://testmycode-usermanual.github.io/usermanual/customcourse.html) to creating a custom course, but to get your repository added as a source for your course you should contact the TMC admins.

## Course repository structure

If you're creating a custom course, we'd suggest you structure your course repsitory something like this:

- *Root*
    - `part01`
        - `part01-01_ExampleExercise`
            - `src`
                - `Exercise`
                    - `Program.cs`
                    - `Exercise.csproj`
            - `test`
                - `ExerciseTest`
                    - `ExerciseTests.cs`
                    - `ExerciseTests.csproj`
        - `part01-02_AnotherExample`
            - `src`
                - `Exercise`
                    - `Program.cs`
                    - `AnotherFile.cs`
                    - `Exercise.csproj`
            - `test`
                - `ExerciseTest`
                    - `ExerciseTests.cs`
                    - `ExerciseTests.csproj`
    - `part02`
        - `...`


We recommend partitioning your course into logical parts, such as weeks. If you do this, you should make every part it's own folder for that part's exercises. In every exercise folder (`part01-02_AnotherExample` above, for example) you need to have a `src` folder and a `test` folder. 
The `src` folder will have all the code that the student edits. You can have the student create new files or just edit ones that already exist in the exercise template. The exercise templates will also contain the model solutions that won't be visible to the student before completing the exercise, but more on how those are implemented later. Note that you can have the code either in the root of the `src` folder or in a folder in the `src` folder (like the `Exercise` folder above) but if you have no code (i.e. `.cs` files) in any folder that is an immediate child of `src` (i.e deeper than one folder under `src`) the project won't be recongized as C# by the plugin!
The `test` folder will have your tests for the student's program. We suggest that you follow the naming convention used in the example above where the test `.csproj` file is named after the corresponding `.csproj` file. Note that the test `.csproj` file name **must** end in `...Tests.csproj` or the test runner won't reconize them!

## Writing the tests

I'll put something here too

## Model solutions

This will be added soon
