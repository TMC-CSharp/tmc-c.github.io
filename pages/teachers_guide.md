---
layout: page
title: Guide for teachers
permalink: /guide_teachers/
lang: en
categories:
    - en
---

## TMC account & organization

To create courses to [**TMC**](https://tmc.mooc.fi/) you need an account and an organization under which you’ll create your course. Register an account in TMC and follow [**this guide**](http://testmycode-usermanual.github.io/usermanual/teachers.html) to make a new organization if your school doesn’t already have one. If an organization under your school’s name already exists, ask a teacher who has access to it to give you access as well. You can see all the organizations on the TMC front page.

## Creating a course

Currently the only straightforward way for a non-admin to create a course is by using existing templates. If you just want to clone an existing course to teach it in your school, this is what you're looking for. To create a course from a template follow [**this handy guide**](http://testmycode-usermanual.github.io/usermanual/teachers.html#creating_a_course).
If you want to create your own exercises you need to create a custom course from a [**Git**](https://en.wikipedia.org/wiki/Git) repository. There is a [**guide**](http://testmycode-usermanual.github.io/usermanual/customcourse.html) to creating a custom course, but at the moment (as of 6/2020) you can't create a one by following it since only admins can add source repositories due to security risks. To get your repository added as a source for your course you should contact the TMC admins.

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

The exercises should be tested using [**xUnit**](https://xunit.net/). 

The tests should have their own .csproj file that contains the following imports:

```csproj
  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="16.8.0"/>
    <PackageReference Include="TestMyCode.CSharp.API" Version="1.1" />
    <PackageReference Include="xunit" Version="2.4.1" />
    <PackageReference Include="xunit.runner.visualstudio" Version="2.4.3" />
  </ItemGroup>
```
and the test class itself should include:

```csharp
using Xunit;
using TestMyCode.CSharp.API.Attributes;
```

Tests are written in normal xUnit fashion, each test containing an xUnit attribute, such as ```[Fact]```, and one or more Assert-statements. In addition, tests can contain a ```[Points("number")]``` attribute, which denotes the amount of points gained in TMC from passing said test. For example:

```csharp
[Fact]
[Points("1")]
public void DummyTest() {
    Assert.True(true);
}
```

The test class itself can also contain a Points-attribute, which should be placed directly above the class declaration. These points are awarded when all tests in that class are passed.

### Testing whether a class or method exists (with reflection)

For an alternative more straightforward way to test for missing classes or methods, refer to *Testing whether a class or method exists (with the stub generation library)* below.

Testing whether a class or method exists can be done using [**reflection**](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/reflection). For this your test class should include two string variables, one containing the name of your exercise's namespace and the other containing the name of the assembly file generated by your project (this is the project name by default). For an example, if the exercise project is called Exercise1 and it's in the namespace "Exercise", you would include variables like

```csharp
private const AssemblyName = "Exercise1";
private const @namespace = "Exercise";
```

Reflection features must also be added to use with ```using System.Reflection;```.

An example method testing if a class called ```CustomClass``` exists:

```csharp
[Fact]
[Points("1")]
public void TestCustomClassIsCreated()
{   
    string className = "CustomClass";
    Type ClassType = Type.GetType($"{@namespace}.{className},{AssemblyName}");
    Assert.NotNull(ClassType);
}
```

An example method testing if a method called ```CustomMethod``` exists in class ```CustomClass``` (this could also be the main class):

```csharp
[Fact]
[Points("1")]
public void TestMethodExists()
{
    string className = "CustomClass";
    string methodName = "CustomMethod";
    Type ClassType = Type.GetType($"{@namespace}.{className},{AssemblyName}");
    Assert.NotNull(ClassType);
    MethodInfo info = ClassType.GetMethod(methodName);
    Assert.NotNull(info);
}
```

These methods can be used to test the existence of any classes or methods by just changing the relevant strings.

### Stub generation library

There is an additional library that can be added to use in the tests' .csproj file that will generate error throwing stubs in place of missing classes or methods. This has the benefit of avoiding visible errors in the test classes caused by missing methods or classes and the error messages given to the programmer can be easier to understand. To use the library, simply add:

```csproj
<PackageReference Include="CodeExerciseLibrary.SourceGenerator" Version="1.2.1" />
```
to the .csproj file's ItemGroup containing other packages.

The library doesn't require any further setup and works on it's own. **Magic!**

### Testing whether a class or method exists (with the stub generation library)

Testing whether a class or method exists can be made more straightforward with the stub generation library. 

When the library is added, you can simply try to create an instance of a class or call a method. If the class or method does not exist, the library throws a [**NotImplementedException**](https://docs.microsoft.com/en-us/dotnet/api/system.notimplementedexception) and the test will fail. Thus using reflection or more complex testing methods is not needed.

**Note!**
Remember to reference the exercice namespace with the `using` keyword in order for the students classes to be picked up! Otherwise the classes are not found and stubs will be generated.

Also, static field and method access (includes enums!) requires the use of the `partial` keyword on the test class.

## Model solutions and stubs & repository access

You can (and should) add model solutions and stubs to your exercises by using the default TMC way detailed in [**the guide**](http://testmycode-usermanual.github.io/usermanual/customcourse.html#writing_the_exercise).

```csharp
using System;

namespace TestProject
{
    public class Program
    {
        public static void Main(string[] args)
        {
	    // BEGIN SOLUTION
            Console.WriteLine("Hello world");
	    // END SOLUTION
            
	    // STUB: Console.WriteLine("Hello ");
        }
    }
}
```

The server will parse the stub version to the exercise template given to the student and give them access to the model solution once they have comleted the exercise. Note that this means that you should make your course repository **private** since crafty students could otherwise find the model solutions in GitHub. You then need to give repository access to [**tmc-deploy**](https://github.com/tmc-deploy) for the server to function properly.

## The use of NuGet packages

The NuGet packages are **not** fetched from the Internet when exercise tests are ran! The server runs the tests inside a docker container which has locally cached packages and no access to the Internet. The list of available packages can be found [**here**](https://github.com/rage/tmc-sandbox-images/blob/master/csharp/NuGetDownloader/config.json). If you would like to use newer versions of specific packages or a new package altogether, you are welcome to open a pull request to expand the list!
