ATG Ant Plugin
==============

atg-ant-plugin is a set of build tools for making the compilation
of ATG Dynamo projects much easier. As its name suggests, this is
an [Apache Ant][ant] plugin forked from the original
[AtgAnt][atgantsfnet] plugin.

Installation
------------

First, download the source code:

    git clone https://github.com/jvz/atg-ant-plugin.git

Next, build and install the project to your local Ant lib directory:

    ant install

Usage
-----

The following is your ideal `build.xml` file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project name="base" default="clean-build" xmlns:atg="antlib:atg.tools.ant">
	<atg:atg home="${user.home}/ATG/ATG11.0"/>

    <target name="clean-build" description="Clean and build directory">
        <subant target="clean-build" genericantfile="atg-module-build.xml">
            <atgRequiredModules
                    atgHome="${atg.home}"
                    modules="proj3,proj1"
                    filters="proj*,base"/>
        </subant>
    </target>
</project>
```

The `taskdef` command loads in the new AtgAnt tasks. You should
point the `classpath` attribute of this task at the location of the
`atgant.jar` file.

The `atgRequiredModules` task provides a list of atg modules
referenced by the two modules given, using their `META-INF/MANIFEST.MF`'s
`ATG-Required` entries. The filters make sure we only run the
projects with the given names.

The code uses the list of ATG Modules to run another ant script
`atg-module-build.xml` over each of the found modules

The `atg-module-build.xml` file looks like this

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project name="base" default="clean-build">
    <property name="src.dir" value="src"/>
    <property name="build.dir" value="classes"/>

    <target name="-init">
        <available property="src.dir.exists" file="${src.dir}" type="dir"/>
        <atgModuleName atgHome="${atg.home}" property="module.name" module="."/>
        <echo message="ATG Module ${module.name}"/>
    </target>

    <target name="-clean">
        <delete dir="${build.dir}"/>
    </target>

    <target name="-compile-java" if="src.dir.exists">
        <mkdir dir="${build.dir}"/>
        <javac srcdir="${src.dir}"
               destdir="${build.dir}"
               debug="on">
            <classpath>
                <atgClasspath atgHome="${atg.home}" modules="."/>
            </classpath>
        </javac>
    </target>

    <target name="-build" depends="-compile-java"/>

    <target name="clean-build" depends="-init,-clean,-build"
            description="Clean and Build module"/>

</project>
```

This file will compile a single ATG Module, and the previous file
runs this ant script over all of your project's modules

The `atgModuleName` task is used to get the name of a particular
module, in this case the module "." -- this is a special name which
means my current build directory. Because this script is run over
each module, it will print out the name of the module it's current
working on.

The `atgClasspath` task generates the classpath of the current
module. This is the classes listed in the module's `META-INF/MANIFEST.MF`
file `ATG-Class-Path` entries, plus the `ATG-Class-Path` of all the
modules this module is dependent on.

More details on these commands is given later

This system could be extended to perform other ATG module related
tasks, such as running code quality tests or jsp compilation.

AtgClasspath task
-----------------

### Description

Gets the classpath for one or more ATG modules

The classpath is generated by referencing the modules'
`META-INF/MANIFEST.MF` file's `ATG-Class-Path` entries. If the
modules reference other modules using `ATG-Required` entries, then
those modules are added to the classpath. This happens recursively.

The classpath built is in the same order as that will be used by
ATG when the module is run.

The command is mainly used to generate the classpath for compiling
the java code in a module. It could also be used for the classpath
for [JUnit][junit] tests run against the code in the module.

### Parameters

* `atgHome`
  - The directory into which ATG has been installed.
  - Required.
* `modules`
  - A list of modules you want to include into the classpath, separated by commas, semi-colons or colons. The module "." means the current ant base directory. You can use simple wildcards such as `mod*` to indicate all modules starting with "mod", or `mod**` to mean all modules, including in subdirectories, starting with "mod". For example, `module1.base` would be matched by `mod**` but not by `mod*`.
  - Optional. Default is "."
* `filters`
  - Archive file to expand.
  - Optional.

AtgModuleName task
------------------

### Description

Gets the name of the ATG module given to it.

The name of a module is its name as would be recognised by another
ATG module: e.g., `PublishingAgent.base`.

The command is mainly used to get the name of the module which is
the current Ant base directory, but it could be used simply to
convert directory names from the system format classpath to ATG's
dot notation.

### Parameters

* `atgHome`
  - The directory into which ATG has been installed.
  - Required.
* `module`
  - The module whose name is being sought.
  - Optional. Default is "."
* `dir`
  - The directory of the module whose name is being sought. You can not specify both `dir` and `module`.
  - Optional. Default is the value from `module`.

AtgRequiredModules task
-----------------------

### Description

Gets the list of modules need by (and including) the modules given.

The classpath is generated by referencing the modules'
`META-INF/MANIFEST.MF` file's `Required` entries. This happens
recursively.

The classpath built is in the same order as that will be used by
ATG when the module is run, with the base modules appearing first.

The command is mainly used to generate a list of modules that need
to be compiled for a particular system. For example, if you provide
your top level module or list of modules to make a .ear, then it
will list all the modules that need to be compiled.

This list will include your custom modules, and ATG's proprietary
modules. You may need to use the `filter` parameter to restrict the
list of modules returned

### Parameters

* `atgHome`
  - The directory into which ATG has been installed.
  - Required.
* `modules`
  - A list of modules you want to include into the classpath, separated by commas, semi-colons or colons. The module "." means the current ant base directory. You can use simple wildcards such as `mod*` to indicate all modules starting with "mod", or `mod**` to mean all modules, including in subdirectories, starting with "mod". For example, `module1.base` would be matched by `mod**` but not by `mod*`.
  - Optional. Default is "."
* `filters`
  - Archive file to expand.
  - Optional.

Notes
-----

Developed by Piran Montford, with the kind help of e2x Ltd. Updated by @jvz.

Warranty THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY
KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES
OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
IN NO EVENT SHALL THE DEVELOPERS OR ANY OTHER CONTRIBUTOR BE LIABLE
FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF
CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

[atgplugin]: https://github.com/jvz/atg-gradle-plugin
[atgantsfnet]: http://atgant.sf.net/
[junit]: http://junit.org/
[ant]: http://ant.apache.org/
