The other weekend I created what soon Ncapsulate, a nuget wrapper for all things node.  Then I got to thinking what about GUI?  Right now there is not good GUI for the tools that Ncapsulate exposes.  We integrate very well with MSBuild, but what if we have a build task that only needs to happen periodically?  Maybe you have a task that automates the update process, for example it will locate and find the latest version of all the bower packages, and then run through the update process for each package, and then rerun your unit tests.  Typically you wouldn't want to do this as part of the build, but it is a task that you want to be able to run, and you infact want everyone to be able to run easily.


#### Grunt Launcher
This is where [Grunt Launcher][gruntlauncher] comes into play.  This is a wonderful tool, that keeps an eye on your gulpfile, gruntfile, bower dependencies and more.  I am mainly going to focus on the Grunt and Gulp parts of this tool
for this post, but please [take a look][gruntlauncher] at what else the tool offers.


After looking at the Grunt Launcher [source][gruntlaunchersource] I was able to determine that Ncapsulate should be entirely compatible with how Grunt Launcher runs the commands from the command line.  Ncapsulte creates a .cmd file for each tool it supports, these include bower.cmd, grunt.cmd, gulp.cmd, etc.  These are a required part of the tooling that lets Ncapsulates magic work correctly, and they mimic the behavior seen if you were to be using `npm install -g`.  Grunt Launcher launches the command expecting that `grunt` or `gulp` will be in the `%PATH%` of the cmd. In addition Grunt Launcher launches from the root project directory of the project in Visual Studio.


What this leads to is that even if the command `grunt` is not in the `%PATH%`, but [Ncapsulate.Grunt] is installed.  When Grunt Launcher goes to run the grunt command, the Ncapsulate version of Grunt will be picked up and will execute the task, as if it were in `%PATH%`.

#### Does it work?
To demonstrate this, I removed all of the npm `.cmd` files from my machine temporarily, rather I ziped them up and then deleted them.

![npm files removed and attempt to run "grunt"](/public/grunt-launcher-ncapsulate/2014-04-24 10_05_36-Windows PowerShell.png)


Then I installed Grunt Launcher into my Visual Studio instance, and loaded up [Ncapsulate.Example] to see what, if anything happens.  Ncapsulate Example is the test project I used to verify most of the initial functionality of Ncapsulate, and it has bower, grunt and gulp installed as packages.

#### Bower
![Bower Update All](/public/grunt-launcher-ncapsulate/2014-04-24 10_19_28-master • NCapsulate.Example - Microsoft Visual Studio.png)
After loading up the project, and right-clicking on bower_components I'm presented with a new menu item.

![Bower Update](/public/grunt-launcher-ncapsulate/2014-04-24 10_22_22-master • NCapsulate.Example - Microsoft Visual Studio.png)
For each different bower component, we get an appropriate item in the context menu.

![Bower Update result](/public/grunt-launcher-ncapsulate/2014-04-24 10_21_45-master • NCapsulate.Example - Microsoft Visual Studio.png)
And the bower commands even run as expected!


#### Grunt
![Grunt](/public/grunt-launcher-ncapsulate/2014-04-24 10_24_47-master • NCapsulate.Example - Microsoft Visual Studio.png)
Under the grunt submenu there is a list for each grunt command

![Grunt result](/public/grunt-launcher-ncapsulate/2014-04-24 10_25_22-master • NCapsulate.Example - Microsoft Visual Studio.png)
In Ncapsulte example the grunt file just registeres an empty task, but it does run as expected.


#### Gulp
![Gulp](/public/grunt-launcher-ncapsulate/2014-04-24 10_26_37-master • NCapsulate.Example - Microsoft Visual Studio.png)
Exactly like Grunt we see all the commands for Gulp under there.

![Gulp result](/public/grunt-launcher-ncapsulate/2014-04-24 10_27_10-master • NCapsulate.Example - Microsoft Visual Studio.png)
The output here is exactly what we should expect from the given task.


#### Summary
Grunt Launcher is yet another tool that we can use to allow developers to not leave their comfort zone of the Visual Studio IDE.  While also making sure that each developer does not need to install node and grunt or gulp on their local machine.


[gruntlauncher]: http://visualstudiogallery.msdn.microsoft.com/dcbc5325-79ef-4b72-960e-0a51ee33a0ff
[gruntlaunchersource]: https://github.com/Bjornej/GruntLauncher
[Ncapsulate.Grunt]: https://www.nuget.org/packages/Ncapsulate.Grunt/
[Ncapsulate.Example]: https://github.com/Blacklite/Ncapsulate.Example
