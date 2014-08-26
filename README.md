Build System

1.  System's purpose
This paper describes a new automated system for project building. The system is intended primarily for software developers. The system is designed to construct and build projects on different operating systems and using different development environments.

2.  General principles of implementation
Current version of automated system for project  (Build Machine) has been built in accordance with the following principles:
Each project must have a Build Project File, containing the necessary information on the  of the project. For example, a unique name of the project, dependence on other projects and libraries, methods of , etc. Thus, the entire repository should be marked by the project description files to provide unambiguous information on the projects and their dependencies.
Build Machine must be able to perform different actions (project , external applications launch, file copying, etc.) which are needed to build a product. Build Machine should provide an opportunity to create flexible scripting actions on  projects. In this case , all operations with projects should be carried out on the basis of information that Build Machine derives from the project description files.
At the moment Build Project File uses am XML-file in the format described below. XML file is also used as a Job File.

3.Build Machine's command prompt
Right after its launch, Build Machine usually scans the repository and builds a full tree of projects with all their dependencies on the basis of Build Project File. Right after scanning the Build Machine starts to execute tasks that can be specified on the command line. 
The name for executable file for Build Machine is nbuilder.exe (for Windows).
Command line for Build Machinehas the following form:

nbuilder.exe [/build <Project ID> /output <Output Dir> [/config <Configuration>] [/target <Target>] [/platform <Platform>]]
             [/properties <Properties File>]
             [/pmanifest <Project Description File>]
             [/job <File with Tasks>]

Where
<Project ID> - is a unique name for the  project ( the name must be specified in the " Project description file")
<Output Dir> - the directory where all the files and dependencies of the constructed project will be copied
<Configuration> - project configuration. For example, Release, Debug
<Target> - building technology. For example, Build, Rebuild, Clean
<Platform> - «Win32», «x64», ...
<Properties File> - path to the environment variables. These variables can be used during the execution of tasks
<Project Description File> - path to the Project Description File. Usually this file contains information about other Project Description Files (which will be scanned before the execution of tasks).
<File with Tasks> - path to the file with a list of tasks to be performed
At start Build Machine performs the following actions:
Initialization of global variables (see List of global variables). 
Loading build.properties file, which is next to it. If the file is loaded, the program adds its variables into the general global variables. If build.properties file is not found , then Build Machine tries to load variables from build.properties.base file
Command line recognition
Loading variables from the additional «Properties» file specified on the command line ( key «/ properties»)
Build a list of tasks from Job File, specified on the command line ( key «/ job»)
Building of a list of tasks from the command line ( using key «/ build» in this case to build a project )
Repository will be scanned to build a tree of projects with all dependencies. Scanning directory will be defined as follows.  If the parameter «/ pmanifest» is specified, the scan will be conducted in the folder of the file (defined by «/ pmanifest»). If the value of the parameter «/ pmanifest» is not specified and the value of svn.repository.dir contains the directory, the scan will be carried out in this directory . Otherwise, Build Machine will start scanning Build Project File in the directory, where the executable is located
List of tasks is executed.

4. Set up Build Machine
The main way to configure Build Machine is to use environment variables from Properties file and script files.
As mentioned above, at its start Build Machine tries to load the list of variables from the files with already predetermined names («build.properties» and «build.properties.base»). These files generally contain the most common and important parameters.
The structure of Properties file is standard. It is a text file and each Property is set in a separate line. Here is an example:

# --------------------------------------------------
# My description
# --------------------------------------------------
repository.dir=”d:\svn\”
qt.dir=”D:\Qt\Qt5.1.0\5.1.0\msvc2010\”
qt.bin.dir=${qt.dir}bin\
msbuild.dir=”C:\Windows\Microsoft.NET\Framework\v4.0.30319\”
msbuild.exe=${msbuild.dir}MSBuild.exe
qt.linguist=${qt.dir}bin\linguist.exe

Where
# - symbol is the beginning of the comment line
${...} value for an already defined variable (see Usage of Variables). Besides, global variables set in the beginning of work with Build Machine, can also be used.

