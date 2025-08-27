# Grant Employability Report Application (GERA)

<img src="https://github.com/purplesprinklesdev/gera_docs/blob/main/resources/logo.png" width="120" align="right">

A lightweight desktop application that generates Visual Employability Reports from student data
- Highly performant Rust backend to handle data between file system and an In-Memory SQLite database
- Tauri desktop app with a simple SvelteKit frontend
- Powerful [Views and Patches system](#custom-views-and-patches) for user-defined automation
- Fully offline and FERPA compliant

This application is specifically designed for use in combination with Grant Career Center's existing proprietary technology and is *not* a broad student data processing solution. This documentation is intended for Grant employees using GERA and future maintainers of the software.

GERA's source code is property of Grant Career Center.

## Table of Contents
- For Users
  - [Installation](#installation)
  - [How to Use](#how-to-use)
    - [Export From Access](#export-from-access)
    - [Step 1: Select Workspace and Quarter](#step-1-select-workspace-and-quarter)
    - [Step 2: Table Fixer](#step-2-table-fixer)
      - [Preserving Manual Corrections](#preserving-manual-corrections)
      - [Employability Score Recalculation](#employability-score-recalculation)
      - [Custom Views and Patches](#custom-views-and-patches)
    - [Step 3: Generate PDFs](#step-3-generate-pdfs)
      - [Chrome Requirement](#chrome-requirement)
    - [Avoiding Conflicts with OneDrive](#avoiding-conflicts-with-onedrive)
    - [Avoiding Conflicts with Excel](#avoiding-conflicts-with-excel)
  - [Configuration](#configuration)
    - [Logging](#logging)
    - [PDFs and Charts](#pdfs-and-charts)
    - [Simple Graph Style Changes](#simple-graph-style-changes)
    - [Grade Category Distribution](#grade-category-distribution)
    - [Views and Patches System](#views-and-patches-system)
    - [Resetting Configuration](#resetting-configuration)
  - [FERPA Compliance](#ferpa-compliance)
- For Developers
  - Tech Stack
  - Cloning & Build Process
  - Understanding the Configuration System
  - Table Fixer Process
  - PDF Generation Process


# Installation

Simply run the GERA installer exe and follow the wizard's instructions. Launch either through a desktop shortcut, or by searching in the Start menu.

Open the modified Access Database which should include a button named "Export To CSV". If you see that button on the main menu, there's nothing else you have to do.

# How to Use

GERA builds off of functionality already a part of the GrantEMP Access program. Start by importing data from the HCCA servers into Access. Then use the provided Export to CSV script to get the data ready for GERA.

### Export From Access

Open Access, import new data from DASL and ProgressBook if necessary, then use the Export To CSV button on the Main Menu. Select the folder you want to be your workspace. Make sure it [isn't set to sync to OneDrive](#avoiding-conflicts-with-onedrive).

After the export has finished, you should see the following CSV files in your workspace folder:
- StudentEmployabilityData
- StudentDailyAbsence
- StudentDemographic
- StudentDiscipline
- vwStudentClassAverage

Now you have the opportunity to look through the data in the five input tables and correct values if absolutely neccessary, but **it's recommended to wait until you have ran Step 2: Table Fixer in GERA before doing manual corrections.**

## Step 1: Select Workspace and Quarter

Now it's time to open GERA. Select the same folder you exported the tables from Excel into, and pick a quarter. Choosing the right quarter for your data is very important. Below are all the features that take quarters into account:

- [Preserving past manual corrections](#preserving-manual-corrections). Manual corrections will be preserved if they are on a previous quarter's value. Selecting First Quarter will not preserve any corrections, selecting Third will preserve ones in Qtr1 and Qtr2 columns
- [Employability Score Recalculation](#employability-score-recalculation). All previous quarters and the current quarter will have their employability scores recalculated. The `OverallEmployabilityScore` column will average the employability scores in previous quarters and the current quarter, but not future ones.
- PDFs. Impacts the naming of PDFs, the number of bars each graph has, and the number of rows the table has.

Even if your tables have more data filled in than your selected quarter would suggest, GERA will handle it and ignore the "future" data. This means you can redo past quarters even when newer data is present. The only problem this could cause is that manual corrections will be overridden. Create a backup of the `CampusDataReport.csv` file if there were manual corrections you don't want to be overridden. For more info, see [manual correction](#preserving-manual-corrections).

## Step 2: Table Fixer

The Table Fixer step is really a collection of processes that build off of GrantEMP's basic views by automating frequent actions. It uses SQL to interface with the tables it recieves from the workspace folder, meaning the process is highly extensible. After processing the data, it exports all views to the `OutputTables` folder within the workspace folder.

Understanding how to use Table Fixer and its [Custom Views and Patches system](#custom-views-and-patches) to its fullest requires knowing the **order** these actions will happen in:

1. Create SQL Tables from CSV files.
2. [Process Custom Patches](#custom-views-and-patches)
3. [Copy Manual Corrections from Past Output](#preserving-manual-corrections)
4. [Recalculate Employability Scores](#employability-score-recalculation)
5. Create Views and Export them to CSV

#### Preserving Manual Corrections

GERA will preserve manual corrections made to the `CampusDataReport.csv` file, if the changes are made in columns of a past quarter. For example: you run First Quarter and then manually change some scores. Then you run Second Quarter and the changes will remain. However, if you instead decided to run First Quarter again then the changes would be overridden. When editing `CampusDataReport.csv`, ensure that Excel does not [change the file type or reformat columns](#avoiding-conflicts-with-excel).

##### The following columns will preserve manual corrections:
- First Name
- Last Name
- Grade Level
- Homeroom Teacher

##### The following columns are preserved, for each quarter
- Attendance Score
- Behavior Score
- Timeliness Score
- Professional Skills Score 1
- Professional Skills Score 2

The other columns in `CampusDataReport.csv` are the Employability Scores, which will be recalculated based on the manual changes to the above scores.

#### Employability Score Recalculation

After manual changes are copied, Employability Scores are recalculated. This takes the form of a weighted average for each quarter's Employability Score. The weights are defined under `gradeCategoryPercentages` in `config.json` in the [Configuration Folder](#configuration). Only the current quarter and past quarters will be recalculated. The Overall Employability Score is a simple average of each quarter's Employability Score, but this also will only average the Employability Scores in the current and past quarters.

Because of score recalculation, there is no need to do manual corrections for Employability Scores.

#### Custom Views and Patches

Creating custom SQL views and patches can help automate tasks that follow clear patterns. Views should be palced in the `views` folder in the [Configuration Folder](#configuration), and Patches should be placed in the `patches` folder. Both views and patches must be `.sql` files. If you're wondering about the exact feature set available for views and patches, GERA uses [SQLite](https://sqlite.org/).

**Patches** in practice have no limits, but generally should be used to overwrite values in the five base tables that come from GrantEMP. This is because CampusDataReport and the other output files are all views, not tables. Changing values in CampusDataReport, for example, actually requires overriding values in StudentEmployabilityData or StudentDemographic, depending on the column. `LabProfScoreCopy.sql`, which copies Professional Skills Score 1 to Professional Skills Score 2 if Professional Skills Score 2 is zero, is a patch that comes with GERA and serves as a good example of what patches are capable of. Changes made by a patch will be overridden if there is a manual correction at that value.

**Views** are more limited than patches, but GERA will handle the export of views to CSV files. Views should be in the form of a SELECT query which takes a collection of columns from multiple of the base tables. In addition, each column can use expressions and SQL methods to transform data. `CampusDataReportByTeacher.sql`, which comes with GERA, is a good example of the tools available for creating new views.

## Step 3: Generate PDFs

<img src="https://github.com/purplesprinklesdev/gera_docs/blob/main/resources/pdfExample.jpg" width="900">

GERA will create a Employability Report PDF for every student in CampusDataReport. There must exist a `CampusDataReport.csv` in the `OutputTables` folder and a `StudentDemographic.csv` in the workspace folder in order to generate PDFs. **PDF Generation takes a long time to complete. Do not close GERA or power off your computer until it is finished.** Each PDF will take about 1-2s, and the completion time scales linearly.

#### Chrome Requirement

An error will display if you attempt to generate PDFs without a valid install of Chrome or Chromium. If you are recieving this error despite Chrome being installed, it might be the case that GERA cannot find your Chrome executable because it isn't in the default location. Try uninstalling Chrome and reinstalling in the default location. Chrome is used to render an HTML file and export a PDF. The background Chrome process will not attempt to connect to the internet. For more information about GERA's use of Chrome, see the [Proof of FERPA Compliance](#ferpa-compliance).

### Avoiding Conflicts with OneDrive

GERA creates and deletes lots of temporary files in the workspace directory while it is running a task. OneDrive will attempt to sync the changes to all of these files, which will quickly overwhelm it. Unfortunately, there isn't much an application like GERA can do to remedy this, so the only solution is to **not select a folder that is backed up by OneDrive** when choosing your workspace.

If any of the following icons below show up next to your folder, then it is being backed up by OneDrive and should **not** be used with GERA.

<img src="https://github.com/purplesprinklesdev/gera_docs/blob/main/resources/onedriveicons.jpg" width="380">

If, for example, your desired workspace folder is in the `Documents` folder but is syncing with OneDrive, you may be at the path:

`C:\Users\USERNAME\OneDrive\Documents\` - Notice the "OneDrive" here

Whereas this path will probably not be set up to sync:

`C:\Users\USERNAME\Documents\` - Notice there's no "OneDrive"

Ultimately, your system will probably differ in some way, so you might just have to explore to find a place that OneDrive isn't syncing to. If all else fails, try creating a workspace at the root. (`C:\Temp\Workspace\` for example)

### Avoiding Conflicts with Excel

Excel can be used to edit the .csv files GERA exports. However, Excel will complain about saving any spreadsheet into CSV, giving the following warning:

<img src="https://github.com/purplesprinklesdev/gera_docs/blob/main/resources/excel2.jpg" width="600">

You should **NOT** follow Excel's advice here. If you save your manual changes to a .xlsx file, GERA will not be able to detect and preserve them. It's best to click "Don't Show Again" here and use Ctrl+S to save the spreadsheet to the same location when you're done.

<img src="https://github.com/purplesprinklesdev/gera_docs/blob/main/resources/excel1.jpg" width="600">

Once again, don't follow Excel's instructions if this pops up. Simply save to the same file location where GERA initially generated this CampusDataReport.

If Excel ever asks to convert numbers to scientific notation, or some other kind of value conversion, **DO NOT** let it. GERA is not able to read numbers in formats like scientific notation or percentages.

# Configuration

Configuration files can be accessed by clicking the "Open App Data Folder" button in GERA and opening the "config" folder. After the configuration is changed, you must restart GERA for the changes to take effect.

Default configuration files (the ones that were already there when GERA was installed) will be reacquired if they are not found. (deleted or renamed) However, their contents can be modified. Editing the default files is discouraged unless absolutely necessary, as it could break important functionality. If you want to undo your changes to a default file, simply delete the file and run GERA again.

### Logging

GERA will automatically log important events, and these logs are incredibly useful for troubleshooting. Logging is off by default, but can be enabled by setting the `logging` key in `config.json` to `true`. Verbose logs will include messages marked `DEBUG` and should only be enabled if the normal logs did not give enough information.

### PDFs and Charts

The template used for the PDFs is the `pdf_template.html` file. If you want to make changes, just make sure not to change anything that is between two `$` characters, as these are used to insert values into the PDF. The graphs follow styling values listed in the `config.json` file. Test your style changes appropriately to make sure elements don't fall off the page. If you accidentally break the PDFs because of style changes and want to go back to default, just delete `config.json` and run GERA.

`paperType` determines if the PDFs use US Letter or A4 (Metric) paper size. The default is Letter.

### Simple Graph Style Changes

All of the following changes can be made within the `config.json` file.

The colors of all charts besides overall employability fall under the `grayscaleColors` array. The last chart, corresponding to the employability scores, has its own color set under `employabilityColors`. Each color is an array of three integers between 0 and 256 corresponding to RGB values. To pick a new color, use a color picker website that displays RGB values which can then be copied into the config file. There must be exactly four colors in `grayscaleColors` and `employabilityColors`.

In the `chartTitles` object you can define the title of each chart. Entering a name that is very long could lead to the text getting cut off by the borders of the chart.

Changing the size or spacing parameters of the charts is not recommended, because it could easily lead to the charts becoming uncentered or falling off the page.

### Grade Category Distribution

`gradeCategoryPercentages` determines how Employability Scores are calculated. Since they are recalculated every time Table Fixer is ran, this setting effectively overrides the same setting in GrantEMP. The sum of all the values must equal 100.

### Resetting Configuration

To reset your configuration to default, simply delete the file you would like to be reset. Alternatively, if all config files should be reset, delete the entire config folder. Then, run GERA. The deleted files will now be recreated and reset to defaults.

### Views and Patches

The "views" and "patches" folder contain `.sql` files which give GERA additional functionality during the Table Fixer process. They are written in SQL, where patches should be a statement that is executed, and views should be a query that will be preceded by `CREATE VIEW view_name AS {}`, where your view will be inserted into the `{}`. GERA uses SQLite, and its complete feature set for in-memory databases should be available.

The default views in GERA will always be reacquired if they are deleted or renamed. They can be edited, however it is not recommended to edit `CampusDataReport.sql` because PDF generation relies on certain values being present.

There is one default patch, `LabCopyToProfSkills.sql`. This patch copies ProfessionalSkillsScore1 (Lab Score) to ProfessionalSkillsScore2 (Academic Score) if the latter is 0. While this can't be disabled or deleted, you can effectively disable this patch by replacing the file's content with `SELECT * FROM StudentEmployabilityData LIMIT 1`. Patches have access to the data in all five base tables, and can overwrite the values in them. If the goal you want is to change the value in a view, like CampusDataReport, you should use the patch to change its corresponding value in its constituent tables. (StudentEmployabilityData and StudentDemographic) To find what the constituent tables of a view are, look at the tables names following the `FROM` and `JOIN` statements.

# FERPA Compliance

GERA is fully FERPA compliant as-is because it never connects to the internet and thus will never transfer student data off of the user's device. The only exception is if the user specifies a workspace path that leads to a network drive. Even in that case, GERA will only utilize the network in order to read from and write to the specified drive. It will never, under any circumstance, "phone home" to a remote server. This can be independently verified by reading GERA's and its dependencies' code. GERA's code is property of Grant Career Center and all of its dependencies are open source, meaning all of the code GERA will be running can be reviewed by Grant. The full list of dependencies can be found in the GERA code repo under `src-tauri/Cargo.toml`

Chrome is one of a small set of browsers GERA can use to render PDFs. GERA interacts with Chrome through the headless_chrome crate, only using it for PDF rendering and never requesting an internet connection. Chrome is already used by staff at Grant, but just in case an open source alternative is required, Chromium can also be used for PDF rendering.

# For Developers
If you're just a user of GERA, you can safely ignore everything below this. If you're a developer, this may have some important info for you.

## Tech Stack
GERA uses Tauri to build a high performance Rust app to multiple platforms easily. The frontend uses SvelteKit. For more information about the structure of a Tauri app, consult the [Tauri website](https://v2.tauri.app/).
The app relies on several crates for core functionality. The full list is available in the `Cargo.toml` file. Here are some notable ones:

