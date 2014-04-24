The other weekend I created what soon Ncapsulate, a nuget wrapper for all things node.  Then I got to thinking what about GUI?  Right now there is not good GUI for the tools that Ncapsulate exposes.  We integrate very well with MSBuild, but what if we have a build task that only needs to happen periodically?  Maybe you have a task that automates the update process, for example it will locate and find the latest version of all the bower packages, and then run through the update process for each package, and then rerun your unit tests.  Typically you wouldn't want to do this as part of the build, but it is a task that you want to be able to run, and you infact want everyone to be able to run easily.

This is where [Grunt Launcher](gruntlauncher)

[gruntlauncher]: http://visualstudiogallery.msdn.microsoft.com/dcbc5325-79ef-4b72-960e-0a51ee33a0ff
