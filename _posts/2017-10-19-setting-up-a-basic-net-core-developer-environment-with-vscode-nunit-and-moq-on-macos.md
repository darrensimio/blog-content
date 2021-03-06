---
ID: 127
post_title: >
  Setting up a Basic .NET Core Developer
  Environment with VSCode, xUnit and Moq
  on macOS
author: Darren Sim
post_excerpt: ""
layout: post
permalink: >
  https://darrensim.io/posts/setting-up-a-basic-net-core-developer-environment-with-vscode-nunit-and-moq-on-macos/
published: true
post_date: 2017-10-19 06:17:00
---
I've recently been kept awake late into the wee hours; working on really interesting code challenges on <a href="https://www.codewars.com/users/darrensimsg/">Code Wars</a>. While the problems are really fun to solve, one of the pains I found was to code on the web editor, which lacks the usual code editor features that I am comfortable with. Reluctant to load up the battery-draining Windows Virtual Machine just for this, I thought "<em>Why not let me just create a simple boiler plate project in .NET Core that supports xUnit, which I can readily work on using VSCode on my macOS?</em>"
<h3>.NET Core</h3>
<strong>What is .NET Core?</strong>

.NET Core is the cross-platform version of .NET Framework (at the layer of the Base Class Libraries) that is <a href="https://www.microsoft.com/net/core/support/">officially supported</a> my Microsoft, running on Windows, macOS, and Linux. Languages supported includes C#, Visual Basic and F#.

.NET Core is made up of FOUR primary components:
<ul>
 	<li><a href="https://github.com/dotnet/coreclr">.NET Core Runtime</a> (also known as CoreCLR)</li>
 	<li><a href="https://github.com/dotnet/corefx">Framework libraries</a> (also known as CoreFX)</li>
 	<li><a href="https://github.com/dotnet/cli">SDK tools</a> (made up of the Cli and Roslyn language compiler)</li>
 	<li>App Host, which is used to launch .NET Core Apps</li>
</ul>
<em>(I will eventually be doing a couple of deep dive post on .NET Core in the near future)</em><strong>What is the current version?</strong>

The current version of <em><strong>.NET Core is version 2.0</strong></em>, which was shipped on August 14, 2017. The next version of the framework <em><strong>.NET Core 2.1</strong></em> is expected to ship in the first quarter of 2018.

The easiest way to find out what version of .NET Core you have installed is to enter this command into your terminal <code>dotnet --version</code>. If terminal responds with something like <code>command not found: dotnet</code>, that probably means you do not have .NET Core installed, or it's PATH has not been setup.

<strong>Installing .NET Core 2.x on macOS</strong>

In order to install .NET Core 2.x on macOS, you need to be on macOS 10.12 "Sierra" or later versions. The best way to install .NET Core 2.x is to use the installer that's available for free, via <a href="https://www.microsoft.com/net/core#macos">https://www.microsoft.com/net/core#macos</a>.

The installer will install the latest stable version of .NET Core SDK along with the tools, setting up the necessary PATH, that makes it easy for you to simply run the <code>dotnet</code> command from the console.
<h3>Visual Studio Code</h3>
Many who have worked with Visual Studio would be left with the impression that anything Visual Studio will be "bloated" and "slow". Like many, I was hesitant to even install Visual Studio Code (also known as VSCode, a free and open sourced code editor by Microsoft) initially, passing it off as a glorified notepad app.

I eventually took the plunge; I've not had any regrets since! VS Code is amazingly lightweight, fast and powerful.

Based on the <a href="https://electron.atom.io/">electron framework</a>, VS Code is in essence, a Node.js app, powered by a large repository of largely community contributed <a href="https://marketplace.visualstudio.com/VSCode">add-ons</a> (called extensions). Based the <a href="https://github.com/Microsoft/monaco-editor">monaco editor</a> (an open source browser based code editor by Microsoft), VSCode comes with IntelliSense, git support, and debugging built-in.

VSCode is available for free, and runs on Windows, Linux and macOS.

<strong>Installing VS Code on macOS</strong>

The two most popular ways of installing VSCode on the macOS are via <code>brew cask install visual-studio-code</code>, or <a href="https://code.visualstudio.com/">simply downloading the executable</a>.
<h3>Creating a Basic C# Project with Unit Test</h3>
<strong>Create a new .net console project</strong><code>dotnet new console</code><strong>Resolve the build assets</strong><code>dotnet restore</code>
A required step in .NET Core 1.x, this step is not required starting with .NET Core 2.0.

Starting with .NET Core 2.0, <code>dotnet restore</code> executes automatically when a new project is created. It is also run implicitly as part of <code>dotnet build</code> or <code>dotnet run</code>.

<strong>Adding support for Debugging in VSCode</strong>
If you are lucky, Visual Studio Code will prompt you with the following message: "Required assets to build and debug are missing from 'c-sharp-quickstart'. Add them?". Click "Yes".

What this does, is that it creates two files (<a href="https://github.com/darrensimsg/c-sharp-quickstart/blob/master/.vscode/tasks.json">tasks.json</a> &amp; <a href="https://github.com/darrensimsg/c-sharp-quickstart/blob/master/.vscode/launch.json">launch.json</a>) in the <code>.vscode</code> folder. More information on what these two files does, can be found on the <a href="https://github.com/OmniSharp/omnisharp-vscode/blob/master/debugger-launchjson.md">Omnisharp help page</a>.

<strong>Add Unit Testing Frameworks</strong><code>dotnet add package xunit``dotnet add package moq</code>

Doing so will add packages references the latest versions of xUnit and Moq to your <code>.csproj</code> files; represented by the following configurations.

[code language="plain"]
[/code]

In order for xUnit to be used conveniently, an additional line of configuration needs to be added to the <code>.csproj</code> file as follows:

[code language="plain"]
```What we just did was to add a new .NET CLI Tool reference to dotnet-xunit, allowing us to execute the following command in the terminal.

[/code]

dotnet restore
dotnet xunit

[code language="plain"]

_NOTE: At the time of writing, nUnit does not support .NET Core out of the box._

### Github Repository

For my convenience, I have maintained a _**quick-start C# project**_ on Github, that allows me to get started right away simply by forking the repository. This is accessible via [https://github.com/darrensimsg/c-sharp-quickstart/](https://github.com/darrensimsg/c-sharp-quickstart/).

### Useful Keyboard Shortcuts

As a developer, I found keyboard shortcuts to have increased my productivity significantly. I've recently started taking notes on [keyboard shortcuts](http://darrensim.io/posts/mac-keyboard-shortcuts-for-developers/) that I consider useful; on top of my notes on [useful GIT commands](http://darrensim.io/posts/notes-on-useful-git-commands/).

### Further Readings

* [https://docs.microsoft.com/en-us/dotnet/core/tutorials/with-visual-studio-code](https://docs.microsoft.com/en-us/dotnet/core/tutorials/with-visual-studio-code)
* [http://asp.net-hacker.rocks/2017/03/31/unit-testing-with-dotnetcore.html](http://asp.net-hacker.rocks/2017/03/31/unit-testing-with-dotnetcore.html)
[/code]