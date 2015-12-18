### What is out there right now
Today (Dec 2015) there are a few officially supported project types using project.json to describe package dependencies officially.

These project types are UWP projects, Portable class libraries and ASP.NET 5 projects (class libraries, web applications and console applications).

These two can be divided into two:
1. Projects that rely on project.json as the main way to describe the build/deployment and packaging story
2. Projects that rely on project.json as the mechanism to define package dependencies (and perhaps later packaging), but rely on a csproj (or vbproj/fsproj) to define the rest of the build requirements.

### What is this document about
Other project types that do not yet officially support project.json can still benefit from the advantages project.json  brings. This is about how to get from a project with a packages.config file to one with project.json to enjoy:

1. No changes to csproj files (preventing merge conflicts)
2. No need to run nuget update (by using * notations in dependencies)
3. Transitive restore (allowing defining just top level dependencies)
4. P2P dependency flow (allowing for a top level app to always pick up dependencies from class libraries it depends on, without explicitly installing them).

### Getting started


### What doesn't work yet
Project types that's rely heavily on content packages and xdt transforms, won't work with project.json at the moment. The biggest ones are the MVC/Web API/Web Forms applications that pull in dependencies like jQuery, or expect nuget packages to modify web.config automatically for them.

For these project types you can move to using bower in combination with grunt/gulp.

### 3rd part projects/blogs related to this
A conversion script - https://github.com/wgtmpeters/nugetprojectjson
Marc Gravell's blog on DNX - http://blog.marcgravell.com/2015/11/the-road-to-dnxpart-3.html

