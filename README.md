Easy-PQXX Build for Windows Visual Studio (0.0.6)
-------------------------------------------------

[PQXX](http://pqxx.org/development/libpqxx/) is the official C++ client API
for PostgreSQL. Build the libpqxx library for Windows quickly, with both
Debug and Release configurations, install in Program Files, fix problems,
create property sheets to easily use in Visual Studio applications, with
named versions for configurations and options. Skip past the notes to
[Easy-PQXX Batch Build for Visual
Studio](#easy-pqxx-batch-build-for-visual-studio) to jump right in with:

1. Automated batch file to build and install in the selected location
   (typically C:\Program Files\libpqxx), asking for administrator
   privileges if needed to copy the files.
2. EASY_PATCH settings for compile flags for features that are not enabled
   by CMake, allowing more efficient code in Windows.
3. Configure for Unicode or MultiByte character sets. (For example Unicode
   character set may be needed to interface wxWidgets in the same
   application.)
4. Keep named installations for different configurations.
5. One library installation with both Debug and Release compiles that
   automatically switch in Visual Studio.
6. Property Sheets so your Visual Studio application (with both Debug and
   Release compiles) can be instantly configured.
7. Gather DLLs from PostgreSQL library into your application automatically.

Additionally various configuration solving selections can be made, for
example if you have more than one PostgreSQL library or want to compile for
more than one version of Visual Studio. Sometimes the basic CMake default
process does not work out with an ideal result.

Notes
-----

The new [libpqxx](http://pqxx.org/development/libpqxx/) version 7 and
beyond can most easily be built for Windows Visual Studio using the "CMake"
build process. Since version 7 requires C++ 17 standard, to build for
Microsoft Visual Studio the 2017 version of Visual Studio or later is
required. And of course you would start with downloading the [source code
on GitHub](https://github.com/jtv/libpqxx) for the version of libpqxx that
is right for you. In this explanation the `libpqxx-master` is the current
development version, and the release versions have names like
`libpqxx-7.0.0`, which is the earliest version that requires CMake to
build.
 
CMake is a wonderful "make" like system that runs on a large variety of
operating systems. This allows a single build methodology to be used by the
libpqxx developers which targets an enormous variety of operating systems
and compiler options. However due to the requirements of compatibility of
this large number of target systems, bugs and issues with CMake, and
programming methods needed for the CMake process to be compatible with this
large number of targets, some problems remain for the specific installation
on Windows operating system using Visual Studio compiler.

CMake is available for several compilers for Windows, including Microsoft
Visual Studio. If you are using Visual Studio, only the 2017 and later
versions support the C++ 17 standard required by libpqxx 7. And libpqxx
version 7 can most easily be compiled for Visual Studio using the CMake
build process.
 
The methods here show how to install in Windows, and get past Windows
install problems if they occur for you. A configurable batch file is
presented which will fully automate the installation of the libpqxx library
with options and features discussed in the introduction.

Basics of the Cmake Build for Visual Studio
-------------------------------------------

You can do the basic CMake build process without using the batch file and
advanced features, and understand the underlying steps, according to these
instructions. The first example will build and install only one
configuration, and can't be directly placed in a privileged location like
`C:\Program Files`.
 
In my opinion the best way to build a static libpqxx library on Windows for
Visual Studio is an "out-of-source build" using CMake, in which source and
build files are subdirectories of a top-level directory. (Check that your
[version of CMake](https://cmake.org/) is up to date. The minimum CMake
version should be 3.17.1 and the latest is preferred. And of course a late
version of [PostgreSQL
database](https://www.postgresql.org/download/windows/) must also be
installed, version 12 is preferred but at least version 10. Version 10 is
the last with a 32 bit Win32 installation.) In this method you put the
input source files subdirectory (`libpqxx-master` or `libpqxx-<version>`)
into the top-level directory. Then in this top-level directory execute the
following two commands (adjusting the name of the input source
accordingly):

```
cmake -S libpqxx-master -B build -DCMAKE_INSTALL_PREFIX="libpqxx"
```

This constructs a Visual Studio solution with multiple projects and CMake
build information, in the 'build' subdirectory, but does not build those
projects. The CMAKE_INSTALL_PREFIX parameter establishes the final install
directory in advance as a third subdirectory 'libpqxx', of the top level
directory. This cannot be located in the protected Program Files directory
without running CMake as administrator. Leaving off the
CMAKE_INSTALL_PREFIX parameter usually causes CMake to default to
`C:\Program Files (x86)\libpqxx` in the next step. This is often the wrong
install subdirectory, requires administrative privileges or will fail, and
those privileges are not granted just by logging in as admin.
 
Then
```
cmake --build build --target INSTALL
```

will build the projects using the native Visual Studio tools, including
copying the final installation files into the libpqxx subdirectory of your
build directory, as previously established. That libpqxx subdirectory can
be manually copied into the `C:\Program Files` directory if desired, to
give it a standardized location that may be expected on Windows. A manual
copy will ask for administrative rights.

That last step is actually two separate actions, and they could also be
accomplished by using the native Microsoft build tools directly within the
build that is configured in that first CMake step. That Microsoft Visual
Studio solution, called `libpqxx.sln` will contain projects to build the
library and to organize the generated files into an "install" directory.
That final `INSTALL` step exists as a project in the Visual Studio
solution, but actually just calls back to the CMake system to script the
"installation" into an organized output subdirectory. The `--target
INSTALL` flag triggers that final `INSTALL` step, which depends upon the
building of the library. (Since this is a "make" system, the entire library
build is triggered by this one command, followed by the install process.)

A version of PostgreSQL [must be installed on the build
computer](https://www.postgresql.org/download/windows/), at least the
libraries must be installed, in order to build the libpqxx library. See the
notes on additional flags below if the first CMake command cannot locate
the correct PostgreSQL library automatically.

One way to automate this build process is to copy the following lines and
paste into a good programming editor, which will fix any Windows new line
problems. Then save as `build.bat` batch script file, into your top-level
directory. Click on the batch file to automate the build process:

```bat
REM Automated build of libpqxx placing install files in libpqxx subdirectory.
cmake -S libpqxx-master -B build -DCMAKE_INSTALL_PREFIX="libpqxx"
if %errorlevel%==0  cmake --build build --target INSTALL
pause
```

Any changes can be made by clicking the batch build file again and only
necessary changes will be compiled, as is normal for a "make" process. Once
again, use `libpqxx-master` for the most recent build of libpqxx, or change
that to match the `libpqxx-<version>` number you are using, here and in the
examples that follow.
 
Flags that can be added on the first `cmake` line include: `-A x64` or `-A
Win32` These will select the target architecture as 64 or 32 bit. `-G
"Visual Studio 15 2017"`  may be used to select the 2017 version of Visual
Studio, or explicitly select `-G "Visual Studio 16 2019"` (default  if
installed). The architecture and generator parameter values must be exactly
as shown. Use 
`-DPostgreSQL_ROOT="C:\Program Files\PostgreSQL\12"`
to select the version of PostgreSQL at the specified location. To compile
for Win32, the `"C:\Program Files (x86)\PostgreSQL\10"` version of libpq
will probably be required from the x86 directory (and include `-A Win32`).
These flags can also be added to the first `cmake` line in the example
below.

Some additional flags that can be used on the first `cmake` command line,
as noted in libpqxx's `BUILDING-cmake.md` document: `-DSKIP_BUILD_TEST=on`
skips compiling libpqxx's tests. `-DBUILD_SHARED_LIBS=on` to build a shared
library, a DLL version of libpqxx library. `-DBUILD_SHARED_LIBS=off` to
build a static library. `-DBUILD_DOC=on` to use Doxygen to build the
documentation (not related to whether documentation is present in the
installation). (None of this last set of flags have been implemented in the
current version of the `Easy-PQXX` batch file presented below, by the way.)

The second (build) `cmake` command can also have: `--config Release` to
switch the install from a Debug to a Release configuration.

The build `cmake` line can also have, at the very end, `--
/property:CharacterSet=Unicode` to change the character set from MultiByte
to Unicode. That flag (including the `--` separated by spaces at the end
before the added flag) can also be applied in the 2nd and 3rd `cmake` build
lines in the example below to change the character set, and the added flag
is passed directly to the native compiler tool without `cmake` interpreting
it.

A slightly more complex sequence of commands will build (completely
separate) Debug and Release installations:

```bat
REM Automated build of libpqxx, both Debug and Release
cmake -S libpqxx-master -B build
if %errorlevel%==0  cmake --build build   --config Debug
if %errorlevel%==0  cmake --build build   --config Release
if %errorlevel%==0  cmake --install build --config Debug   --prefix "libpqxx/Debug"
if %errorlevel%==0  cmake --install build --config Release --prefix "libpqxx/Release"
pause
```
See the notes above for added flags that can be used in this example as
well. (Note most flags listed above go on the first `cmake` line, but the
character set flags go at the end of the two `--build` lines.)

If you copy the `basic_libpqxx.props` property sheet from this repository
to the `libpqxx` subdirectory created above, and edit line 17 to correct
the location of the PostgreSQL library if necessary, then you can include
that property sheet in your new applications. This means you don't have to
manually enter the required lists of libraries and their locations into
your programs to use the new library. Just use the Property Manager tab in
Visual Studio, and right click on project to select `Add Existing Property
Sheet...` to incorporate that. You should not have errors in include files
or libraries in your new application regarding libpqxx.

If you want to automatically copy the required DLLs from PostgreSQL into
your executable directory in your new project, then also add the
`basic_DLL.props` property sheet to your project configurations. That
property page should be saved in the libpqxx subdirectory also, and next to
that file add a `bin` subdirectory into which you copy needed DLL files
from PostgreSQL as you discover them. 

Each time you run your application it will probably ask for a DLL file,
until you fill out the entire set needed and build them into the executable
directory. For version 12 PostgreSQL, x64, the following list of DLL files
was needed: `libpq.dll` `libcrypto-1_1-x64.dll` `libiconv-2.dll`
`libintl-8.dll` `libssl-1_1-x64.dll`. For version 10, Win32, the files are
`libpq.dll` `libcrypto-1_1.dll` `libiconv-2.dll` `libintl-8.dll`
`libssl-1_1.dll`.

If you want the final installation of libpqxx in the `C:\Program
Files\libpqxx` directory, you could just copy and paste the `libpqxx`
subdirectory created above into your Program Files directory--and Windows
will ask for permissions. 

A couple of compiling configuration flags that CMake does not set in
Windows, even though the features are available, are as follows:
```cpp
#define PQXX_HAVE_CHARCONV_FLOAT
#define PQXX_HAVE_CHARCONV_INT
```
In your `build\include\pqxx` directory, are three files:
```
config-internal-compiler.h
config-internal-libpq.h
config-public-compiler.h
```
Adding those definitions at the end of each of those three files will
enable efficiency features in libpqxx. As of version 7.1.2 "master" in
development on June 15, 2020, some changes are needed before these
efficiency modifications will work. But after those fixes are made so the
library compiles for Visual Studio, those definitions can be added.

To make those changes, edit the files after the very first CMake step which
constructs the Visual Studio solution files in your `build` directory.

Easy-PQXX Batch Build for Visual Studio
---------------------------------------

The batch file `Easy-PQXX.bat` supplied here does basically the same thing
as the last example above, building libpqxx with both Release and Debug
libraries, but also creates appropriate property sheet for use of the
library, installs the libraries and DLLs in a standardized location
(usually C:\Program Files\libpqxx), and automatically asks for admin
privileges. Optionally the patches needed to implement efficiency changes
can be applied between constructing the solution and the compiling it. It
places the library in a named subdirectory of libpqxx install, so that
variously configured installs have different names. Furthermore it
organizes the installation as a single directory with include and 'share'
directories (with documentation), but the lib directory has both a Debug
and Release subdirectory with the respective library. (The simplified
example above wound up with two copies of the include and the documentation
directories.)

Three other batch files are provided in this repository:
`Easy-PQXX-patch.bat` is pre-configured to patch the CMake generated files
with efficiency modifications, consistent with the latest development copy
of libpqxx-master. `Easy-PQXX-x64Unicode.bat` is pre-configured for Unicode
character set as well as the efficiency modifications in a 64 bit
compilation. (Useful along with wxWidgets library.) `Easy-PQXX-Win32.bat`
is pre-configured for Win32 (x86) compiling. The latter has the typical 32
bit PostgreSQL location selected. 

In all cases the user may have to make some configuration changes to adapt
the use. The `Easy-PQXX.bat` is always the master copy, and the others are
derived by copying and changing one or more of the settings. If there is
ever a disparity, `Easy-PQXX.bat` is the maintained master copy.

This batch file has a top section to allow the user to configure the build.
As noted in the batch file, it should be copied to the top level directory
used to build the libpqxx system, **using your favorite programming text
editor.** Since presumably you are using Visual Studio 2017 or later for
the compile (you wouldn't be here otherwise), Visual Studio can be used for
that purpose, and check that you have the menu `Edit--Advanced--End of Line
Sequence` set to CRLF when you save. (Hopefully any code-page issues would
be resolved in this step as well.) Set the configuration variables (top
section) to the values you want, then save the edited batch file in that
top level directory for the install.

**Please do not change the operations sections of this batch file! This
batch file can recursively run itself as administrator for the last
installation step (which just uses `xcopy` to place files), but that gives
the batch file privileges to do almost anything. Making changes if you do
not completely understand them can have unexpected results.**

Please do change the configuration variables in the top section to match
your requirements. Each variable has an explanation.

To activate the batch file (in the top-level subdirectory for the project)
just click (or double click) the file. It will open in a command window,
and has a pause at the end so you can read the results. After build it may
ask permissions to run in administrative mode to do the final copy to the
Program Files directory. After giving permission, a yellow background
command window is doing the final copy. Then hit a key to end each window,
after checking for proper operation.
 
This batch file first configures the build, applies patches to the
configuration if specified, compiles the libraries (both Release and Debug
as selected), but in separate local pre-installation work directories. The
CMake methods are only configured to do this as separate file groupings.

Then the batch file copies the separate installations into a singe coherent
directory with just one include subdirectory, just one 'share' subdirectory
with documentation, and with a lib directory that is subdivided by Debug
and Release folders. Another subdirectory, 'bin' is added to contain the
external PostgreSQL libraries and DLL files required for applications using
the libpqxx library. Property page files are also provided, which can be
used in Visual Studio projects to easily reference the libraries and DLL
files needed.

Using the PQXX Library
----------------------

Create your new project in Visual Studio. (Often a completely new project
will be required, because the changes are significant from previous
versions if you are upgrading old work. For example you don't need to
provide locations for libraries and include files any more!)

The first setting that is **required** is to change your C++ Language
Standard to ISO C++ 17. (We can't do that from Property Sheets.)

Then in the "Property Manager" tab, select each of your Debug and Release
versions (in sequence). Right click and "Add Existing Property Sheet".
Unfortunately VS seems to forget where you were, so you have to click
around to the install directory each time. Find the appropriately named
subdirectory of the libpqxx install directory, for the respective build
library to be used. At the base of that directory are three property sheets
(.props) named libpqxx, libpqxx_ALL, and libpqxx_DLL. The first is just the
library (no DLL). The _DLL version just copies the DLL's to your executable
directory as a post-build event. And the _ALL version does it all. Select
the libpqxx_ALL.props sheet. (You need to do this for both the Debug and
the Release configurations, but use the same property sheet for both.)
 
Then compile your application! (Thats all.)

You can switch your application between Debug and Release configurations
and the correctly built library is chosen automatically.

(If you are building both x64 and Win32 versions of your application, you
will have to configure a 2nd copy of the batch file for Win32, and the
output libraries will appear in a separate subdirectory of the libpqxx
installation. When adding property sheets, add the sheets from the
appropriate subdirectory in the respective configuration from the Property
Manager.)

Note that the installation directory looks like this:
```
bin
include
lib
libpqxx.props
libpqxx_ALL.props
libpqxx_DLL.props
share
```

The lib subdirectory is different from the normal libpqxx make, since it
contains separate Debug and Release subdirectories, each with a library.
The libpqxx_ALL.props property page causes the application to refer to the
library, the include files, and also copy the DLLs for PostgreSQL to the
execute directory. The libpqxx.props refers to the library, the include
files, but does not copy the DLLs. The libpqxx_DLL.props property page only
copies the DLLs. (This last can be used in the test suite generated in the
CMake build to get it going quickly.)

Running the Test Suite
----------------------

In the configured Visual Studio solution is a test suite with project name
"runner". To run this test suite to test the build, some steps need to be
done in this Windows version. Note that the build inserts references to the
local copies of the library and references the insert files path from the
build source and build directories. However the executable may not have
available the DLL files from PostgreSQL. 

Those DLLs could be made available by putting the bin directory of the
PostgreSQL installation of the Program Files directory into the path for
the local user. (This has an advantage of providing access to the `psql`
utility, for example, in any open command window without special
environment settings.) However this may also become troublesome if you have
both 64 and 32 bit versions of PostgreSQL installed, for example.

Another way to provide those DLLs with little effort is to use the property
page for the respective compile to load the DLLs with the runner
application. Open the libpqxx.sln solution in the build subdirectory with
the Visual Studio IDE. The DLL only property page will be used. However we
are lucky that there is only one architecture specified in the libpqxx
projects, so one property page insertion can apply to all the
sub-configurations. In the Property Manager tab, look for the 'runner'
application. Right click on that (as a whole, not its individual
configurations) and right click and pick Add Existing Property Sheet. You
could use the libpqxx_DLL.props property sheet from the final installation,
but I just choose the local install subdirectory matching the installation
type, since it is in a fixed relative position to the library build, like
the other files used in this test project.

Then you need to find or create a PostgreSQL database for the testing
application play in. To connect that, set the environment variables, here
shown on the local PostgreSQL server. Of course use the appropriate values
for these variables, especially the name of the testing database you have
chosen, and appropriate user. 

```
PGHOST=localhost
PGPORT=5432
PGDATABASE=testing
PGUSER=postgres
```

To do this, find the project properties debugging environment
configuration.

You may have to add `PGPASSWORD=` setting to the list if you don't have a
pgpass.conf file configured for applications (like .pgpass on Linux). (See
https://www.postgresql.org/docs/current/libpq-pgpass.html for pgpass file
configuration.) But be forewarned placing the password in your program is
probably a security risk, the pgpass file is probably better.

With only the Property Page for the DLL include added to the runner
application, and the environment variables supplied above, the runner test
suite should run correctly by starting from the Visual Studio IDE.
