# Visual Studio Docker Issue

Visual Studio Developer Community bug report can be found [here](https://developercommunity.visualstudio.com/t/Docker-launch-configuration-doesnt-work/1643628).

⚠ **'Docker launch configuration doesn't work when BaseOutputPath is outside of Docker context'**

Setting the `BaseOutputPath` to a directory that is outside of the Docker build context (the directory the `.sln` is in) will result in Visual Studio being unable to launch the project using `Fast` (or default) `ContainerDevelopmentMode`.

```xml
<PropertyGroup>
  <BaseOutputPath>$(MSBuildThisFileDirectory)../artifacts/bin/$(MSBuildProjectName)/</BaseOutputPath>
</PropertyGroup>
```

Changing `ContainerDevelopmentMode` to `Regular` will instruct Visual Studio to use the `Dockerfile` to build the container verbatim, instead of utilising the `Fast` mode. The auto-generated `Dockerfile` from the "Add Docker Support" context menu uses the `-o` option to specify an output file, which overrides the `BaseOutputPath` behaviour.

## Reproduction

Each commit in this repro represents a step in reproducing this issue.

1. Create a new minimal WebAPI project using `dotnet create webapi`
2. Open project in Visual Studio 2022
3. Right-click the project, navigate through "Add" and select "Docker Support"
4. Select "Linux" as a target platform
5. Add a `<BaseOutputPath>` which specifies a directory outside of the .sln
6. Attempt to launch using the Docker launch configuration

## Actual Result

Attempting to run the project using the Docker launch configuration will result in an error message.

![actual result](/img/actual-result.png)

## Expected Result

Docker container should launch properly.

**OR**

A more verbose build error could be shown earlier in the pipeline (when constructing the Fast-mode `Dockerfile`, for example)

## Workaround

Insert into the .csproj file:

```xml
<PropertyGroup>
  <ContainerDevelopmentMode>Regular</ContainerDevelopmentMode>
</PropertyGroup>
```

**Warning:** This will considerably slow down your build process when launching using the Docker launch configuration.

**OR**

Change your `BaseOutputPath` to a value that is contained within the Docker context.

```xml
<PropertyGroup>
  <BaseOutputPath>$(MSBuildThisFileDirectory)/bin/$(MSBuildProjectName)/</BaseOutputPath>
</PropertyGroup>
```
