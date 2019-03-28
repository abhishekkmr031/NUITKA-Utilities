# NUITKA-Utilities
A collection of scripts involving Python compilations with NUITKA

-------
## exe-maker.py (currently tested on Windows)
This script shows a GUI (using tkinter / [PySimpleGUI](https://github.com/MikeTheWatchGuy/PySimpleGUI)) to ask for a Python script name and then invokes NUITKA to generate a **standalone EXE** file from it.

### Features
* sets a number of NUITKA default parameters
* several configuration options
* arbitrary additional NUITKA parameters
* optional invocation of UPX packer for binary output
* optional request to rebuild the dependency cache
* experimental: button to remove unneeded binaries (**"skimming"**)

### Note 1
A central folder to merge several binary `.dist` folders is no longer supported by this script. Use `exe-merger.py` for this purpose - it contains all the required functiuonality.

### Note 2
If your program uses Tkinter, you **must check** the respective button. This requirement only exists on Windows platforms.

> This script is no longer maintained. Please use ``exe-maker2.py`` instead - or even better: use the **user plugin** ``make-exe.py``, see below.

-----
## upx-packer.py
NUITKA binary output folders tend to have sizes in excess of 60 MB. While this is largely irrelevant if you continue to use the compiles on the original computer, it may become an issue when distributing stuff.

If you want to reduce your binaries' **distribution** size, the obvious way is to create a self-extracing archive. The compression results for NUITKA binaries are generally very good and yield sizes of 25% or less of the original by using e.g. 7zip. As a matter of course, the original size is re-created on the target computer.

This script in contrast aims to reduce the **current binary folder size** by UPX-compressing all eligible binaries ("exe", "pyd" and most of the "dll" files).

### Features
* Takes a folder and recursively compresses each eligible file by applying UPX to it. The compressions are started as sub-tasks -- so overall execution time depends on the number of available CPUs on your machine.
* It assumes that the ``upx.exe`` executable is contained on a path definition. Otherwise please change the script accordingly.
* Binaries are compressed *in-place*, so the **folder will have changed** after execution.
* Depending on the folder content, the resulting size should be significantly less than 50% of the original -- expect something like a 60% reduction.
* I am filtering out a number of binaries, which I found make the EXE files no longer executable. Among these are several PyQt binaries. Add more where you run into problems -- and please submit issues in these cases.

### Note
The resulting folder is still somewhat compressible e.g. when creating a self-extracting archive, but not as well as without applying the script.

Sample output:

```
D:\Jorj\Desktop\test-folder>python upx-packer.py bin
UPX Compression of binaries in folder 'D:\Jorj\Desktop\test-folder\bin'
Started 101 compressions out of 127 total files ...
Finished in 20 seconds.

Folder Compression Results (MB)
before: 108.45
after: 46.70
savings: 61.75 (56.94%)

D:\Jorj\Desktop\test-folder>
```
The self-extracting archive of the resulting **packed** `bin` folder (7zip) has a size of **34.8 MB**.

-----
## upx-unpacker.py
Does the opposite of `upx-packer.py`.

Use it to undo a upx-compression -- for example if you encounter problems.

> Please note that - at least under Windows - UPX decompression **does not restore** binary identity with the original. Therefore, merging new compiles into a packed or unpacked folder ***will always fail*** when using `exe-maker.py`. But look at script ``exe-merger.py`` for ways out of this stumble block.

```
D:\Jorj\Desktop\rest-folder>python upx-unpacker.py bin
UPX De-Compression of binaries in folder 'D:\Jorj\Desktop\test-folder\bin'
Started 119 de-compressions out of 127 total files ...
Finished in 2 seconds.

Folder De-Compression Results (MB)
before: 46.70
after: 108.45
growth: 61.75 (132.24%)

D:\Jorj\Desktop\test-folder>
```
The self-extracting archive of the **unpacked** `bin` folder has a size of **28.7 MB**, i.e. notably smaller than a UPX-compressed folder.

### Note
The binaries which the script tries to decompress are more than during compression, because a less restrictive selection is applied.

This is no problem: UPX ignores files that it didn't compress previously.

Decompression runtime is very short anyway.

-----
## exe-merger.py
Yet another script to merge two folders with binaries. Can be used if `exe-maker.py` refuses to merge compilation output because of "incompatible" binaries.

This script can resolve all "incompatibility" situations via a "force-mode" merge. In this case source files overwrite same-named files in the target.

> If you want to exercise greater care, you can first try to compress or decompress both, source and target folders repeatedly and then use this script again:
>
> * after first decompression, UPX will **not re-instate** binary equality with the original file. But after another round of compression / decompression, results will remain stable and you should be able to successfully merge them.

You will finally have a merged target folder, which you then again can (de-) compress as required.

-----
## link-maker.py
Scan a folder for ``.exe`` files and create a ``.lnk`` (link) file for each of them. If the specified folder has no ``.exe`` files, its ``bin`` sub-folder will be checked (if present).

Output folder default is the user desktop.

Script runs under Windows only and requires packages ``pythoncom`` and ``win32com`` to be installed.

-----
## exe-maker2.py
This is an advanced version of exe-maker. It supports the following features in addition:

1. Automatical use of the new dependency checker (based on [pefile](https://github.com/erocarrera/pefile)), which currently still is in experimental state. This feature uses a Python package to trace down binaries that are used by the script. Experience so far looks very promising - especially in terms of execution speed.
2. A new Nuitka plugin for ``Tkinter``. This obsoletes any special precautions by the programmer: The plugin will automatically copy Tkinter libraries and re-direct Tk requests to them.
    > The tkinter plugin is only relevant for Windows and should not be used on other platforms. If used on non-Windows platforms it will de-activate itself and issue a warning.
3. A new Nuitka plugin for ``numpy``. If your Numpy installation is not the *"vanilla"* version from PyPI, but includes acceleration libraries (like Intel's MKL, installations via [blis](https://pypi.org/project/blis/), OpenBlas, and similar), then you probably must request support for Numpy explicitely, similar to that for Qt and Tk.
4. exe-maker2.py will actively avoid scanning for imported packages that are not checkmarked, and will delete any releated binaries from the ``dist`` folder.

> This script is being phased out. Please use the user plugin ``make-exe.py`` below. It offers a seamless use and better integration with Nuitka.

-----
## onefile-maker.py (working on Windows)
Turns the ``dist`` folder of a standalone compiled script (e.g. ``script.py``) into an executable file, which can be distributed / handled like an installation file. Its name equals that of the script's EXE name (i.e. ``script.exe``).

When executed on the target computer, the installation file will first decompress itself into the user's ``$TEMP`` (environment variable) folder and then invoke the **original** ``script.exe``, passing any arguments to it.

After the Python program finishes, the folder will be deleted again from ``$TEMP``.

Alternatively, you can do the following:

Execute the file with parameter ``/D=<folder>`` specifying a directory of your choice. The ``dist`` folder will then be decompressed into ``<folder>`` and nothing else will happen.

## make-exe.py
This script is a Nuitka **user plugin**. It is intended to **replace** ``exe-maker.py`` and ``exe-maker2.py``, as it obsoletes the use of separate scripts to achieve the same results. In addition, it also works on Linux platforms (see comments below). This is how it is used:

```
python -m nuitka --standalone ... --user-plugin=make-exe.py=options <yourscript.py>
```

The string ``"=options"`` following the script name is optional and can be used to pass options to the plugin. If used, it must consist of keyword strings separated by comma. The following are currently supported:

* **qt, tk, np**: Activate / enable the respective standard plugin ``"qt-plugins"``, ``"tk-plugin"``, or ``"numpy-plugin"``. The option can be prefixed with ``"no"``. In this case, the corresponding plugin is **disabled** and recursing to certain modules / packages is suppressed. For example: ``"nonp"`` generates the options ``--recurse-not-to=numpy`` and ``--disable-plugin=numpy-plugin``.
* **onefile**: After successfully compiling the script, create a software distribution file in "OneFile" format from the script's ``".dist"`` folder. This format is an executable file which, when executed, automatically decompresses itself into a temporary folder, and then invokes the binary excutable in it that corresponds to the compiled script.
* **onedir**: Much like "OneFile", an executable file is created from the ``".dist"`` folder. But when executed on the target system, it decompresses its content into a specified folder and exits.
* **upx**: After successful compilation, invoke the binary compression program UPX to compress the ``".dist"`` folder.

Keywords ``onefile``, ``onedir`` and ``upx`` are mutually exclusive (other options can be combined). For each of these options, the plugin invokes the corresponding script for the platform, e.g. there exist different versions for Windows and Linux of the script ``onefile-maker.py``, etc.

> Please note that the downstream scripts (corresponding to onefile, onefile, upx) are still under development and not all of them are available on all platforms yet.

Example command:
```
python -m nuitka --standalone ... --user-plugin=make-exe.py=notk,qt,onefile <yourscript.py>
```

> Although it obviously is handy to put the plugin script in the same folder as ``<yourscript.py>``, this is not required -  just add enough information to locate it as a normal file. This also applies to any downstream scripts (like for onefile option, etc.).

> There is no general decision yet, how we want to distribute user plugins etc. with the main repository, if at all.


## hints.zip
This file contains 4 Python scripts, which collectively have the goal to create standalone compile distribution folders, which contain **no unneeded stuff**.

1. ``hints.py`` - a logger which intercepts and records every import statement made in a program.
2. ``get-hints.py`` - a script which executes your test script under the control of ``hints.py``. When your script ends, it creates a consolidated ``yourscript.json`` file.
3. ``hinted-mods.py`` - a Nuitka user plugin, which is used while your program gets compiled. It reads the JSON file from before and decides for each module or package, whether it should become part of the ``yourscript.dist`` folder.
4. ``nuitka-hints.py`` - invokes the Nuitka compiler in standalone mode. It also passes a number of other command line options to it. Among those is the user plugin ``hinted-mods.py``.

I invite you to use and test this little package. It is still work in progress - so please do provide feedback.

1. Execute ``python get-hints.py yourscript.py``. Your script will be executed like normal in interpreter mode. But in the end, a protocol of all imported packages / modules has been created and put in file ``yourscript.json`` (same folder as ``yourscript.py``).
2. Now compile your script like this: ``python nuitka-hints.py yourscript.py``. This is a standalone compile. It automatically generates all required parameters - including numpy or tkinter plugins. If you need other parameters, you can use them like normal (e.g. for using the Qt plugin).

### Remarks
* Look at the list of Nuitka options contained in ``nuitka-hints.py``. Chances are that you want to adjust them for your environment.
* If you intend to use ``scipy`` on **Windows**, please be aware that there still is plugin support missing. The scipy package comes with some extra DLLs, which are undetectable by a compiler, so a plugin must take care of them (work in progress).
* I am currently working on some improvements:
    - include implicit imports in the JSON file (done - being tested)
    - ad support for other standard plugins
    - add ``scipy`` support
