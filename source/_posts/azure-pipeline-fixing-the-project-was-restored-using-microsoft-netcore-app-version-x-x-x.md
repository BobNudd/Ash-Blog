---
  title: Azure Pipeline- Fixing ‘The project was restored using Microsoft.NETCore.App version x.x.x
  date: 2019/05/29 15:07:00
  tags: c#, devops
---


I couldn’t find an answer for this, so figured a quick post may help someone out on this issue since people are experiencing this.

Initially despite best practice, explicitly setting a version for the Microsoft.AspNetCore.Net package resolved a pipeline library confliction issue, where it could not determine the correct package to restore on the Pipeline side.

Then came the following host of errors:

`Error : NETSDK1061: The project was restored using Microsoft.NETCore.App version 1.0.0, but with current settings, version 2.2.0 would be used instead. To resolve this issue, make sure the same settings are used for restore and for subsequent operations such as build or publish. Typically this issue can occur if the RuntimeIdentifier property is set during build or publish but not during restore. For more information, see https://aka.ms/dotnet-runtime-patch-selection.`

Some people resolved this issue, by specifying their projects to target the latest runtime, however in my scenario, apparently the pipeline still wanted to restore using 1.0.0

### Solution(s)
If you’re using the default generated YAML build script, quite simply despite the pipeline *detecting* your project type and building you a generic script, this doesn’t work.

You’ll want to replace

`- task: NuGetCommand@2
  inputs:
    restoreSolution: '$(solution)'
with:`

`- task: DotNetCoreCLI@2
  inputs:
    command: 'restore'
    projects: '**/*.csproj'`

Now yes, if you have many projects in your solution this likely would add a good amount of time to your build, but that’s not an issue for myself.

#### Even Easier Solution:

![](/post/azure-pipeline-fixing-the-project-was-restored-using-microsoft-netcore-app-version-x-x-x/classic-editor.png)

Using the classic editor, you’ll get a visual build task pipeline that should get you 90% of way there by default. Without changing anything this almost fully built my default SPA with .NET Core without changing many settings.

Honestly I feel a little disappointment in the new YAML builder, yes it’s nice to have a version controlled build script, but there are so many people experiencing issues, especially those new to .NET that shouldn’t have to spent hours on their vanilla .NET Core app building to a branch, or staging site.