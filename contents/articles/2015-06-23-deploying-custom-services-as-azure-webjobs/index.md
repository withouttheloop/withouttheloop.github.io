---
title: Deploying custom services as Azure Webjobs
author: Liam McLennan
date: 2015-06-23 20:32
template: article.jade
---

Azure has this thing called [webjobs](http://www.hanselman.com/blog/IntroducingWindowsAzureWebJobs.aspx). It is a curious way of deploying continous or scheduled services as sub-processes of an Azure website. This may be a desirable option because:

* it's a PaaS thing. You don't have to manage a server.
* you get configurable scale - including autoscaling
* you can run many logical services within a single management unit (Azure Website)

I did not enjoy my time with the MSDN documentation for Azure Webjobs. They document the manual process for working with Webjobs, either the Visual Studio Azure tooling or through the Azure portal. Manual deployment processes, involving clicking lots of buttons with a mouse, a not going to be workable on my team, so I need a way to automate deployment. Also, the documentation only shows how to write services that are coupled to Azure. I want to be able to run my service locally. 

Step 1: Use a console app for your service
--------------------

The holy grail of .net services has been wrapping console apps for a long time, often using something like Topshelf. The goal is to get the reliability of a service for deployment but the ease of development and testing of a console app. 

To create a console app for deployment as an Azure Webjob:

* create a normal, regular, vanilla, wonderful console application.
* create a web project
* in the web projects context menu choose Add -> Existing project as Azure webjob and select your console application

Step 2: Packaging for deployment
-----------------------------

The secret to Azure Webjob deployment is to include your executable in the web project at:

> App_Data\jobs\\{job type}\\{job name}

That is the one bit of information to really need to know and it is conspicuously absent from the documentation. 

<img src="shake.gif"/>

There are a number of different ways to accomplish this goal. One is to use a post-build step like:

```
copy "$(SolutionDir)mywebjob\bin\$(ConfigurationName)\*.*" "$(ProjectDir)App_Data\jobs\continuous\mywebjob"
```

which you can add via the `Build Events` tab of the web project's properties page. 

Because we are using Teamcity to build and package our web project as a nuget package for deployment via Octopus Deploy we can use a nuspec file instead. 

A simple nuspec file looks something like:

```
<?xml version="1.0"?>
<package >
  <metadata>
    <id>MyProjectName</id>
      <version>0.0.0.0</version>
      <authors>NA</authors>
    <owners>NA</owners>
    <requireLicenseAcceptance>false</requireLicenseAcceptance>
    <description>NA</description>
    <copyright>Copyright 2015</copyright>
  </metadata>
    <files>
        <file src="..\MyWebJob\bin\release\**" target="App_Data\jobs\continuous\MyWebJob"/>
    </files>
</package>
```

Using Octopack in Teamcity to produce a nupkg package from this nuspec file will fail because the `files` element will override the default behaviour of Octopack and only include the listed files. To instruct Octopack to include all the normal project files, as well as the webjob files listed in the nuspec file, add the following element to the web project's csproj file (in the `<PropertyGroup>` element):

```
<OctoPackEnforceAddingFiles>True</OctoPackEnforceAddingFiles>
```

This seems to be an undocumented feature, but it works. 

Step 3: Deploy
-------

Deploy the website to Azure Websites using your favourite method (my family likes to use Octopus Deploy 3 every day). Check in the Azure portal and you will hopefully have a Webjob listed in your Web App. If not, you can connect the Visual Studio Azure tooling to your Azure subscription and use the Service Explorer (App Service node) to see if the files have been deployed as expected. 