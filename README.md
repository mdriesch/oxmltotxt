# oxmltotxt: Open XML to text

[![PyPI Latest Release](https://img.shields.io/pypi/v/oxmltotxt.svg)](https://pypi.org/project/oxmltotxt/)
[![Package Status](https://img.shields.io/pypi/status/oxmltotxt.svg)](https://pypi.org/project/oxmltotxt/)
[![License](https://img.shields.io/pypi/l/oxmltotxt.svg)](https://github.com/mdriesch/oxmltotxt/blob/main/LICENSE)
[![Code style: black](https://img.shields.io/badge/code%20style-black-000000.svg)](https://github.com/psf/black)
[![Imports: isort](https://img.shields.io/badge/%20imports-isort-%231674b1?style=flat&labelColor=ef8336)](https://pycqa.github.io/isort/)

## What is it?

**oxmltotxt** is a Python package that helps extracting text (including vba!) from Open XML documents (see [here](https://support.microsoft.com/en-us/office/learn-about-file-formats-56dc3b55-7681-402e-a727-c59fa0884b30#:~:text=docx%2F.,using%20for%20your%20Office%20files.) for more details on Microsofts Open XML format). The idea is, to provide for a flexible, fast & efficient way to extract information from e.g. MS Excel files to use them together with git to enable version control of Open XML documents.

## How to install it?

Just install this package (ideally in an isolated virtual environment) from pypi:

```sh
pip install oxmltotxt
```

This will be enough, to use the text extraction feature of this package (if you want to try it out without git, just head over to [this section](#a-simple-use-case-for-the-package)). But to use it together with git, you will have to add some config changes to your local (or global, depending on your use case) git config settings. Don't worry, it should only take a few minutes to set this up.

### Preparing git

To use this package together with git, you will have to use git's [textconv](https://git.wiki.kernel.org/index.php/Textconv) tool. If you have experience in setting this up, you can skip this section.

This setup is OS and / or environment (shell) specific. The following section describes the setup necessary to enable the usage together with `git bash` on windows. If you want to use e.g. powershell you will have to adjust these steps.

- Copy the bash shell script `oxml2txt` contained in this repo ([link](https://github.com/mdriesch/oxmltotxt/blob/main/sample_setup/oxml2txt)) to a location on your git bash search path.

- To make sure, that this changes will take effect any time you open a new bash shell, you can e.g. extend your ~/.bashrc by adding the following lines to it's end:

  ```sh
  export PATH=$PATH:/path/to/oxm2txt_script
  export GIT_OXMLTOTXT=/path/to/oxmltotxt/venv/activators
  ```

  The second export is only needed, if you have installed `oxmltotxt` into a virtual environment (which I recommend). This variable is used to activate the virtual environment when the script `oxml2txt` is called. It therefore needs to point to the path, where the _activation_ scripts of your virtual env are located. On windows this is usually `/path/to/your/venv/Scripts`. Here you will find 3 activation scripts for: bash, dos batch and Powershell.

  If you have installed the package in your _global_ python environment, you can ignore this variable.
  To test your setup execute:

  ```sh
  which oxm2txt_script
  ```

  This should return the path to the location where the _oxm2txt_script_ is installed.

- Create a git repo containing your Open XML document (e.g. an MS Excel file with an extension like \*.xlsx))
- Edit your git config (located in the repo created in the previous step at .git/config), so that it will contain:

  ```
  [diff "msoffice"]
      textconv=oxml2txt <t1> <t2>
  ```

  This will tell git to use `oxml2txt` (a tiny bash wrapper script used in the bash context to call) as "the text converter" for all documents of type _msoffice_. Optional tags (\<t1>, \<t2>, ...) can be provided as needed (see [here](#a-simple-use-case-for-the-package) for use cases for \<tags>).

- Create / edit your .gitattributes within the repo, so that it will contain e.g.:

  ```
  *.xlsx diff=msoffice
  *.xlsm diff=msoffice
  ```

  This is yet another mapping telling git to use the text converter assigned to _msoffice_ documents to all files ending on .xlsx or .xlsm. Add other extensions as needed.
  Take a look [here](https://stackoverflow.com/a/28027656) to find out how to apply these changes _system-wide_.

## How to use it?

Once you have installed the package and possibly activated the virtual environment, you can start using the package.

### A simple use case for the package

A first, simple use case is, to extract _content_ from any Open XML format by issuing:

```python
python -m oxmltotxt.oxmltotxt /path/to/your/OXMLFile
```

When executing this command, you will get the xml content of e.g. your XL file as _nicely_ formatted text output. This should look similar to:

```xml
<!---- Start of file [Content_Types].xml ----!>
<?xml version="1.0" encoding="utf-8"?>
<Types xmlns="http://schemas.openxmlformats.org/package/2006/content-types">
 <Default ContentType="application/vnd.openxmlformats-package.relationships+xml" Extension="rels"/>
 <Default ContentType="application/xml" Extension="xml"/>
 ...
```

Notice the xml comment \<!---- Start of file [Content_Types].xml ----!> which indicates the relevant file with the zip file structure.

You can add _tags_ to this command:

```python
python -m oxmltotxt.oxmltotxt <t1> <t2> ... /path/to/your/OXMLFile
```

The tags \<t1>, \<t2>, ... will be used internally by [beautifulsoup4](https://pypi.org/project/beautifulsoup4/) to filter all tags relevant to your use case contained in each of the xml files within the zip file.

Formulas within XL files are tagged with `f` like in `<f>some formula</f>`, so if you try

```python
python -m oxmltotxt.oxmltotxt f /path/to/your/OXMLFile
```

on an XL workbook containing only one formula, you will get something along the lines of:

```xml
<!---- Start of file xl/worksheets/sheet1.xml ----!>
<f>
 A4+5
</f>
<!---- End of file xl/worksheets/sheet1.xml ----!>
```

### Intended use case for the package together with git

Now you are ready to use `git diff` on Open XML documents. A simple example would be, to store two empty XL files say _workbook1.xlsx_ and _workbook2.xlsx_. Just make sure the sheets are not exact copies (e.g. by just renaming the second workbook and _save it again_). Then do the following:

```sh
git diff --no-index workbook1.xlsx workbook2.xlsx
```

which will give an answer similar to this:

```diff
diff --git a/workbook1.xlsx b/workbook2.xlsx
index c54f836..017e614 100644
--- a/workbook1.xlsx
+++ b/workbook2.xlsx
@@ -34,7 +34,7 @@ Falling back to utf-8 decoding ...
   </mc:Choice>
  </mc:AlternateContent>
- <xr:revisionPtr documentId="13_ncr:1_{f876a870-98ec-4e82-91d5-47803a4d11c8}" revIDLastSave="0" xr10:uidLastSave="{00000000-0000-0000-0000-000000000000}" xr6:coauthVersionLast="47" xr6:coauthVersionMax="47"/>
+ <xr:revisionPtr documentId="13_ncr:1_{d0df556a-f8ad-4a22-b63e-7039adbe7d3d}" revIDLastSave="0" xr10:uidLastSave="{00000000-0000-0000-0000-000000000000}" xr6:coauthVersionLast="47" xr6:coauthVersionMax="47"/>
  <bookViews>
   <workbookView activeTab="1" windowHeight="11040" windowWidth="20730" xWindow="-120" xr2:uid="{2d46327d-59ce-41b3-b533-5578442c95a1}" yWindow="-120"/>
  </bookViews>
@@ -568,7 +568,7 @@ Falling back to utf-8 decoding ...
   2021-11-04T11:20:51Z
  </dcterms:created>
  <dcterms:modified xsi:type="dcterms:W3CDTF">
-  2022-07-30T05:34:51Z
+  2022-07-30T05:34:55Z
  </dcterms:modified>
 </cp:coreProperties>
 <!---- End of file docProps/core.xml ----!>
(END)
```

As you can see, although workbook2.xlsx is just a resaved copy of workbook1.xlsx **without any further changes** both files differ and `oxmltotxt` together with git will tell you where exactly they differ:

1. On each "save" of an XL file the class `documentId` within the tag `<xr:revisionPtr ...>` will be updated with a new [UUID](https://en.wikipedia.org/wiki/Universally_unique_identifier).
2. The tag `<dcterms:modified ...>` will be updated by the _datetime_ of the latest saving time.

### Spotting relevant changes

Depending on your use case, the _last save date_ or the color of a cell might be of limited interest to you. Maybe it even _breaks_ your version control workflow, because you don't consider this update as a _new version_ of the file.

In it's current version _oxmltotxt_ allows you to filter by a set of _relevant tags_ to look out for changes (see [here](#a-simple-use-case-for-the-package)).

To use the _tag feature_ together with git, one needs to update a line in the .git/config from

```sh
[diff "msoffice"]
	textconv=oxml2txt
```

to e.g.

```sh
[diff "msoffice"]
	textconv=oxml2txt f
```

where the `f` will be interpreted as _find all f tags_ (anything between \<f>..\</f>), which is how XL stores formulas.

Taking our _workbook1 / workbook2_ example from above, a simple formula update with enabled \<f> tag filter would look like this with `git diff`:

```diff
diff --git a/workbook1.xlsx b/workbook2.xlsx
index 5e22d8e..5a4ea5d 100644
--- a/workbook1.xlsx
+++ b/workbook2.xlsx
@@ -13,7 +13,7 @@ Falling back to utf-8 decoding ...
 <!---- End of file xl/_rels/workbook.xml.rels ----!>^M
 <!---- Start of file xl/worksheets/sheet1.xml ----!>^M
 <f>
- A1+1
+ A1+2^M
 </f>
 ^M
 <!---- End of file xl/worksheets/sheet1.xml ----!>^M
```

As you can see, the \<tag>-feature might be able to help you focus on changes that are relevant to your use case.

### VBA differences included

XL workbooks containing vba code will end on .xlsm instead of .xlsx.
Using our example workbooks from the previous [section](#extended-use-case-for-the-package-together-with-git) one has to create / rename two \*.xlsm files. Here I only changed the output of a simple vba `debug.print` statement.

```diff
diff --git a/workbook1.xlsm b/workbook2.xlsm
index 5676e81..8bf0f12 100644
--- a/workbook1.xlsm
+++ b/workbook2.xlsm
@@ -27,10 +27,9 @@ Falling back to utf-8 decoding ...
     Attribute VB_Name = "Modul1"^M
     ^M
     Public Function hello_world()^M
-        Debug.Print "Hello world"^M
+        Debug.Print "Hello world2"^M
     End Function^M
     ^M
-    ^M
   <!--- END xl/vbaProject.bin Modul1 Modul1 --->^M
 ^M
 <!---- End of file VBACode ----!>^M
```

### Analyzing effects of changes

The next example can be understood as a simple application of how one could use `oxmltotxt` to understand how information is stored within on Open XML document (make sure, that no tag-filters are active).

If, for example, you only change the color of cell `A1` to yellow in workbook2.xlsx and save again, you'll get something along the lines of (excerpt):

```diff
diff --git a/workbook1.xlsx b/workbook2.xlsx
index b5a635d..50c7bdd 100644
--- a/workbook1.xlsx
+++ b/workbook2.xlsx
...
  <sheetFormatPr baseColWidth="10" defaultRowHeight="15" x14ac:dyDescent="0.25"/>
- <sheetData/>
+ <sheetData>^M
+  <row r="1" spans="1:1" x14ac:dyDescent="0.25">^M
+   <c r="A1" s="1"/>^M
+  </row>^M
+ </sheetData>^M
  <pageMargins bottom="0.78740157499999996" footer="0.3" header="0.3" left="0.7" right="0.7" top="0.78740157499999996"/>
 </worksheet>^M
 <!---- End of file xl/worksheets/sheet1.xml ----!>^M
@@ -383,13 +387,19 @@ Falling back to utf-8 decoding ...
    <scheme val="minor"/>
    <scheme val="minor"/>
   </font>
  </fonts>
- <fills count="2">
+ <fills count="3">^M
   <fill>
    <patternFill patternType="none"/>
   </fill>
   <fill>
    <patternFill patternType="gray125"/>
   </fill>
+  <fill>^M
+   <patternFill patternType="solid">^M
+    <fgColor rgb="FFFFFF00"/>^M
+    <bgColor indexed="64"/>^M
+   </patternFill>^M
+  </fill>^M
  </fills>
  <borders count="1">
   <border>
@@ -403,8 +413,9 @@ Falling back to utf-8 decoding ...
  <cellStyleXfs count="1">
   <xf borderId="0" fillId="0" fontId="0" numFmtId="0"/>
  </cellStyleXfs>
- <cellXfs count="1">
+ <cellXfs count="2">^M
   <xf borderId="0" fillId="0" fontId="0" numFmtId="0" xfId="0"/>
+  <xf applyFill="1" borderId="0" fillId="2" fontId="0" numFmtId="0" xfId="0"/>^M
  </cellXfs>
...
```

## Where to get it

The source code is currently hosted on GitHub at:
https://github.com/mdriesch/oxmltotxt

Binary installers for the latest released version are available at the [Python
Package Index (PyPI)](https://pypi.org/project/oxmtotxt).

## Limitations

- There will probably be performance issues once this package is used on _larger_ Open XML documents. It took almost 10 min (on an old and small laptop, albeit) to just do the text extraction (i.e. not taking into account the diff part, which would include a second extraction run) on XL file containing just over 1 million formulas. There is probably a lot of optimization, which could be done.
- _Information overload_ in the sense of too many diffs could also get in the way, once there a large amounts of e.g. formula changes.

## Dependencies

- beautifulsoup4,
- lxml,
- oletools

## Inspiration

There are various tools around trying to do something similar. I would like to give credits specifically to [odt2txt](https://github.com/dstosberg/odt2txt/blob/master/odt2txt.1) which kind of inspired my approach for this package.

## License

[MIT](LICENSE)