5. Create descriptions/definitions of the projects
Usually, Build Machine tries to collect all information about project before performing any task. This information is then used during the work with projects (for example, during building of the projects, library copying, projects' versions editing, …). To get information about the projects, Build Machine scans directories with initial code, finds the files with project descriptions (Build Project File) and builds a detailed project tree with its dependencies based on this information. Build Machine tries to get the following information for each project:
1. Type of the project. At present there are two kinds of projects:
«build» - resulting files of the project can be received only after the project has been built.
«files» - such a project contains ready-to-use files (there is no need to build such a project).
2. Ways of building. This parameter defines by what means the project will be built (for example, Microsoft Visual Studio, Qt Creator, Perl script, ...). The following ways of building have been put into realization by now:
«MSBuild» - project building using  Microsoft Visual Studio.
3. Name of the project. The name must be unique. 
4. Path to the main project file.
5. Path to the project directory.
6. List of all projects, which the project depends on. 
7. List of the files, which contain version numbers for the components of the project.
8. List of resulting files (which are created as a result of project building process or which are the components of the project, for example libraries).
9. List of modificators for the parameter, that indicated for which platform (x86, x64, …) the project has been built.
10. Filter. It is the text, which can be used during the search of projects within a text filter. Usually used for an extra classification of projects.
11. Comments.
12. Path to the file with project description.
Build Project File has an XML structure and an already defined name: ”pmanifest.xml”.
The XML file, which described the projects, can contain the following information:
List of variables.
Properties files with variables (will be loaded at the start of file processing with Project Desription File).
A list of directories, where Build Machine will search for other Build Project Files (in case Build Machine does not scan all folders recursively)
Descriptions of the projects.
A few words about how Build Machine scans directories to find the Build Project File. Build Machine supports three methods of scanning. When using the first method Build Machine just needs to specify the directory with source code. Build Machine recursively scans all subdirectories in the Build Project File search. When using the second method, Build Machine needs to specify a directory in which there is the Build Project File. Having read the found Build Project File, Build Machine will continue scanning only in case the Build Project File will contain any other directories for scanning. Exactly this method is used by default. The third method is the mix of first two methods (implemented partially).

5.1 XML structure of the document with project description
The main node of the document has the name “Project”. List of attributes for the node “project”.

Attribute name
Description 
Required
version
Default value: «1.0»
No

List of subnodes for “project”:

Node name
Description
Node number
vars
This node defines environmental variables. 
This node does not contain attributes. 
The node may comprise any number of nodes named “var”. “var” contains information about the variable.
Any
loadproperties
This node defines Properties files.
This node does not contain attributes. 
The node may comprise any number of nodes named “file”.  “file” contains information about the files, that should be loaded.
Any
scan
This node defines directories for further search of Build Project File.
This node does not contain attributes. 
The node may comprise any number of nodes named “targets“ (these nodes do not contain attributes).  In turn, “targets“ can contain any quantity of nodes with “target“ name. “Targets“ defines directories.
Not more than one
solutions
This node defines projects.
This node does not contain attributes. 
The node may comprise any number of nodes named “solution“. “solution“ contains information about the project.
Not more than one

Nodes “vars“, “loadproperties“ and “solutions“ are processed in the order they appear. Node “scan” is always processed first. Scanning in the directories, specified in “scan” can be made only after processing of the current file.
5.1.1 „Var“ nodes
„Var“ nodes contains following attributes 

Attribute name
Description 
Required
name
Name of variable
Yes
value
Value of the variable. 
Is an empty string by default. 
No
replace
If given as a “true”, “ok”, “yes” or “1”, then the value of the variable will be replaced by a value, defined in the “value” attribute.
If given as “false”, “no” or “0” and such a variable already exists, the global variable will not change. 
If given as “false”, “no” or “0” and there is no variable with such name, a new variable will be added.
Is “false” by default.
No

This section allows to add (change) global variables. 
In attribute values (“name”, “value”, “replace”) other variables can be used (see Usage of Variables).
5.1.2 „File“ nodes
„File“ node contains the following attributes

Attribute name
Description 
Required
file
Path to the Properties-file, from which the global variables will be loaded.
Yes
replace
If given as “true”, “ok”, “yes” or “1”, the variables from the file will replace the already existing variables.
Is “false” by default.
No
varprefix
Prefix added to the beginning of the name of each variable, loaded from the file. 
Is an empty string by default.
No

This section allows to add (change) global variables from Properties file.
You can use value of previously defined variables as attribute values (see  Usage of Variables ).
5.1.3 „Target” nodes
„Target” node contains the following attributes:

Attribute name
Description 
Required
path
Directory
Yes
enable
If given as “false”, “no” or “0”, the node will not be  processed
No

This section allows you to define the following directories in which Build Machine will scan project identifications (Build Project Files). 

You can use value of previously defined variables as attribute values (see  Usage of Variables ).
5.1.4 „Solution” nodes
Exactly this XML node defines the properties of the project used in Build Machine when working with the project.
“Solution” node has the following attributes

Attribute name
Description 
Required
name
Name of the project. Must be unique.
Yes
type
Type of project. One of the following values: 
„build“ - resulting project files can be obtained only after the  of the project.
„files“ - this project is a set of ready -to-use files (the  of such a project is not needed).
Yes
filter
This is the text that can be used when searching for projects in the text filter. Usually used for additional classification of projects. 
No
solutiondir
Path to the directory with the project.
No
projectfile
Path to the main project file.
No
enable
If given as “false”, “no” or “0”, this description is not to be used.
“True” by default
No

“Solution” node can contain a subsidiary “output”. “output” node defines the list of resulting files, that will be created after project building as well as already existing files. “output” node can contain any quantity of “fileset” nodes. “fileset” node contains the following attributes:
 
Attribute name
Description 
Required
dir
Files directory
Yes
id
Identificator, which identifies the group of files declared inside the “fileset”.
Build Machine allows to manipulate these files, finding them using their names or ID.
The usage of IDs lets you copy groups of files, specified in different “filesets” to different directories (see Build Task). 
No
condition
Expression written using variables, operators and functions.  If given as “false”, “no” or “0”, this description is not to be used.
No

List of files inside «fileset» if defined by attributes in “include” node. “Include”node has the following attributes

Attribute name
Description 
Required
name
Defines the name of the file or the relative path toe the file (relative to the directory, specified in the “dir” or “fileset” attribute).
If given as “*”, all the files from the directory, specified in the “dir” attribute of “fileset” node, will be used. 
Yes
targetsubdir
Subdirectory. If set, the system will always create this subdirectory in the output folder and place the output file into this subdirectory.
For example, it can be used when describing certain third party dlls, which must be placed in certain directories.
No

Dependencies project list is described in the “dependencies” node (inside “solution”). “dependencies” node includes “dependence”, which define the names of the dependency projects. “Dependence” node contains the following attributes: 

Attribute name
Description 
Required
name
Name of the project, which the described project depends on.
Yes

Version number is usually placed in a separate file (or various files). During build of the projects, there is often a need to change the version number in these files. To do this automatically, Build Machine must know where the version files are placed. Therefore, there is a subsidiary node “versionfiles” (inside “solution”). “versionfiles” node can contain any quantity of “versionfile” nodes, which define the path to the file with the version. “versionfile” node contains the following attributes: 

Attribute name
Description 
Required
file
Absolute or relative (relative to the directory, where the project description file is placed) path to the file with version.
Yes
id
ID which identifies this file. ID is helpful in those cases, when the file contains various files with version. Build Machine lets lets to manipulate with these files, finding them by the name or ID.
No

Some product projects are destined for different kinds of platforms, for example x86, x64, etc. Moreover, many projects can be built in different environment, in which the name of the same platform varies. This is why during the  of a specific project there is always a need to explicitly specify the name of the used platform. When building a release, many projects can be build as subsidiary (if they determine the project, launched for building). In this case these subsidiary projects will be build in the environment, that was created to build the original project. To “on-the-fly” correct the name of the platform, that is going to be used during the project build, a node “platformmappings” can be used (inside “solution”). “platformmappings” can contain any ammount of subnodes “platformmapping” - these nodes will correct the name of the platform. Node “platformmapping” contains the following attributes: 

Attribute name
Description 
Required
from
Platform name, which will be replaced by the value from the attribute “to”, if the “platform” setpoint  (from the  environment) coincides with the value of this attribute.
Can be an empty line.
Yes
to
Platform name, which will be used during the project build, if the specified “platform” value coincides with the “from” value.
Can be an empty line.
Yes

We recall that Build Machine build projects using “Build” command (see Build task). To determine the type of the platform this task has a property “platform”.

Note that you can use the values of already defined variables as values of attributes (see Usage of Variables ).

Example 1.
<?xml version="1.0" encoding="UTF-8"?>
<project version="1.0">
    <scan>
        <targets>
            <target path="3rdparty/curl-7.30.0/"/>
            <target path="3rdparty/openssl-1.0.1e"/>
            <target path="3rdparty/microsoft/x86/Microsoft.VC90.CRT"/>
            <target path="3rdparty/microsoft/x86/Microsoft.VC100.CRT"/>
            <target path="3rdparty/microsoft/x64/Microsoft.VC90.CRT"/>
            <target path="3rdparty/microsoft/x64/Microsoft.VC100.CRT"/>
            <target path="3rdparty/jansson"/>
        </targets>
        <targets>
            <target enable="ok" path="modules/module1"/>
            <target enable="ok" path="modules/module2"/>
            <target enable="ok" path="modules/module3"/>
        </targets>
        <targets>
            <target enable="ok" path="ui/application1"/>
            <target enable="ok" path="ui/application2"/>
            <target enable="ok" path="ui/application3"/>
            <target enable="ok" path="ui/application4"/>
            <target enable="ok" path="ui/application5"/>
        </targets>
    </scan>
    <solutions>
        <solution enable="ok" name="qt5.libraries.windows.ui" type="files" filter="thirdparty" >
            <output>
                <fileset dir="${qt.dir}bin/">
                    <include name="icudt51.dll" />
                    <include name="icuin51.dll" />
                    <include name="icuuc51.dll" />
                </fileset>
                <fileset dir="${qt.dir}bin/">
                    <namecorrectors>
                        <namecorrector configuration="debug" suffix="d"/>
                    </namecorrectors>
                    <include name="libEGL.dll" />
                    <include name="libGLESv2.dll" />
                    <include name="Qt5CLucene.dll" />
                    <include name="Qt5Concurrent.dll" />
                    <include name="Qt5Core.dll" />
                    <include name="Qt5Gui.dll" />
                    <include name="Qt5Widgets.dll" />
                </fileset>
                <fileset dir="${qt.dir}plugins/iconengines">
                    <include name="qsvgicon.dll" targetsubdir="plugins/iconengines"/>
                </fileset>
                <fileset dir="${qt.dir}plugins/imageformats">
                    <namecorrectors>
                        <namecorrector configuration="debug" suffix="d"/>
                    </namecorrectors>
                    <include name="qgif.dll" targetsubdir="plugins/imageformats"/>
                    <include name="qico.dll" targetsubdir="plugins/imageformats"/>
                    <include name="qjpeg.dll" targetsubdir="plugins/imageformats"/>
                    <include name="qmng.dll" targetsubdir="plugins/imageformats"/>
                    <include name="qsvg.dll" targetsubdir="plugins/imageformats"/>
                    <include name="qtga.dll" targetsubdir="plugins/imageformats"/>
                    <include name="qtiff.dll" targetsubdir="plugins/imageformats"/>
                    <include name="qwbmp.dll" targetsubdir="plugins/imageformats"/>
                </fileset>
                <fileset dir="${qt.dir}plugins/platforms">
                    <namecorrectors>
                        <namecorrector configuration="debug" suffix="d"/>
                    </namecorrectors>
                    <include name="qminimal.dll" targetsubdir="platforms"/>
                    <include name="qoffscreen.dll" targetsubdir="platforms"/>
                    <include name="qwindows.dll" targetsubdir="platforms"/>
                </fileset>
            </output>
        </solution>
        <solution enable="ok" name="qt5.libraries.windows.common" type="files" filter="thirdparty" >
            <output>
                <fileset dir="${qt.dir}bin/">
                    <namecorrectors>
                        <namecorrector configuration="debug" suffix="d"/>
                    </namecorrectors>
                    <include name="Qt5Core.dll" />
                    <include name="Qt5Gui.dll" />
                    <include name="Qt5Widgets.dll" />
                    <include name="libEGL.dll" />
                    <include name="libGLESv2.dll" />
                </fileset>
                <fileset dir="${qt.dir}bin/">
                    <include name="icudt51.dll" />
                    <include name="icuin51.dll" />
                    <include name="icuuc51.dll" />
                </fileset>
                <fileset dir="${qt.dir}plugins/platforms">
                    <namecorrectors>
                        <namecorrector configuration="debug" suffix="d"/>
                    </namecorrectors>
                    <include name="qwindows.dll" targetsubdir="platforms"/>
                </fileset>
            </output>
        </solution>
    </solutions>
</project>

Example 2.
<?xml version="1.0" encoding="UTF-8"?>
<project version="1.0">
    <solutions>
        <solution
            name="microsoft.vc90.crt"
            filter="thirdparty"
            type="files">
            <output>
                <fileset dir="" condition="!containsIgnoreCase(getBuildConfiguration(), 'debug')">
                    <include name="msvcm90.dll" />
                    <include name="msvcp90.dll" />
                    <include name="msvcr90.dll" />
                    <include name="Microsoft.VC90.CRT.manifest" />
                </fileset>
                <fileset dir="" condition="containsIgnoreCase(getBuildConfiguration(), 'debug')">
                    <include name="msvcm90d.dll" />
                    <include name="msvcp90d.dll" />
                    <include name="msvcr90d.dll" />
                    <include name="Microsoft.VC90.DebugCRT.manifest" />
                </fileset>
            </output>
        </solution>
    </solutions>
</project>

Example 3.
<?xml version="1.0" encoding="UTF-8"?>
<project version="1.0">
    <solutions>
        <solution
            name="MyApplication"
            filter="programs.cpp"
            type="build"
            solutiondir="."
            projectfile="MyApplication.sln">
            <versionfiles>
                <versionfile file="MyApplication/version.h" id="application"/>
                <versionfile file="MyHelper/version.h" id="helper"/>
            </versionfiles>
            <dependencies>
                <dependence name="qt5.libraries.windows.ui" />
                <dependence name="curl.windows" />
                <dependence name="openssl.windows" />
                <dependence name="microsoft.vc100.crt" />
            </dependencies>
            <platformmappings>
                <platformmapping from="" to="win32"/>
                <platformmapping from="x86" to="win32"/>
                <platformmapping from="Any CPU" to="win32"/>
                <platformmapping from="x64" to="win32"/>
            </platformmappings>
            <output>
                <fileset dir="../x86/Release" id="exe" condition="!containsIgnoreCase(getBuildConfiguration(), 'debug')">
                    <include name="MyApplication.exe"/>
                </fileset>
                <fileset dir="../x86/Release" id="dll" condition="!containsIgnoreCase(getBuildConfiguration(), 'debug')">
                    <include name="MyHelper.dll"/>
                </fileset>
                <fileset dir="../x86/Debug" id="exe" condition="containsIgnoreCase(getBuildConfiguration(), 'debug')">
                    <include name="MyApplicationd.exe"/>
                </fileset>
                <fileset dir="../x86/Debug" id="dll" condition="containsIgnoreCase(getBuildConfiguration(), 'debug')">
                    <include name="MyHelperd.dll"/>
                </fileset>
            </output>
        </solution>
        <solution
            name="MyApplication.x64"
            filter="programs.cpp"
            type="build"
            solutiondir="."
            projectfile="MyApplication.sln">
            <versionfiles>
                <versionfile file="MyApplication/version.h" id="application"/>
                <versionfile file="MyHelper/version.h" id="helper"/>
            </versionfiles>
            <dependencies>
                <dependence name="qt5.libraries.windows.ui.x64" />
                <dependence name="curl.windows.x64" />
                <dependence name="openssl.windows.x64" />
                <dependence name="microsoft.vc100.crt.x64" />
            </dependencies>
            <platformmappings>
                <platformmapping from="" to="x64"/>
                <platformmapping from="x86" to="x64"/>
                <platformmapping from="Any CPU" to="x64"/>
                <platformmapping from="win32" to="x64"/>
            </platformmappings>
            <output>
                <fileset dir="../x64/Release" id="exe" condition="!containsIgnoreCase(getBuildConfiguration(), 'debug')">
                    <include name="MyApplication.exe"/>
                </fileset>
                <fileset dir="../x64/Release" id="dll" condition="!containsIgnoreCase(getBuildConfiguration(), 'debug')">
                    <include name="MyHelper.dll"/>
                </fileset>
                <fileset dir="../x64/Debug" id="exe" condition="containsIgnoreCase(getBuildConfiguration(), 'debug')">
                    <include name="MyApplicationd.exe"/>
                </fileset>
                <fileset dir="../x64/Debug" id="dll" condition="containsIgnoreCase(getBuildConfiguration(), 'debug')">
                    <include name="MyHelperd.dll"/>
                </fileset>
            </output>
        </solution>
    </solutions>
</project>

6. Project 
To begin to build a project, it is necessary (as mentioned above) to perform the following: 
to have executables of Build Machine
to customize Build Machine
to describe necessary projects using Build Project File 
To build a separate project there is a possibility to use command line.
For example, you can use the following Windows command line to build “MyProject” project:

nbuilder.exe /build MyProject /output "d:/temp8" /config release /target rebuild

When building the project Build Machine uses full project tree with its dependencies. Build Machine sequentially builds projects, which affect the project, beginning with those which are independent. The project, which name has been specifies in the command line, will be build as the last one. All projects which depend on the mentioned project are build with same parameters (“configuration”, “target”, “platform”, “platformtoolset”, “output”, …) which are specified in the command line. All files got as a result of project  (including resulting files that don't require ) will be by default copied into the directory specified by the “output” command. You can change the settings for this copying task (see Build task). When building, Build Machine remembers all projects which it builds in order to exclude duplications of other projects with same parameters “configuration”, “platform”, “platformtoolset”. In addition, Build Machine remembers all directories, into which resulting files have been already copied, in order to exclude the possibility of duplications in the “output”. 
Incomparably more opportunities for project  provides the usage of scripts. Apart from tasks for project build you can also describe additional actions of almost any complexity. These options will be describes below. Now we are going to give an example of a script, which builds any project (this script displays a dialog box with project list and builds selected projects):

<project version="1.0">
    <vars>
        <var name="play.sound.when.complete" value="true" replace="true"/>
        <var name="show.message.when.complete" value="true" replace="true"/>
        <var name="build.properties.file" value="..\build.properties" replace="true"/>
    </vars>
    <tasks>
        <loadproperties file="${build.properties.file}" replace="true" varprefix=""/>
        <printvars/>
        <setvar name="programs.all" value="getAllBuildProjects()" asexpression="true"/>
        <dialog resultvar="dialog.result">
            <list caption="Project to build:" items="${programs.all}" default="" valuevar="projects.to.build" height="500" />
            <combobox caption="Build configuration:" items="Release,Debug" default="Release" valuevar="configuration"/>
            <combobox caption="Build target:" items="Build,Rebuild" default="Rebuild" valuevar="target"/>
            <selectdir caption="Output directory:" dir="d:\temp\Projects\Bin\" valuevar="output.dir"/>
        </dialog>
        <exit condition="isFalse(${dialog.result})"/>
        <foreach item="current.project" in="${projects.to.build}">
            <build
                name="#{current.project}"
                configuration="${configuration}"
                target="${target}"
                output="${output.dir}"/>
        </foreach>
    </tasks>
</project>

If this scenario can be saves in the folder with nbuilder.exe under the name “build_all.xml”, then it will be possible to execute the file using Windows command line: 

nbuilder.exe /job “build_all.xml”

7. Scripting
Nobody has ever been able to put all tasks about /assemble of project into realisation using a few parameters of the command line. Growing needs always claim to create possibilities for more flexible, scalable scripts to solve any problems.
Build Machine offers to use XML files (Job files) with a certain structure. These files will describe actions which the Build Machine must perform when processing files. 
Such files are launched for processing via command line, where the file name will be specified by parameter “/job”:

nbuilder.exe /job “my_tasks.xml”

There are no restrictions for the name of “Job File”.
In fact, the contents of “Job File” is the text (source code) of the program, performed by Build Machine. To make the possibilities of this file sufficient, an option to work with global and local variables have been implemented in Build Machine along with standard set of “functions” (tasks), an option to create subprograms (“macrodef”) and a build-in interpreter.
Job File allows you to do the following:
To specify an optional list of variables and change the values of already existing variables.
To include other “Job Files”. There is an option to modify the variables and their names by adding prefixes.
To include variables by “Properties files” (with an option of modification)
To set subprograms/templates for repetitive tasks. Templates can use other previously determined templates (also from other files). Nesting level is not limited.
To define any number of actions (tasks) to be performed. Any atomic task must be represented as a properly decorated XML section. 
While performing the script Build Machine sequentially processes the elements of the XML document. Processing takes place “in a single pass”.
This is the simplest example of the file, after processing of which a message “Hello, World!” will be displayed:

<project version="1.0">
    <vars>
        <var name="var1" value="Hello World!"/>
    </vars>
    <tasks>
        <messagebox message="${var1}"/>
    </tasks>
</project>

Let's go on to the description of XML files structure with scripts.
7.1 XML document structure with script
The name of the main node is “project”. List of the attributes for “project” node are as follows:

Attribute name
Description 
Required
version
Default value: “1.0”
No

List of subsequent nodes for “project”:

Node name
Description 
Quantity of nodes
vars
This site defined environment variables.
This site does not contain attributes.
A node can contain any quantity of sub-nodes with “var” name. “var” contains information about the variable.
Any
loadproperties
This node defines  “properties” files. 
This node does not contain attributes.
A node may comprise any number of sub-nodes with the name “file”. “file“ node contains information about the files that should be loaded.
Any
includes
This node defines other “Job Files” which must be processed before processing of other content.
This node does not contain attributes. 
The node may contain any quantity of sub-nodes with the name “include”. “include” defines the files that should be loaded.
The file processing algorithm is the same as processing of the current file.
Any
macrodefs
This node defines the templates for repetitive tasks. Templates, described in this section, can be used in the “tasks” section.
This node does not contain attributes. 
Node can contain any quantity of sub-nodes with the name “macrodef”. “macrodef” contains description of the template.
Not more than one
tasks
This node contains tasks to be performed.
This node does not contain attributes.  
Node can contain any quantity of sub-nodes, described in the List of common tasks for creating scripts. Node can contain any quantity of tasks, described in “macrodef”.
Not more than one

All nodes thing the “project” are processed in the order they appear.


“Scan” node is always processed first. Scanning in the directories, specified in “scan” can be performed only after processing of the current file.
The overall structure of the document is as follows:

“vars” section defines additional variables
“loadproperties“ section loads additional variables from “Properties” files
“includes“ section loads the content from other scripts
“macrodef” section defines templates and macros for repetitive tasks
“Tasks” section defines the list of tasks

To allow structure programming and to get rid of duplicates when writing repetitive sequences of tasks macros can be used. After definition the macros can be used in “tasks” section as a task.
Macros are specified in the “macrodefs” section in “macrodef” node.
Each “macrodef” must have a unique name, specifies in the “name” attribute. Macros name is the ID, later used as the name of the XML-node to find the macros.
“Macrodef” node contains the following sub-nodes:
attribute - for the definition of macros arguments (call parameters). Each such a node must have a “name” attribute. The quantity of “attribute” is not limited.
“Attribute” nodes should be viewed as a list of parameters to determine “macrodef”. The value of “name” attribute (“attribute” node) acts then as a list of parameters to identify “macrodef”. Value of the additional attributes can be used inside the “macrodef” in the notation of  @{parameter_name}.
“Tasks“ -section (which can be only one) contains calls for “tasks” and predefined macros.
7.1.1. “Var” nodes
“Var” node contains the following attributes:

Attribute name
Description 
Required
name
Name of variable
Yes
value
Value of the variable.
Is an empty line by default
No
replace
If set as “true”, “ok”, “yes” or “1”, the value of the variable will be replaced by the value, set in “value” attribute.
If set as “false”, “no” or “0” and there is already a variable with such a value, then the variable will not change.
If set as “false”, “no” or “0” and there is no variable with such name, a new variable will be added.
Is “False” by default.
No

This section allows to add (change) global variables.
You can use the values of already defined variables as values of attributes (“name”, “value” and “replace”). For more information, see Usage of Variables.
7.1.2 “File” nodes
“File” node contains the following attributes:

Attribute name
Description 
Required
file
Path to Properties file, from which global variables will be loaded.
Yes
replace
If set as “true”, “ok”, “yes” or “1”, the variables of the file will be replaces by the already existing ones. 
Is “false” by default.
No
varprefix
Prefix, which will be added to the beginning of each variable's name, loaded from the file.
Is an empty line by default.
No

This section allows to add (change) global variables from Properties file.
You can use the values of already defined variables as values of attributes. For more information, see Usage of Variables.
7.1.3 “Macrodef” nodes
To allow structured programming and to get rid of repetitions in repetitive sequences of tasks different macros are supported (macrodef, macroprototypes, templates). After the macros has been defined, it can be used in “task” section as a “task”. Macros can also be used in “tasks” section in other files with scripts.
Macros are defined in “macrodefs” section using the node “macrodef”.
Each “macrodef” must have a unique name, mentioned in the “name” attribute. Macros name is his ID, which will be used as the name of XML-node for search and launch.
“Macrodef” node contains the following sub-nodes:
Node name
Description 
Quantity of nodes
attribute
“Attribute“ nodes are used to define arguments (call parameters) of the macros. Each such a node must have a “name” attribute”. The quantity of “attribute” nodes is not limited. 
“Attribute” nodes should be seen as a list of parameters for “macrodef”. The value of the “name” attribute (“attribute” node) can be later used as a name for an additional attribute for a new task. Values of these additional attributes can be used inside “macrodef” in the notation of  @{parameter_name}.
Any
tasks
This section (which can be only one) contains calls for “tasks” and predefined macros. Templates can use other previously determined templates (also from other files). Nesting level is not limited.
This node doesn't contain attributes.
Not more than one

This is a simple example of a macros creation and using:

<project version="1.0">
    <macrodefs>
        <macrodef name="myinfo">
            <attribute name="mymessage"/>
            <tasks>
                <messagebox
                    condition="!isEmpty(@{mymessage})"
                    message="@{mymessage}"
                    icon="info"/>
            </tasks>
        </macrodef>
    </macrodefs>
    <tasks>
        <myinfo mymessage="Hello World!"/>
    </tasks>
</project>

This example uses a template “myinfo”. The template “myinfo” has only one defined attribute “mymessage”. The value of the “mymessage” attribute (which can be taken from @{mymessage}) is used as a display message text. “Task” message calls to a previously defined template “myinfo”.
7.1.4 “Include” node
“Include” node contain the following attributes:

Attribute name
Description 
Required
file
Path to another file to the script
Yes
replace
If given as “true”, “ok”, “yes” or “1”, the variables from the file will replace the already existing variables.
Is “false” by default
No
varprefix
Prefix which will be added in the beginning of each variable's name, loaded from the file.
An empty line by default
No
objprefix
Prefix which can be added to the beginning of each macros name, loaded from the file.
An empty line by default
No

A possibility to include other files with scenarios lets to create a correct structure of files. For example, general-purpose macros can be taken in a separate file, which will connect to other scripts.
7.1.5 List of tasks to be performed
Inside the “tasks” sections (“project” node)  there are nodes with tasks to perform. Among these nodes there can be “standard” tasks, which are described in the “List of standard tasks to create scripts” and macros. 
Every atomic task is represented as a properly arranged XML section. It is recommended to use variables in the task attributes.
It is remarkable, that each task has an “condition” attribute, in which one can white statements of any complexity and use functions, comparison operators, variables and logic operators. “Condition” attribute defines if the task will be carried out.
7.2 Global variables which affect the execution of scripts
Below is the list of variables which affect the execution of tasks and project .

Variable name
Description 
interrupt.tasks.on.error
If set as “false”, “no” or “0” - Build Machine will continue to work even in case error will occur during the performing of task.
Is “True” by default
interrupt.tasks
If set as “true”, “yes”, “ok” or “1” Build Machine will stop to perform tasks. Build Machine checks the value of this variable before performing of each task.
Is “false” by default
<project name>.prefer.out.subdir
Here the “<project name> “is the name of the project
Is used to edit the directory, where the project build files will be copied (see Build task and Libs task)
For example:


FreeTorrentDownload.prefer.out.subdir=torrent


<project name>.prefer.out.subdir.recursively
“True“ or “false“.
Is used to edit the directory, where the project build files will be copied (see Build task and Libs task)
For example:
<project name>.<fileset id>.fileset.prefer.out.subdir
Is used to edit the directory, where the project build files will be copied (see Build task and Libs task)
For example:
The value of the <project name>.<fileset ID>.fileset.prefer.out.subdir variable can be a list, elements of which are separated by commas. Each element of the list is the subdirectory, which will be created inside the “output” directory. During processing of the project all resulting files defined within the “fileset” will be copied into these subdirectories.
For example: 

FreeTorrentDownload.abc.fileset.prefer.out.subdir=common
curl.windows.abc.fileset.prefer.out.subdir=common,.,torrent



To learn more about how to use variables read the corresponding chapter Usage of Variables.

7.3 Examples

<project version="1.0">
    <vars>
        <var name="testfile1" value="${repository.dir}test1.txt" type="const"/>
        <var name="testfile2" value="${repository.dir}test2.txt" type="const"/>
        <var name="macrodef.result"/>
    </vars>

    <loadproperties comment="" condition="">
        <file file="test.properties" replace="false" varprefix="TEST"/>
    </loadproperties>

    <includes>
        <include file="job_inc.xml" objprefix="ABC" replace="false" varprefix="ABC"/>
    </includes>

    <macrodefs>
        <macrodef name="MyScenario">
            <attribute name="project"/>
            <attribute name="result"/>
            <tasks>
                <echo message="It is 'MyScenario' macrodef!"/>
                <build
                    name="@{project}"
                    configuration="Release"
                    platform="win32"
                    target="rebuild"
                    output="d:\temp8"
                    toolsversion=""
                    comment="Task to build the project @{project}"
                    condition="existsFile(${testfile1}) and existsFile(${testfile2})"/>
                <exec exe="notepad.exe" waittime="5000" show="ok" resultvar="@{result}" stdout="${exeout}">
                    <arg value="/c"/>
                    <arg value="test.bat"/>
                    <arg value="-p"/>
                </exec>
            </tasks>
        </macrodef>
        <macrodef name="MyScenario2">
            <attribute name="project2"/>
            <attribute name="result2"/>
            <tasks>
                <echo comment="" condition="" message="It is 'MyScenario2' macrodef!"/>
                <setvar name="${macrodef.result}" value="OK!"/>
                <MyScenario project="ABC" result="QWERTY"/>
            </tasks>
        </macrodef>
    </macrodefs>

    <tasks>
        <echo message="Try to build Torrent Download"/>
        <MyScenario2 project="FreeTorrentDownload"/>
    </tasks>
</project>

8. Usage of variables
Usage of variables allows to control the actions of Build Machine in the most convenient way as well as to eliminate double work when composing same texts. 

“Build Machine“ keeps a big range of variables in its memory, each variable contains a name and value. The name and the value of the variable is a text. Name of the variable must be unique. “Build Machine” compares names of variables case-insensitively.  
All variables are divided into two categories:
“Standard variables” - their value can be changed at any time.
“constants“ - their value cannot be changed. Value of this variable can be set only when you create it. If attempting to change the value, an error will occur and Build Machine will stop further execution.

Variables can be created/changed in the following ways:
1. In Build Machine itself. For example, in the beginning of its work Build Machine creates a standard list of variables (see List of global variables). 
2. As a result of “properties” files processing.
3. As a result of “Build Project File” processing. In these XML files it is possible to use “var” sections, in which value of the variables will be set/changed.
4. As a result of “Job File” processing. In these XML files it is possible to use “var” and “setvar” sections, in which value of the variables will be set/changed.

When trying to access a non-existing variable there will be no error, just another empty string. Global variables are available for the entire period of Build Machine.

As usual the work of Build Machine consists of 3 stages:
1. Initialization
2. Scanning of directories with source code to build the list of projects and its dependencies.
3. Tasks performing (which are defined in the XML files as “Job file”).
During initialization Build Machine created standard (global) variables (see List of global variables) and a part of these variables are constants. On this stage Build Machine can load variables from already known “properties” file.
When scanning the source directory Build Machine creates a global array of variables, adding variables created on the stage of initialization. Build Machine also adds variables from processed Build Project Files into the same array. All Build Project Files can work only with variables from this global array of variables.
When performing tasks Build Machine creates a second array of variables and also adds variables created on the stage of initialization. Build Machine also adds variables loaded from all processed Job Files. All Job Files can work only with variables from this second array of variables.

Variables have been separated into two arrays, because Build Project Files and all its variables have a “fundamental” nature – they describe product's projects and can be changed only if the project is changed. Job Files are file, created for different, sometimes temporary tasks. To avoid collisions between variables from these two groups of files, the variables have been divided. 

Let's explain some rules of the usage of variables:
Working with text, Build Machine interprets the text between the symbols “${“ and “}“ as the name of the variable. In case variable with such name already exists, Build Machine replaces the variable name with “${“ and “}“  with its value. If there is no value, the text will remain unchanged. 

For example, if a system already has a variable with “var1” and “value1” names, the when processing them such text as “Variable value: ${var1}” will be changed for “Variable value: value1”.

You can use any number of “${variable_name}” in the text processed by Build Machine. Moreover, it is possible to use nested structures.
For example, if a system has a “var1” variable and a “value1” value, a “var2” variable and a “value2” value, a “var3” variable and a “value3” value with  “var.base.name” name and “var” value, “var.index” name and “1” value, then the following text


Variable value: '${var1}'
Value 1: '${var1}', value 2: '${var2}', value 3: '${var3}'
Value N: ${${var.base.name}${var.index}}


will look like this after Build Machine processing: 


Variable value: 'value1'
Value 1: 'value1', value 2: 'value2', value 3: 'value3'
Value N: value1


The variables can be used:
In “Properties” files
In the attributes of XML nodes of Build Machine files 
In the attributes of XML nodes of Job files 

Here is an example of how to use variables in Build Project File:

<project version="1.0">
    <loadproperties comment="" condition="">
        <file file="${repository}build/myvariables.properties" replace="false" varprefix=""/>
    </loadproperties>
    <vars>
        <var name="additional.src.dir" value="${repository}myprojects/" replace="true"/>
    </vars>
    <scan>
        <targets>
            <target path="${additional.src.dir}apps/"/>
            <target path="${additional.src.dir}modules/"/>
        </targets>
    </scan>
    <solutions>
        <solution name="MyApplication1" filter="programs" type="build" solutiondir="." projectfile="MyApplication1.sln">
            <dependencies>
                <dependence name="curl.windows" />
                <dependence name="openssl.windows" />
                <dependence name="microsoft.vc100.crt" />
            </dependencies>
            <output>
                <fileset dir="${builds.bin}">
                    <include name="MyApplication1.exe"/>
                </fileset>
            </output>
        </solution>
    </solutions>
</project>

In Job File it is also possible to use Macrodef variables and “local” variables, not only global variables.

“Macrodef” variables (see Usage of “Macrodef”) can be used only in the definition of “Macrodef”-tasks. To access Macrodef variables “@{variable_name}” syntax is used. Lifetime of Macrodef variables is limited by task execution time, created by usage of Macrodef. 

Local variables (see, for example, “ForEach” task) can be used only when writing some tasks. To access local variables “#{variable_name}” syntax is used. Lifetime of local variables is limited by the time of task execution.

Build Machine allows to use various functions and operations to set values for variables. Job Files provide usage of a special task “SetVar” (see “SetVar” task),  which provides ability to calculate the variables based on expressions of any complexity.
9. Usage of expressions
When writing Build Project Files, Job Files it is possible to use expressions. Usage of expressions makes project descriptions and creation of scripts flexible.

In the expressions you can use:
Comparison operations “==“, “eq“, “!=“, “noteq“, “less“, “eless“, “great“ and “egreat“ (see List of operators)
Logical operations “and” and “or” (see List of operators)
Unary operations “!” and “not” (see List of operators)
Functions. List of functions and their descriptions, see List of functions. Note that Build Machine is case-insensitive when comparing the names of the functions.
Variables (see Usage of Variables). Build Machine is case-insensitive when comparing the names of the variables.

Usage of expressions is designed in such a way, that text is always the result of expression evaluation. Functions and operators of expressions always manipulate the text. However, some operations are trying to present the text as Boolean value. For example, function “isTrue(...)” returns the string “true”, if the argument of this function is same as one of the values (it isn't case-sensitive): 
”true”
”ok”
”yes”
”1”
Otherwise ”isTrue(...)” returns the string “false”. Another similar function “isFalse(...)” returns the string “true” if the argument of this function is same as one of the values (it isn't case-sensitive):

”false”
”no”
”0”
Otherwise “isFalse(...)“ returns the string “false”. Logical operators “and” and “or”,  unary operators “!” and “not” also operate with text as with “Boolean” value. 

You can use other expressions (and functions) as function arguments, nesting depth is not limited.

Let's describe the rules of expression building in the format “extended Backus – Naur Form” (https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_Form):


white_space            = “ ”, {“ ”}
digit                  = “0” | “1” | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9";
letter                 = “A” | “B” | "C" | "D" | "E" | "F" | "G" | "H" | "I" | "J" | "K" | "L" | "M" | "N" |
                         “O” | "P" | "Q" | "R" | "S" | "T" | "U" | "V" | "W" | "X" | "Y" | "Z" |
                         "a" | "b" | "c" | "d" | "e" | "f" | "g" | "h" | "i" | "j" | "k" | "l" | "m" | "n" |
                         "o" | "p" | "q" | "r" | "s" | "t" | "u" | "v" | "w" | "x" | "y" | "z";
variable_name          = letter, {letter | digit | "_" | “.” | “-”};
logical_operation      = “and” | “or”;
unary_operation        = “!” | “not”;
compare_operation      = “==” | “eq” | “!=” | “noteq” | “less” | “eless” | “great” | “egreat”;
expression             = expression_compare, white_space, [logical_operation, [white_space, expression]];
expression_compare     = [unary_operation], white_space, value, [compare_operation, [white_space, expression_compare]];
value                  = function | variable | expression_in_brackets;
function               = function_name, “(”, function_params, “)”;
function_params        = [expression, {“,” expression}]
variable               = string_value | global_variable | macrodef_variable | task_variable
string_value           = “'”, text, “'”
global_variable        = “${“, variable_name, “}”
macrodef_variable      = “@{“, variable_name, “}”
task_variable          = “#{“, variable_name, “}”
expression_in_brackets = “(”, expression, “)”


Here are examples of usage:


isTrue(${variable1})
existsFile(extractParentDir(${file1})mydoc.html)
contains(getAllProjects(), 'MyProject5')


10. List of standard tasks to build a script
We will give a detailed description of “tasks” which can be used in the files with scripts (Job Files). Modifying the source code of Build Machine you can extent this list with new “tasks”.
Note, that when processing some “tasks” use information about the projects, which are taken by Build Machine when scanning the source code directories.\
“Tasks” can be placed only inside XML section with “tasks”. “Tasks” section can be situated inside “project” or “macrodef”.
10.1 „MessageBox“ task
Allows to display dialogue message box. Section name in XML file: “messagebox”.
List of attributes:
Name
Description 
Required
message
Message text.
Is empty, the window will not appear.
no
icon
Type of icon in the dialogue window (“info”, “warning”, “error”, “stop”)
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be write into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples

<messagebox message="Hello" icon=”info”/>
<messagebox message="Hello" icon=”warning”/>
<messagebox message="Hello" icon=”error”/>

10.2 “Question” task
Allows to display dialog window with question. Section name in XML file: “question”.
List of attributes:
Name
Description 
Required
message
Message text
yes
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be write into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the user clicks on “Yes” in the dialogue box, this variable will contain “true”, if the user clicks “No”, the variable will contain “false”.
no

Examples

<question resultvar="question.result" message="Are you sure you want to remove this file?"/>


10.3 „Build” task
Task to build the project.
XML node name: “build”.
List of attributes for “build” node:
Name
Description 
Required
name
Project name to build.
Project name must coincide with the name of one of the projects, described in Build Project File (see Description / definition of projects for building).  
yes
configuration
Build configuration.
For example, for “Visual Studio” it is: “release”, “debug”,...
”Release” by default.
no
target
Build target.
For example, for “Visual Studio” it is: “Build”, “rebuild”, “clean”,...
By default: «build».
no
platform
Build platform.
For example, for “Visual Studio” it is: “x86“, “x64“, ....
By default: «».
no
platformtoolset
Build platform tool-set.
For example, for “Visual Studio” it is:  “v90“, “v100“, “v110“, ....
By default: «».
no
output
Output folder.
yes
extraparam
Additional command line, which will be given “as it is” to the executable.
For example, for “Visual Studio” it will be given as “MSBuild.exe”.
no
skipbuild
If the value is same as one of the following values: “true”, “ok”, “yes” or “1” the current project will not be built.
Is “false” by default.
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Information, which Build Machine gets while scanning the directories with source code are needed to perform this task.

When building a project, Build Machine uses the whole project tree and its dependencies. Build Machine builds projects sequentially, starting from those which are independent. Project, which name is given in the attribute “name” will be built as the last one. All projects, which affect this project, are built based with same parameters (“ configuration“, “target“, “platform“, “platformtoolset“, “output“, ...), specified in the attributes of the “build” note. All resulting the project building files (and resulting files which aren't needed for project build) will be copied into the same directory given by “output” attribute by default.

When building, Build Machine remembers all projects which it builds in order to exclude duplications of other projects with same parameters “configuration”, “platform”, “platformtoolset”. In addition, Build Machine remembers all directories, into which resulting files have been already copied, in order to exclude the possibility of duplications in the “output”. 

We'll recall that the list of resulting file for the project is defined in its description (Description / definition of projects for building). Several output files can be defines for one project. Moreover, resulting project files can be divided into groups (“fileset”), each of which can be marked with ID (“fileset ID”). All these files will be copied into the “output” after the project will be built.

It is often necessary, that some dependable projects could copy their resulting files into the directories, different from “output” while building the project. To put this into realization it is necessary to use the following functionality.
After building the project Build Machine tries to define the directory which will contain all resulting files. The algorithm of output directory detection is as follows:
1. Directory of resulting files is “output”.
2. “Build Machine“ checks if there is a variable with the following name: “<project name>.prefer.out.subdir“, so that the value of this variable is interpreted as subdirectory which will be created inside the “output” directory. The resulting project files will be copied exactly into this subdirectory. 
3. Groups of files, specified inside of different “fileset” for the same project can be copied into different directories. “Build Machine” verifies each “fileset's” ID. If “fileset ID' is set, Build Machine checks the existence of variable with “<project name>.<fileset ID>.fileset.prefer.out.subdir” name. In case there is “<project name>.<fileset ID>.fileset.prefer.out.subdir” variable, its value is interpreted as subdirectory(ies), created inside the output directory created before. In this case all resulting files defined inside this “fileset” will be copied exactly into this subdirectory. 
Moreover, before constructing of each project the system will check the existence of “<project name>.prefer.out.subdir.recursively“ (here and after “<project name>” - name of the project). If there is such a variable and its value is “true”, “ok”, “yes” or “1”, Build Machine defines subdirectory from variable “<project name>.prefer.out.subdir“ and then installs this subdirectory for all dependable projects. As a result, dependable projects will also copy their files into this subdirectory. This mechanism lets to avoid creation of global variables with “<project name>.prefer.out.subdir“ names for dependable projects. 

Value of “<project name>.<fileset ID>.fileset.prefer.out.subdir” variable can be a list, elements of which are divided by a comma. Each element of this list is a subdirectory, which will be created in the “output” directory. When processing the project, all resulting files defined inside this “fileset” will be copied into these directories.
Currently the  of projects has been implemented only in Windows and only by using “MSBuild.exe“. “MSBuild.exe“ is a file from Microsoft Visual Studio, which is used for creation of apps for Windows. The path to “MSBuild.exe” is defined by the “win.msbuild.path” variable. If the value of “win.msbuild.path “ variable is not specified, Build Machine will try to find “MSBuild.exe” as “C:/Windows/Microsoft.NET/Framework/v4.0.30319/MSBuild.exe“  (actually, it is needed to determine the path to the installed Build Machine using the appropriate registry branch).

Examples

<build name="Project_A" configuration="Release" target="Rebuild" output="d:\temp8" resultvar="${build.result}"/>

10.4 “Libs” task
Task for copying files of the project (which is not needed to be built) into specified directory.
Node name: «libs».
Attributes list for «libs»:
Name
Description 
Required
name
Project name.
Project name must coincide with the name of one of the projects, described with Build Project File (see Description / definition of projects for building).  . This project doesn't require building (“Files” type).
yes
configuration
Build configuration.
For example, for “Visual Studio” it is: “release”, “debug”,...
”Release” by default.
no
platform
Build platform.
For example, for “Visual Studio” it is: “x86“, “x64“, ....
By default: «».
no
output
Output folder
yes
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

The information collected by Build Machine after scanning the source code directories is needed to perform this task.

When building a project, Build Machine uses the whole project tree and its dependencies. Build Machine builds projects sequentially, starting from those which are independent. Project, which name is given in the attribute “name” will be built as the last one. All projects, which affect this project, are built based with same parameters (“ configuration“, “target“, “platform“, “platformtoolset“, “output“, ...), specified in the attributes of the “build” note. All resulting the project building files (and resulting files which aren't needed for project build) will be copied into the same directory given by “output” attribute by default.

Note, that Build Machine remembers all directories, into which resulting files have been already copied, in order to exclude the possibility of duplications in the “output”. 

It is often necessary, that some dependable projects could copy their resulting files into the directories, different from “output” while building the project. To put this into realization it is necessary to use the functionality, connected with “<project name>.prefer.out.subdir», «<project name>.<fileset ID>.fileset.prefer.out.subdir”  and “<project name>.prefer.out.subdir.recursively“ global variables. To learn more, see “Build” task. 

Examples:

<project version="1.0">
    <vars>
        <var name="play.sound.when.complete" value="true" replace="true"/>
        <var name="show.message.when.complete" value="true" replace="true"/>
        <var name="build.properties.file" value="..\build.properties" replace="true"/>
    </vars>
    <tasks>
        <loadproperties file="${build.properties.file}" replace="true" varprefix=""/>
        <setvar name="libs" value="getAllLibs()" asexpression="true"/>
        <dialog resultvar="dialog.result">
            <list caption="Select libraries to copy:" items="${libs}" default="" valuevar="libs.to.build" height="500" />
            <combobox caption="Build configuration:" items="Release,Debug" default="Release" valuevar="configuration"/>
            <selectdir caption="Output directory:" dir="d:\temp8" valuevar="output.dir"/>
        </dialog>
        <exit condition="isFalse(${dialog.result})"/>
        <messagebox condition="isEmpty('${libs.to.build}')" message="Libraries to copy are not defined!" icon="error"/>
        <exit condition="isEmpty('${libs.to.build}')"/>
        <foreach item="current.lib" in="${libs.to.build}">
            <libs name="#{current.lib}" configuration="${configuration}" output="${output.dir}"/>
        </foreach>
    </tasks>
</project>

10.5 ”Exec” task
Used for launching other applications. Node name: “exec”. 
List of attributes of “exec”:
Name
Description 
Required
exe
File path to execute
yes
dir
The directory in which the command should be executed. Defaults not used.
no
waittime
Time in  milliseconds to wait the process. Defaults to “1036800000” (12 days).
no
show
Boolean value (“true” or “false”). Set "false" to hide the window of new process.
Defaults to “true”.
no
catchoutput
Boolean value (“true” or “false”). Set "false" to not print the output of new process into logs.
Defaults to “true”.
no
output
Name of a file to which to write the output. Defaults not used.
no
exitcodevar
Name of a variable to which to write the exit code.
no
waitresultvar
Name of a variable to which to write the waiting result («true» or «false»).
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Command line arguments should be specified as nested <arg> elements.
Attributes list for ”arg”:
Name
Description 
Required
value
Command line parameter
no

Examples:

<exec exe="notepad.exe" condition="existsFile('d:\mydocument.txt')" failonerror="true" resultvar="${result}">
	<arg value="d:\mydocument.txt" />
</exec>

10.6 ”System” task
Execute a command. Node name: “system”. 
List of attributes of “system”:
Name
Description 
Required
cmd
Command to be executed
yes
exitcodevar
Name of a variable to which to write the exit code.
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<system cmd="D:/scripts/run.bat"/>

10.7 ”SvnUpdate” task
Is used to update the SVN repository. The name of the node is “svnupdate”.
List of attributes for “svnupdate”:
Name
Description 
Required
path
Directory path.
If not specified, the directory will be taken from ${svn.repository.dir} variable
no
revision
Number of revision, up to which it is needed to be updated.
no
tortoise
If true, this task will be processed using “TortoiseProc.exe“ (for Windows OS).
Is “false” by default.
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<svnupdate path="d:\svn"/>
<svnupdate path="d:\svn" revision="${svn.revision}"/>
<svnupdate path="d:\svn" revision="57901"/>

10.8 “SvnCommit“ task
Is used to commit all edits into SVN repository. Node's name: “svncommit”.
«Svncommit” list of attributes:
Name
Description 
Required
path
Directory path.
If not specified, the directory will be taken from ${svn.repository.dir} variable
no
message
Description for the commit
no
login
User's login for the SVN
no
password
User's password for the SVN
no
tortoise
If true, this task will be processed using “TortoiseProc.exe“ (for Windows OS).
Is “false” by default.
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<svncommit path="${projSvnPath}" login="${svn.login}" password="${svn.psw}"/>

10.9 “SvnCleanup“ task
Is used to clean up SVN repository. XML node name: “svncleanup”.
List of attributes for “svncleanup”:
Name
Description 
Required
path
Directory path.
If not specified, the directory will be taken from ${svn.repository.dir} variable
no
tortoise
If true, this task will be processed using “TortoiseProc.exe“ (for Windows OS).
Is “false” by default.
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<svncleanup path="${projSvnPath}"/>

10.10 “SvnRevisionNum“ task
Is used to get the latest SVN revision number. Node name: “svnrevisionnum”.
List of attributes for “svnrevisionnum“:
Name
Description 
Required
url
URL repository or the path to repository directory.
If not set, the sirectory will be taken from ${svn.repository.url} or ${svn.repository.dir}
no
valuevar
Name of variable, which will contain information about revision number.
no
login
User's login for the SVN
no
password
User's password for the SVN
no
tortoise
If true, this task will be processed using “TortoiseProc.exe“ (for Windows OS).
Is “false” by default.
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<svnrevisionnum url="${projSvnPath}" login="${svn.login}" password="${svn.psw}" valuevar="svn.revision" />

10.11 “SvnRevert” task
Is used to revert changes in SVN repository. XML node name: “svnrevert”.
List of attributes for “svnrevert”:
Name
Description 
Required
path
Directory path.
If not specified, the directory will be taken from ${svn.repository.dir} variable
no
recursively
If “true”, the changes in the folder will be marked recursively (fully).
Is “false” by default.
no
tortoise
If true, this task will be processed using “TortoiseProc.exe“ (for Windows OS).
Is “false” by default.
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<svnrevert path="D:\svn\3rdparty" recursively="true"/>

10.12 “VersionFileRead” task
Is used to get version number from the file.
This task works only with two specific file formats (see “SetProjectVersion” task).
XML node name: “versionfileread”.
List of attributes for “versiofileread”:
Name
Description 
Required
file
Path to file with version
yes
valuevar
Name of variable, into which the version number will be placed from the file.
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Example:

<versionfileread file="D:/svn/project1/version.h" valuevar="project.version"/>

10.13 “VersionFileWrite” task 
Is used to update version number in the file.
This task works only with two specific file formats (see “SetProjectVersion” task). XML node name: “versionfilewrite”.
List of attributes for “versiofilewrite”:

Name
Description 
Required
file
Path to file with version
yes
value
Version number
yes
autostamp
It is assumed, that version number has the following format: “<major>.<minor>.<release>.<timestamp>”, where “<timestamp>” has the following view: “<month><day>”. Commas can be used as separators.

If “true”, “timestamp” will be replaced in the version number. “Timestamp” is created according to the current date.
Is “false” by default.
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<VersionFileWrite file="d:/projects/project1/version.h" value="1.2.3.4"/>

10.14 “SetProjectVersion” task
Is used to update the version number in the project file.
This task works only with two specific file formats. Files, the contents of which is similar to the following:

#pragma once
#define INTVER 1,0,4,430
#define STRVER "1,0,4,430\0"

and files “AssemblyInfo.cs“ which are parts of Visual Studio C# projects and which contents is similar to the following: 

using System.Reflection;
using System.Runtime.CompilerServices;
using System.Runtime.InteropServices;

[assembly: AssemblyTitle("DVDVideoSoft.PresetEditor")]
[assembly: AssemblyDescription("PresetEditor")]
[assembly: AssemblyConfiguration("")]
[assembly: AssemblyCompany("DVDVideoSoft Ltd.")]
[assembly: AssemblyProduct("Free Studio")]
[assembly: AssemblyCopyright("Copyright © DVDVideoSoft Ltd. 2010")]
[assembly: AssemblyTrademark("")]
[assembly: AssemblyCulture("")]
[assembly: ComVisible(false)]
[assembly: Guid("1b5219c1-20e8-4fc5-a14e-bb97c6147b24")]
[assembly: AssemblyVersion("1.0.6.0827")]
[assembly: AssemblyFileVersion("1.0.6.0827")]

XML node name: “setprojectversion“.
List of attributes for “setprojectversion“:
Name
Description 
Required
projectname
Project name.
Project name must coincide with the name of one of the projects, described with Build Project File. (see Description / definition of projects for building).
yes
versionid
 “Version ID“  for the project.
no
version
Version number.
See comments below.
no
autostamp
(Not implemented)
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no
The information collected by Build Machine after scanning the source code directories is needed to perform this task.

Several files with versions can be specified in the project, and each of them can be marked with its “Version ID” (see Description / definition of projects for building). If “versionid” isn't empty, Build Machine will try to find file with version (full path), which is marked with the value from “versionid”. If “versionid” is empty, Build Machine gets the first file with version from the description. Build Machine will change version number exactly in this found file.
It is assumed, that version number has the following format: “<major>.<minor>.<release>.<timestamp>”, where “<timestamp>” has the following view: “<month><day>”. Commas can be used as separators. Version number, which will be written into the file, is built on the basis of “version” attribute value: “timestamp” will be edited (or added if did not exist), This variable is created according to the current date.
If “version” attribute value is empty, Build Machine will try to get the version number from “<project name>.<version ID>” variable value, where project name is taken from “projectname” attribute and <version ID> from “versionid”. 

Examples:

<setprojectversion projectname="Project1" version="2.15.3"/>

10.15 “LoadProperties“ task
Is used to load variables from Properties file. XML node name: “loadproperties”.
List of attributes for “loadproperties”:
Name
Description 
Required
file
Path to Properties file, from which all global variables will be loaded.
yes
replace
If “true”, “yes”, “ok” or “1”, variables from the file will replace the already existing ones.
Is “false” by default.
no
varprefix
Prefix, which will be added to the beginning of each variable's name, which will be loaded from the file.
Is an empty string by default.
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Example:

<loadproperties file="${build.properties.file}" replace="true" varprefix=""/>

10.16 “SetVar“ task
Is used to change the value of variable. XML node name: “setvar”.
List of attributes for “setvar”:
Name
Description 
Required
name
Name of variable 
yes
value
Value of variable or expression for calculating the value of variable
o
asexpression
If “true”, then text specified in “value” will be processed by interpretator as an expression. The result of calculation will be placed into variable value under the name “name”.
Is “false” by default. 
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<setvar name="compile.studio" value="false" asexpression="false"/>
<setvar name="commonPath" value="${svn.repository.dir}common" asexpression="false"/>
<setvar name="components.dn" value="getAllProjectsByFilter('components.dn')" asexpression="true"/>
<setvar name="components.c" value="getAllProjectsByFilter('components.c')" asexpression="true"/>
<setvar name="components.all" value="" asexpression="false"/>
<setvar name="components.all" value="addToListUnique('${components.all}', '${components.dn}')" asexpression="true"/>
<setvar name="components.all" value="addToListUnique('${components.all}', '${components.c}')" asexpression="true"/>

10.17 “Echo“ task
Is used to record a message into log file (and, additionally, into file). XML node name: “echo”.
List of attributes for “echo”:
Name
Description 
Required
message
Message text
no
level
Logging level (not put into realization yet)
no
file
Name of file, where the message will be recorded (not put into realization yet).
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<echo message=”Hello”/>

10.18 “MkDir“ task
Used for creation of directories. XML node name: “mkdir”.
List of attributes for “mkdir”:
Name
Description 
Required
dir
Name of directory, which needs to be created.
If the directory already exists or directory isn't specified, nothing happens.
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<mkdir dir="d:\temp10"/>

10.19 “SetCurrDir“ task
Is used to change the current directory. XML node name: “setcurrdir”.
List of attributes for “setcurrdir”:
Name
Description 
Required
dir
Name of current directory
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<setcurrdir dir="d:\temp10"/>

10.20 ”Copy”  task
Is used to copy files and directories. XML node name: “copy”.
List of attributes for “copy”:
Name
Description 
Required
file
Name of while which needs to copied.
no
tofile
Name of a new (copied) file.
no
todir
Directory, where the file(s) are copied.
no
overwrite
If existing files need to be overwritten.
Is “true” by default.
no
force
If existing Read-Only files need to be overwritten.
Is “true” by default.
no
verbose
If the names of the copied files should be written into log.
Is “true” by default.
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

«Copy” node can contain any quantity of “fileset” nodes.  In this node one can specify additional groups of files, which should be copied into the directory, specified by “todir” attribute. “Fileset” node must have “dir” attribute, this attribute defines the name of the catalogue, from which the files will be copied.
In turn, “fileset” node can contain any number of “include” and “exclude” nodes. “Include” and “exclude” nodes must have an attribute “name”. “Name” attribute defines files, which will be copied or, alternatively, excluded from copying. “Name” attribute can contain regular expressions.

Examples:

<copy comment="comment" condition="ok" file="d:\temp7\abc.txt" todir="d:\temp8">
    <fileset dir="dir1"/>
    <fileset dir="dir2">
        <include name="*.dll"/>
        <exclude name="*w.dll"/>
    </fileset>
    <fileset dir="dir3">
        <exclude name="*.exe"/>
    </fileset>
</copy>

10.21 “Delete“ task
Is used to delete files and directories. XML node name: “delete”.
List of attributes for “delete”:
Name
Description 
Required
file
Name of file that needs to be deleted.
no
dir
Name of directory that needs to be deleted.
no
verbose
If names of copied files need to be copied into log file.
Is “true” by default.
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

«Delete” node can contain any quantity of “fileset” nodes. “Fileset” contains information about additional groups of files, which need to be deleted. “Fileset” node must contain “dir” attribute”, this attribute defines the name of directory, from which one needs to delete files.
In turn, “fileset” node can contain any number of “include” and “exclude” nodes. “Include” and “exclude” nodes must have an attribute “name”. “Name” attribute defines files, which will be copied or, alternatively, excluded from copying. “Name” attribute can contain regular expressions.

Examples:

<delete file="d:\temp7\mydoc.txt"/>
<delete dir="d:\mydocs\"/>
<delete>
    <fileset dir="D:\temp8\">
        <include name="*.*"/>
        <exclude name="*.iss"/>
    </fileset>
    <fileset dir="D:\temp9">
        <exclude name="*.chm"/>
    </fileset>
</delete>

10.22 “Rename“ task
Used ti rename files and directories. XML node name: “rename”.
List of attributes for “rename”:
Name
Description 
Required
src
Full name of file or directory
no
newname
New name of file or directory.
It is possible to specify the name only (not the whole path).
If file or directory with a new name already exists – it will be deleted before renaming.
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<rename src="d:\temp7\mydoc.txt" newname="mydoc_v2.txt"/>

10.23 “ForEach“ task
Is used for cyclical repetition of task list. XML node name: “foreach”.
List of attributes for “foreach”: 
Name
Description 
Required
item
Name of current element.
Inside “foreach” one can get the contents of the current element by this name. 
The following syntax is used for it: #{item_name}.
no
in
List of elements, divided by comma.
Before executing “foreach” will count the number of “in” elements.
After that “foreach” will launch execution of the list of tasks, specified inside “foreach” the same amount of times as there are elements of “in”.
At each iteration current item value will be placed into the variable with the name, defined in “item”. 
The following syntax is used to approach the variable: #{item_name}.
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

“Foreach” node can contain any quantity of other tasks (including other “foreach”) ,nesting depth is not limited.

Examples:

<foreach item="item1" in="abc,cde">
    <foreach item="item2" in="1,3,5,7">
        <messagebox message="Current item: #{item1}/#{item2}"/>
    </foreach>
</foreach>

10.24 “Break“ task
Is used to exit from “foreach” loop. XML node name: “break”.
List of attributes for “break”:

Name
Description 
Required
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<foreach item="item1" in="abc,cde">
    <foreach item="item2" in="1,3,5,7">
        <messagebox message="Current item: #{item1}/#{item2}"/>
        <break condition="#{item1} == cde"/>
    </foreach>
</foreach>

10.25 “Сontinue“ task
Is used to access the next iteration in “foreach” loop. XML node name: “continue”.
List of attributes for “continue”:
Name
Description 
Required
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<foreach item="item1" in="abc,cde">
    <foreach item="item2" in="1,3,5,7">
        <messagebox message="Current item: #{item1}/#{item2}"/>
        <continue condition="#{item1} == abc"/>
    </foreach>
</foreach>

10.26 “WriteFile“ task
Used to write a text file. XML node name: “writefile”.
List of attributes for “writefile”:
Name
Description 
Required
file
File name
yes
openexisting
If “true” - file must exist.
Is “false” by default.
no
truncate
If “true”, the contents of the file will be deleted before recording.
Is “false” by default (recorded information is placed in the end of file).
no
createnew
If “true”, file must not exist.
Is “false” by default.

no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

”Writefile” node can contain nested nodes “text” (in any quantity). The presence of any attributes in “text” is not provided. “Text” nodes contain text, which must be written into file. Text can be placed inside “CDATA” in “text” nodes.
Examples:

<writefile file="D:\temp\myfile.txt" truncate="1">
    <text>qwerty</text>
    <text><![CDATA[<<*>1234567890<*>>]]></text>
    <text>qwerty-${var2}</text>
</writefile>

10.27 ”WinRegRead” task
Used to read values from the registry Windows. XML node name: “winregread”.
List of attributes for “winregread”:
Name
Description 
Required
key
Node full name in registry.
For example:
HKEY_CLASSES_ROOT\LDAP
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft
HKEY_USERS\.DEFAULT\Console
HKEY_CURRENT_CONFIG\Software\Fonts
Or a shortened version:
HKCR\LDAP
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\SOFTWARE\Microsoft
HKU\.DEFAULT\Console
HKCC\Software\Fonts
yes
valuename
Name of value
no
valuevar
Name of variable. If the name of variable is specified, then the read value from registry will be recorded into the variable with this name.
no
wow
There are following options:
wow32
wow64
“wow32“ - will be read from the 32-bit registry hive (not depending on the application bit)
“wow64“ - will be read from the 64-bit registry hive (not depending on the application bit)
If the attribute is not specified, then the rules of rule of reading from the registry will be determined by Windows applications depending on the application bit.
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<winregread key="HKCU\Software\DVDVideoSoft\BuidSystem\Test" valuevar="reg.val.0"/>
<winregread key="HKCU\Software\DVDVideoSoft\BuidSystem\Test" valuename="StringValueName" valuevar="reg.val.1"/>
<winregread key="HKCU\Software\DVDVideoSoft\BuidSystem\Test" valuename="DwordValueName" valuevar="reg.val.2"/>
<winregread key="HKCU\Software\DVDVideoSoft\BuidSystem\Test" valuename="QwordValueName" valuevar="reg.val.3"/>


10.28 “WinRegWrite“ task
Is used to write names to the Windows registry. XML node name: “winregwrite”.
List of attributes for “winregwrite”:
Name
Description 
Required
key
Node full name in registry.
For example:
HKEY_CLASSES_ROOT\LDAP
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft
HKEY_USERS\.DEFAULT\Console
HKEY_CURRENT_CONFIG\Software\Fonts
Or a shortened version:
HKCR\LDAP
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\SOFTWARE\Microsoft
HKU\.DEFAULT\Console
HKCC\Software\Fonts
yes
valuename
Name of value. If not specified, the value will be updated without any name.
no
valuetype
Value type. There are following valid options: 
string
dword
qword
If the value type is not specified, it will be assumed that this is a string value. 
no
value
What we want to write to Registry.
no
wow
There are following options:
wow32
wow64
“wow32“ - will be read from the 32-bit registry hive (not depending on the application bit)
“wow64“ - will be read from the 64-bit registry hive (not depending on the application bit)
If the attribute is not specified, then the rules of rule of reading from the registry will be determined by Windows applications depending on the application bit.
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<winregwrite key="HKCU\Software\DVDVideoSoft\BuidSystem\Test" valuename="StringValueName" valuetype="string" value="qwerty"/>
<winregwrite key="HKCU\Software\DVDVideoSoft\BuidSystem\Test" valuename="DwordValueName" valuetype="dword" value="12345"/>
<winregwrite key="HKCU\Software\DVDVideoSoft\BuidSystem\Test" valuename="QwordValueName" valuetype="qword" value="123456789"/>

10.29 “WinRegDelete“ task
Used to remove nodes and values from the registry Windows. Name of the XML node: «winregdelete». 
List of attributes for «winregdelete»:
Name
Description 
Required
key
Node full name in registry.
For example:
HKEY_CLASSES_ROOT\LDAP
HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft
HKEY_USERS\.DEFAULT\Console
HKEY_CURRENT_CONFIG\Software\Fonts
Or a shortened version:
HKCR\LDAP
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\SOFTWARE\Microsoft
HKU\.DEFAULT\Console
HKCC\Software\Fonts
yes
valuename
Name of value for deletion. 
If not specified — the entire node will be removed.
no
wow
There are following options:
wow32
wow64
“wow32“ - will be read from the 32-bit registry hive (not depending on the application bit)
“wow64“ - will be read from the 64-bit registry hive (not depending on the application bit)
If the attribute is not specified, then the rules of rule of reading from the registry will be determined by Windows applications depending on the application bit.
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<winregdelete key="HKCU\Software\DVDVideoSoft\BuidSystem\Test" valuename="StringValueName"/>
<winregdelete key="HKCU\Software\DVDVideoSoft\BuidSystem\Test" valuename="DwordValueName"/>
<winregdelete key="HKCU\Software\DVDVideoSoft\BuidSystem\Test"/>


10.30 “Exit“ task
Used to interrupt tasks. Name of the XML node: «exit». 
List of attributes for «exit»:
Name
Description 
Required
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<exit condition="isTrue(${exit})" />

10.31 “ExpandSysVar“ task
Used to obtain the values of system variables. Name of the XML node: «expandsysvar». 
List of attributes for «expandsysvar»:
Name
Description 
Required
text
Text, which must be expanded using system variables.
no
valuevar
Name of variable, into which the expanded value will be saved.
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<expandsysvar text="%ProgramFiles%" valuevar="q123"/>

10.32 “Dialog“ task
Is used to create and display a dialog box. Set of window elements and the displayed data is determined by the contents of XML node for this task.
XML node name: “dialog”.
Put into realization only for Windows. Near Build Machine's EXE file there must be “nbuilder_dlg.dll“.
List of attributes for “dialog”:
Name
Description 
Required
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

“Dialog” node can contain any subsidiary nodes ”combobox”, ”checkbox”, ”editbox”, ”selectdir”, ”selectfile” and ”list”.
”Combobox” node is used to display single-line wordlist. List of attributes for “combobox”:
Name
Description 
Required
valuevar
Name of variable, into which the chosen by user value will be saved.
no
caption
Name of element
no
default
Value of the selected element.
no
items
List of displayed rows in “combobox” element. Rows are divided by somma in this list.
no

List of rows in “combobox” can be set in “item” subnodes as well. “Item” nodes have only one attribute: “text”, which defines the row for the list.
List of attributes for “checkbox”:
Name
Description 
Required
valuevar
Name of variable, into which the chosen by user value will be saved.
no
caption
Name of element
no
value
If equals to “true” or “ok” or “yes” or “1” the element state is checked.
By default: unchecked.
no

“Editbox» node is used to edit the text. List of attributes for «editbox»:
Name
Description 
Required
valuevar
Name of variable, into which the entered by user text will be saved.
no
caption
Name of the element.
no
text
Text displayed at start.
no

“Selectdir” node is used to choose the directory. List of attributes for “selectdir”:
Name
Description 
Required
valuevar
Name of the variable, into which the chosen by user path to directory will be saved.
no
caption
Name of the element.
no
dir
Directory shown at start.
no

“Selectfile“ node is used to choose the file. List of attributes for “selectfile”:
Name
Description 
Required
valuevar
Name of variable, into which the chosen by user path to file will be saved.
no
caption
Name of the element.
no
file
File shown at start.
no

“List” node is used to display multi-line wordlist. List of attributes for “list”:
Name
Description 
Required
valuevar
Name as variable, into which the chosen by user values will be saved. Chosen values are shown as a list of rows divided by comma.
no
caption
Name of the element.
no
default
List of selected rows at start.
no
items
List of displayed rows in “list” element. Rows are divided by comma in this list.
no
height
Height of visual element (in pixel).

List of rows in “list” can also be defined in “item”. “Item” nodes have only one attribute: “text”, which defines the row for the list.

Examples:

<project version="1.0">
    <vars>
        <var name="play.sound.when.complete" value="true" replace="true"/>
        <var name="show.message.when.complete" value="true" replace="true"/>
        <var name="build.properties.file" value="..\build.properties" replace="true"/>
    </vars>
    <tasks>
        <loadproperties file="${build.properties.file}" replace="true" varprefix=""/>

        <printvars/>

        <setvar name="components.dn" value="getAllProjectsByFilter('components.dn')" asexpression="true"/>
        <setvar name="components.c" value="getAllProjectsByFilter('components.c')" asexpression="true"/>

        <setvar name="components.all" value="" asexpression="false"/>
        <setvar name="components.all" value="addToListUnique('${components.all}', '${components.dn}')" asexpression="true"/>
        <setvar name="components.all" value="addToListUnique('${components.all}', '${components.c}')" asexpression="true"/>

        <dialog resultvar="dialog.result">
            <list caption="Select the project:" items="${components.all}" default="" valuevar="projects.to.build" height="500" />
            <combobox caption="Build configuration:" items="Release,Debug" default="Release" valuevar="configuration"/>
            <combobox caption="Build target:" items="Build,Rebuild" default="Rebuild" valuevar="target"/>
            <selectdir caption="Output directory:" dir="d:\temp\Projects\Bin\" valuevar="output.dir"/>
        </dialog>
        <exit condition="isFalse(${dialog.result})"/>

        <foreach item="current.project" in="${projects.to.build}">
            <build
                name="#{current.project}"
                configuration="${configuration}"
                target="${target}"
                output="${output.dir}"/>
        </foreach>
    </tasks>
</project>

10.33 “Sound“ task 
Is used to playback the melody (only .wav format at the moment). XML node name: “sound”.
List of attributes for “sound”:
Name
Description 
Required
file
Name of .wav file
If file is defined and exists, it will be play-backed.
no
type
Defines the type of a “standard” melody for playback.
There are only two types of a standard melody: “error“ and “success“.
“type“ can have the following values: 
“success“, “s“, “ok“
“error“, “err“, “e“, “fail“
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<sound type="ok"/>
<sound type="error"/>
<sound file="d:\error2.wav"/>

10.34 “PrintVars“ task
Is used to display the current list of variables. XML node name: “printvars”.
List of attributes for “printvars”:
Name
Description 
Required
file
Name of file.
If the file is not set, the list of variables will be transferred to this file (except log files and the previous contents will be deleted).
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no

Examples:

<printvars file="d:\temp8\variables.txt"/>

10.35 “PrintProjects“ task 
Is used to display list of the projects (saved in log). XML node name: “printprojects”.
List of attributes for “printprojects”:
Name
Description 
Required
showdependence
If “true”, then a list of dependable projects will be printed for each project.
Is empty (“false”) by default.
no
details
If “true”. Such information will be printed for each project.
Is empty (“false”) by default.
no
file
File name
If file is defined, the list of project will be transferred to this file (except log files and the previous contents will be deleted).
no
comment
Comment
no
condition
This is a condition for execution. Expressions (functions, variables, logic operators) can be used as condition text.
no
failonerror
If “true” - when error appears during task performance other task execution will be interrupted.
If “false” - the actual code of the error will be written into “resultvar” and the execution of other tasks will be continued even if this task will end up with a failure.
Is “true” by default.
no
resultvar
Name of variable. If the name of the variable is indicates, the variable with such name will contain error code ('0' if the task will be performed without mistakes).
no
To perform this task it is required to have information that «Build Machine» receives when scanning the source directory.

Examples:

<printprojects file="d:\temp8\projects1.txt"/>
<printprojects showdependence="true" file="d:\temp8\projects2.txt"/>
<printprojects details="true" file="d:\temp8\projects3.txt"/>

11. Short list of tasks
Below is the shortened list of “tasks”, which can be used in “Job File”.

Task name
Description
MessageBox
Display message box
Question
Display question box
Build
Project build according to the source code (project is defined in one of the “Build Project Files”)
Libs
Copy libraries into a specific directory (library is defined in one of the “Build Project Files”)
SetProjectVersion
Change project version number (defined in one of the “Build Project Files”)
Exec
Execute external application
System
Execute a command
SvnUpdate
Perform “Update” operation in SVN repository
SvnCommit
Perform “Commit” operation in SVN repository
SvnCleanup
Perform “Cleanup” operation in SVN repository
SvnRevisionNum
Obtain revision number from SVN repository
SvnRevert
Perform “Revert” operation in SVN repository
VersionFileRead
Read version number from version file
VersionFileWrite
Edit version number in version file
LoadProperties
Load variables from “properties” file
SetVar
Change/create a global variable
Echo
Forcefully record messages into log-file
MkDir
Create directories
SetCurrDir
Set current directory
Copy
Copy files and directories
Delete
Delete files and directories
WriteFile
Record data into file
ForEach
Implements the cyclic execution of a specified task list
Break
Forced exit from “ForEach” cycle. Is used only inside “ForEach” task.
Сontinue
Forced transition to the next iteration in “ForEach” cycle. Is used only inside the “ForEach” task.
WinRegRead
Read data from Windows Registry (only for Windows OS)
WinRegWrite
Record data to Windows Registry (only for Windows OS)
WinRegDelete
Delete data from Windows Registry  (only for Windows OS)
Exit
Forced termination of task execution
PrintVars
Print all variables into log-file
PrintProjects
Print all projects and libraries (which were defined from the Build Project File) into log-file
Dialog
Display dialogue with a wide range of items to enter information
ExpandSysVar
Get real values for system environment variables
Move
Move files and directories (not implemented)
Rename
Rename names and directories
Sound
Playback music from WAV file

To learn more information about tasks, see List of standard tasks to build a script.
12. List of operators
Below is the list of all operators, which can be used when writing expressions.

Operator name
Description 
!
Negation. Result will be “true” if the expression right from the operator is “true”, “ok”, “yes” or “1” (case-insensitive). Otherwise, “false”.
This statement must not necessarily be followed by a space. 
Examples:

!${variable3}
!existsDir(${app.dir})


not
Negation. Result will be “true” if the expression right from the operator is “true”, “ok”, “yes” or “1” (case-insensitive). Otherwise, “false”.
This statement must be followed by a space. 
Examples:

not ${variable3}
not existsDir(${app.dir})


==
“Equal” operator. Result will be “true” if the expressions on the left and right are equal. Otherwise, result will be “false”.
If the expression on the right and left are represented as integers - expressions will be compared as integers. If the expression on the right and left of the operator are represented as numbers with a fractional part - the expression will be compared as numbers with a fractional part. 
eq
“Equal” operator. Result will be “true” if the expressions on the left and right are equal. Otherwise, result will be “false”.
If the expression on the right and left are represented as integers - expressions will be compared as integers. If the expression on the right and left of the operator are represented as numbers with a fractional part - the expression will be compared as numbers with a fractional part. 
!=
“Not equal” operator. Result will be “true” if the expressions on the left and right are not equal. Otherwise, result will be “false”.
If the expression on the right and left are represented as integers - expressions will be compared as integers. If the expression on the right and left of the operator are represented as numbers with a fractional part - the expression will be compared as numbers with a fractional part. 
noteq
“Not equal” operator. Result will be “true” if the expressions on the left and right are not equal. Otherwise, result will be “false”.
If the expression on the right and left are represented as integers - expressions will be compared as integers. If the expression on the right and left of the operator are represented as numbers with a fractional part - the expression will be compared as numbers with a fractional part. 
less
”Less” operator. Result will be “true”, if the expression on the left is less than the expression on the right. Otherwise, it's “false”.
If the expression on the right and left are represented as integers - expressions will be compared as integers. If the expression on the right and left of the operator are represented as numbers with a fractional part - the expression will be compared as numbers with a fractional part. If the expressions on right or left are not numbers, the expressions will be compared as rows (case-insensitive).
eless
”Less or equal” operator. Result will be “true”, if the expression on the left is less than the expression on the right or equal to it. Otherwise, it's “false”.
If the expression on the right and left are represented as integers - expressions will be compared as integers. If the expression on the right and left of the operator are represented as numbers with a fractional part - the expression will be compared as numbers with a fractional part. If the expressions on right or left are not numbers, the expressions will be compared as rows (case-insensitive).
great
“More” operator. Result will be “true”, if the expression on the left is more than the expression on the right. Otherwise, it's “false”.
If the expression on the right and left are represented as integers - expressions will be compared as integers. If the expression on the right and left of the operator are represented as numbers with a fractional part - the expression will be compared as numbers with a fractional part. If the expressions on right or left are not numbers, the expressions will be compared as rows (case-insensitive).
egreat
”More or equal” operator. Result will be “true”, if the expression on the left is more than the expression on the right or equal to it. Otherwise, it's “false”.
If the expression on the right and left are represented as integers - expressions will be compared as integers. If the expression on the right and left of the operator are represented as numbers with a fractional part - the expression will be compared as numbers with a fractional part. If the expressions on right or left are not numbers, the expressions will be compared as rows (case-insensitive).
or
Logic operator “or”. Result will be “true” is the expressions on the left or right are equal to one of the following values (case-insensitive):
“true“
“ok“
“yes“
“1“
Otherwise, it's “false”.
and
Logic operator “and”. Result will be “true” is the expressions on the left and right are equal to one of the following values (case-insensitive):
“true“
“ok“
“yes“
“1“
Otherwise, it's “false”.

To learn more information about tasks, see Usage of Expressions. 
13. List of global variables
Below is the list of variables, created by Build Machine automatically at start.

Name of variable 
Description 
Constant
symbol.cr
Symbol: \r
yes
symbol.lf
Symbol: \n
yes
symbol.crlf
Symbol: \r\n
yes
symbol.quote
Symbol: “
yes
symbol.squote
Symbol: ’
yes
symbol.left.angle.bracket
Symbol: <
yes
symbol.right.angle.bracket
Symbol: >
yes
current.time
Current time in the following format: “MM/dd/yyyy hh:mm:ss“
yes
version.build.number
Row “<month><day>“, built at launching of the program. <month> - current month. <day> - current day of month
yes
this.computer.name
Computer name
yes
this.domain.name
Domain name for current user
yes
this.user.name
User name
yes
this.user.sid
SID for current user
yes
this.file
Full name of executable file (“Build Machine“)
yes
this.dir
Directory of executable file (“Build Machine“)
yes
interrupt.tasks.on.error
If set a “false”, “no” or “0” - Build Machine will continue to work even if errors occur during performing tasks.
Is “true” by default
no
interrupt.tasks
If set as “True”, “yes”, “ok” or “1” - Build Machine will stop performing tasks. Build Machine checks the value before execution of each task.
Is “false” by default
no
<project name>.prefer.out.subdir
Here “<project name>“ is the name of the project.
Is used to change the pre-defined  directory into which the resulting files of project building will be copied to another one (see «Build» task and «Libs» task)
For example:

FreeTorrentDownload.prefer.out.subdir=torrent


no
<project name>.prefer.out.subdir.recursively
“true“ or “false“.
Is used to make changes in the directory, where the project files will be copied when building a project (see «Build» task and «Libs» task)
no
<project name>.<fileset id>.fileset.prefer.out.subdir
Is used to make changes in the directory, where the project files will be copied when building a project (see «Build» task and «Libs» task)
The value of the <project name>.<fileset ID>.fileset.prefer.out.subdir variable can be a list, elements of which are separated by commas. Each element of the list is the subdirectory, which will be created inside the “output” directory. During processing of the project all resulting files defined within the “fileset” will be copied into these subdirectories.
For example: 

FreeTorrentDownload.abc.fileset.prefer.out.subdir=common
curl.windows.abc.fileset.prefer.out.subdir=common,.,torrent


no
play.sound.when.complete
Is “true” by default
no
show.message.when.complete
Is “true” by default
no
win.msbuild.path
Path to “MSBuild.exe“. “MSBuild.exe“ is the file from Microsoft Visual Studio used to create applications on Windows.
no

To learn more about usage of Variables, see Usage of Variables. 
14. List of functions
Below is the list of all functions, that can be used when writing expressions.

Function name
Description 
isEmpty(...)
Verifies that the function argument is an empty string.
Accepts one argument.
Will return “true” string if the argument is an empty string or an empty string surrounded by single or double quotes. Otherwise, it is “false”.
Example:

isEmpty(${my.variable})


isTrue(...)
Checks, if the string can be represented as “Boolean” with “true” value.
Accepts one argument.
Will return “true” string if the argument is equal to one of the following values:
“true“
“ok“
“yes“
“1“
Otherwise, it is “false”.
Example:

isTrue(${my.variable})


isFalse(...)
Checks, if the string can be represented as “Boolean” with “false” value.
Accepts one argument.
Will return “true” string if the argument is equal to one of the following values:
“false“
“no“
“0“
Otherwise, it is “false”.
Example:

isTrue(${my.variable})


existsFile(<path>)
Checks if the file exists.
Function has one argument.
Will return “true” string if the file defined by argument already exists. Otherwise, it is “false”.
existsDir(<path>)
Checks if the directory exists.
Function has one argument.
Will return “true” string if the directory defined by argument already exists. Otherwise, it is “false”.
extractParentDir(<path>)
Extracts parent directory.
Function has one argument. The argument is interpreted as the path to the file and directory.
Will return path to parent directory.
combinePath(<path1>, <path2>)
Concatenation of paths.
Can have two arguments. Both arguments are paths to files or directories.
Will return to path which was built by adding the second path to the first one.
For example:

combinePath('d:/MyProjects', 'Project1/src') // Result: “d:/MyProjects/Project1/src”


getAllProjects()
Gets the list of all projects.
Has no arguments.
Returns the list of all projects. Names of projects are divided by commas. “Build Machine” builds this list based on the data got after scanning the directories with source code.
getAllBuildProjects()
Gets the list of all projects which are needed to be built.
Has no arguments.
Returns the list of all projects. Names of projects are divided by commas. “Build Machine” builds this list based on the data got after scanning the directories with source code.
getAllLibs()
Gets the list of all projects which are not needed to be built.
Has no arguments.
Returns the list of all projects. Names of projects are divided by commas. “Build Machine” builds this list based on the data got after scanning the directories with source code.
getProjectFile(<project name>)
Get path to “project file“.
Has one argument: name of the project.
Returns the path back to “project file”. This path is defined in “projectfile” attribute when describing the project (see Description/Definition of project for build). Build Machine returns this data based on the information got after scanning the directories with source code.
getProjectDir(<project name>)
Get directory with project.
Has one argument: name of the project.
Returns the path back to “project file”. This path is defined in “solutiondir” attribute when describing the project (see Description/Definition of project for build). Build Machine returns this data based on the information got after scanning the directories with source code.
getProjectVerFileById(<project name>, <id>)
Return path to the file with project version.
Function has two arguments.
The first one is the project name. The other one is Version ID, specified in the project description.
An error will occur if one of these arguments isn't specified.
Project can contain several files with versions and each of them can have its own version ID (see Description/Definition of project for build).
Function returns path to the version file for defined project, marked by the specified ID. If the file with specified Version ID isn't found, an empty string will  be returned.
Build Machine returns this data based on the information got after scanning the directories with source code.

getProjectVerFiles(<project name>)
Return path to the list of files with project version.
Function has one argument – name of project.
An error will occur if the name of this argument isn't specified or isn't found.
Project can contain several files with versions and each of them can have its own version ID (see Description/Definition of project for build). Function returns the list with all file versions for the specified project. Elements are now divided by comma.
Build Machine returns this data based on the information got after scanning the directories with source code.
getProjectVerFileIds(<project name>)
Returns the list with all Version Ids for the project.
Function has one argument – name of project.
An error will occur if the name of this argument isn't specified or isn't found.
Project can contain several files with versions and each of them can have its own version ID (see Description/Definition of project for build).
Function returns the list with all Version IDs for the specified project. Elements are now divided by comma.
Build Machine returns this data based on the information got after scanning the directories with source code.
getProjectVerFileCount(<project name>)
Return number of files with project version.
Function has one argument – name of project.
An error will occur if the name of this argument isn't specified or isn't found.
Function returns number of version files for specified project.
Build Machine returns this data based on the information got after scanning the directories with source code.
getProjectDefFile(<project name>)
Get path to “pmanifest.xml“ file, in which the project is defines.
Has one argument – project name.
Returns the whole path to “pmanifest.xml“ file, in which the project is defined. Build Machine returns this data based on the information got after scanning the directories with source code.
getProjectOutDirs(<project name>)
Return the list with all output directories for the project.
Has one argument – project name.
An error will occur if the project name isn't specified or isn't found.
Project can contain several output directories and each of them can have its own Fileset ID (see Description/Definition of project for build).
Function returns the list with all output directories for this project. Elements of list are divided by commas.
Build Machine returns this data based on the information got after scanning the directories with source code.
getProjectOutDirIds(<project name>)
Return the list with all Fileset IDs for the project.
Has one argument – project name.
An error will occur if the project name isn't specified or isn't found.
Project can contain several output directories and each of them can have its own Fileset ID (see Description/Definition of project for build).
Function returns the list with all Fileset IDs for this project. Elements of list are divided by commas.
Build Machine returns this data based on the information got after scanning the directories with source code.
getProjectOutDirById(<project name>, <id>)
Return output directory for project.
Function has two arguments. First one is the project name, second one is the Fileset ID, specified in the project description.
Project can contain several output directories and each of them can have its own Fileset ID (see Description/Definition of project for build).
Function returns output directories for specified project, marked by specified Fileset ID. If output directory with specified Fileset ID isn't specified, an empty string will appear.
Build Machine returns this data based on the information got after scanning the directories with source code.
getProjectOutDirIdByPath(<project name>, <path>)
Return Fileset ID for the project.
Fuction has two arguments: first one is the project name, second one is the path to directory.
An error will occur if the project name isn't specified or isn't found.
Project can contain several output directories and each of them can have its own Fileset ID (see Description/Definition of project for build).
Function returns Fileset ID for the specified project, in which output directory coincides with the second argument of function. If the directory isn't found, an empty string will appear.
Build Machine returns this data based on the information got after scanning the directories with source code.
getProjectFinalOutDir(<project name>, <output dir>)
Return “specific output directory” for project.
Function has two arguments: first one is project name, second is path to a directory. An error will occur if the project name isn't specified or isn't found.
If the second argument is empty, function will return the variable value, which looks like this:


<project name>.prefer.out.subdir

If there is no variable with such name and the second argument is empty, am empty string will appear.
If there is no such variable and the second argument isn't empty, function will return the second argument, trying to present a second argument as the full path to the directory.
If there is already a variable with such name and the second argument isn't empty, the function will return the path to directory. It will be built from the second argument and from subdirectory, specified by “<project name>.prefer.out.subdir“ variable value. 
Build Machine returns this data based on the information got after scanning the directories with source code.
getProjectFinalOutDirById(<project name>, <id>, <output dir>)
Returns “specific output directory” for the project. Function has 3 agruments.
First one is the project name. Second one is the Fileset ID, specified in the project (see Description/Definition of project for build). Third one is a path to a directory.
An error will occur if the project name isn't specified or isn't found.
If the third argument is empty, function will return the value of variable, which name looks like this:

<project name>.<fileset id>.fileset.prefer.out.subdir

In case Fileset ID isn't empty. And like this in case Fileset ID argument is empty:

<project name>.fileset.prefer.out.subdir

If there is no variable with such name and the third argument is empty, am empty string will appear.
If there is no such argument and the third argument isn't empty, function will return third argument trying to deploy it as the full path to the directory.
If there is a variable with such name and the third argument isn't empty, the function will return path to the  directory and build it using third argument and subdirectory, specified by “<project name>.prefer.out.subdir“ variable value. 
Build Machine returns this data based on the information got after scanning the directories with source code.
getChildProjects(<project name>)
Get list of projects which the current project depends on.
Has one argument – project name.
Returns project dependencies list (project name specified in the argument). Project names are divided by comma.
Build Machine returns this data based on the information got after scanning the directories with source code.
getAllChildProjects(<project name>)
Get list of all projects, which the current project depends on.
Has one argument – project name.
Returns the entire project dependencies list (project name specified in the argument). List is obtained recursively as a result of scanning of the tree with all dependable projects. Project names are divided by comma.
Build Machine returns this data based on the information got after scanning the directories with source code.
isBuildProject(<project name>)
Return “true” string if the specified project is the project for building. Otherwise is “false”.
Has one argument – project name.
An error will occur if the project name isn't specified or isn't found.
Build Machine returns this data based on the information got after scanning the directories with source code.
isLib(<project name>)
Return “true” string if the specified project does not require building. Otherwise is “false”.
Has one argument – project name.
An error will occur if the project name isn't specified or isn't found.
Build Machine returns this data based on the information got after scanning the directories with source code.
getComputerName()
Current user computer name
getDomainName()
Current user domain name name
getUserName()
Current user name
getUserSid()
SID of current user
removeDuplicates(<text>)
Returns the list from which all repeated elements have been deleted. Elements are divided by comma.
addToList(<list>, <item to add>)
Add new element to the list.
Function has two arguments. First is the “list” - a set or strings, divided by comma (for example, “item1,item2,item5“). Second argument is the string that needs to be added into the list. Function returns the list string, into which the second argument is added.
Example: 

addToList('', 'item1')              // Result: “item1”
addToList('item1,item3', 'item5')   // Result: “item1,item3,item5”
addToList('item1,item3', 'item3')   // Result: “item1,item3,item3”


addToListUnique(<list>, <item to add>)
Add new unique element(s) to the list.
Function has two argument.
First is the “list” - a set or strings, divided by comma (for example, “item1,item2,item5“). Second argument is the string or a list that needs to be added into the list. Function returns the list string, into which only unique elements from second argument have been added.
Example:

addToListUnique('', 'item1')                    // Result: “item1”
addToListUnique('item1,item3', 'item5')         // Result: “item1,item3,item5”
addToListUnique('item1,item3', 'item3')         // Result: “item1,item3”
addToListUnique('item1,item3', 'item3,item9')   // Result: “item1,item3,item9”


getAllProjectsByFilter(<filter name>)
Get list of all projects, that meet filter requirements.
Has one argument: filter text.
Returns the list of all projects, that meet search string requirements, specified in function argument. Project list consists of project names divided by comma. If filter text is empty or equals “*”, function will return the list with all projects.
Any project (projects are described with “build Project File”, see Description/Definition of project for build) can have a filter text property.
Project name will be in the list if its Filter property starts with text filter and there is a “.” sign after filter text in “filter” property.
It is possible to use regular expressions for search as filter text.
Build Machine returns this data based on the information got after scanning the directories with source code.
getProjectFilterName(<project name>)
Get project filter string.
Has one argument – project name.
Returns the project filter string. This value is defined in “filter” attribute when describing projects (see Description/Definition of project for build). Build Machine returns this data based on the information got after scanning the directories with source code.
contains(<text>, <item to search>)
Check occurrence of a substring.
Function has two arguments. 
It will return “true” string is the first text argument contains the second argument as a substring (case-sensitive).
Otherwise is “false”.
containsIgnoreCase(<text>, 
<item to search>)
Function has two arguments. It will return the “true” string is the first argument contains the second argument as a subline (case-insensitive). 
Otherwise, the result is “false”.
getBuildConfiguration()
Current value of “Build Configuration“.
Function doesn't have arguments
Returns the current value of global variable “internal.current.build.configuration“.
Value of this variable will be changed automatically at the beginning of project build (which are built using “Build” and “Libs” tasks). Note, that at the moment all projects are built in the same thread.
“Configuration” property for current project will serve as a value for “internal.current.build.configuration“ (see “Build” task and “Libs” task).
 getBuildTarget()
Current value of “Build Target“. 
Function doesn't have arguments
Returns the current value of global variable “internal.current.build.target“.
Value of this variable will be changed automatically at the beginning of project build (which are built using “Build” and “Libs” tasks). Note, that at the moment all projects are built in the same thread.
“Target” property for current project will serve as a value for “internal.current.build.target“ (see “Build” task).
getBuildPlatform()
Current value of “Build Platform“. 
Function doesn't have arguments
Returns the current value of global variable “internal.current.build.platform“.
Value of this variable will be changed automatically at the beginning of project build (which are built using “Build” and “Libs” tasks). Note, that at the moment all projects are built in the same thread.
“Platform” property for current project will serve as a value for “internal.current.build.platform“ (see “Build” task and “Libs” task).

To learn more about the usage of functions, see Usage of Expressions. 
Note, that Build Machine is case-insensitive when working with function names.
15. Licence agreement
Copyright (C) 2014 DVDVideoSoft Ltd.
All rights reserved.

This software is released under the MIT License (MIT).

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
16. Source code
Source code can be downloaded at http://appmiracle.com/appm-build-automation.
Binar files can be downloaded at http://appmiracle.com/appm-build-automation/.
Project has been built using Qt5, Visual Studio 2010 has been used as development environment. Programming language: C++.
Currently the project is only Windows applicable (Windows XP/7/8).
When using dialog box with dynamic contents (see “Dialog” task) it is necessary to keep “nbuilder_dlg.dll“ module near the executable. There are no other dependencies from external libraries/projects (except Qt5 and “nbuilder_dlg.dll“).
17. Contact developers
You can send your questions, bug reports and suggestions to the following address:
pr@appmircale.com
vlsk@outlook.com (developer)
18. TO-DO
Current project is fully operational.
At the same time, the project may have flaws since some functionality hasn't been implemented yet (project has been developed by one specialist during a month).
Below is the to-do list for future improvements:

Some places of Build Machine require processing edits (including architectural edits)
A more complete integration with the compiler Microsoft Visual Studio is required.
Integration with other compilers and tools is required (“GCC“, “Qt“, “CMake“, “XCode“, “Java“, …)
A possibility to work on other OS needs to be implemented (Mac OS, Linux, …)
Integration for other version control systems is required (Mercurial, Git, ClearCase, ...)
Expand the set of tasks
Expand the possibilities of interpreter
Implement multithreading  of projects
