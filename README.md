# Grant Employability Report Application (GERA)

<img src="https://github.com/purplesprinklesdev/gera_docs/blob/main/resources/logo.png" width="120" align="right">

A lightweight desktop application that generates Visual Employability Reports from student data
- Highly performant and compatible with all major operating systems (just in case)
- [Views and Patches system](#views-and-patches-system) for custom automation
- Simple design



## Table of Contents
- For Users
  - [Installation](#installation)
    - [Setting up MS Access Macro](#set-up-ms-access-macro)
  - [How to Use](#how-to-use)
    - [Avoiding Conflicts with OneDrive](#avoiding-conflicts-with-onedrive)
  - Configuration
    - Views and Patches system
  - FAQ
  - Troubleshooting
  - FERPA Compliance
- For Developers
  - Tech Stack
  - Cloning & Build Process
  - Code Documentation
    - Understanding the Configuration System
    - Table Fixer Process
    - PDF Generation Process


## Installation

Simply run the GERA installer exe and follow the wizard's instructions. Launch either through a desktop shortcut, or by searching in the Start menu.

## Set Up MS Access Macro

The modified Access Database which should include a button named "Export To CSV". If you see that button on the main menu, there's nothing else you have to do.

#### Create macro manually

Copy the following code and put it into Access

--TODO: explain--

```
Option Compare Database
Option Explicit

Public Function exportAsCSV()

On Error GoTo Err_ExportDatabaseObjects

Dim db As Database
Dim td As TableDef
Dim d As Document
Dim c As Container
Dim i As Integer
Dim sExportLocation As String

Set db = CurrentDb()

    sExportLocation = GetFolder()

    For Each td In db.TableDefs 'Tables
        If td.Name <> "tblStudentEmployabilityData" And td.Name <> "vwStudentClassAverage" And td.Name <> "StudentDemographic" And td.Name <> "StudentDiscipline" And td.Name <> "StudentDailyAbsence" Then GoTo Continue:
        If td.Name = "tblStudentEmployabilityData" Then td.Name = "StudentEmployabilityData"

        If Left(td.Name, 4) <> "MSys" Then
            DoCmd.TransferText acExportDelim, , td.Name, sExportLocation & "\" & td.Name & ".csv", True
        End If
Continue:
    Next td
Set db = Nothing
Set c = Nothing

MsgBox "All database objects have been exported as csv files to " & sExportLocation, vbInformation

Exit_ExportDatabaseObjects:
Exit Function

Err_ExportDatabaseObjects:
MsgBox Err.Number & " - " & Err.Description
Resume Exit_ExportDatabaseObjects


End Function

Function GetFolder() As String
    Dim fldr As Object
    Dim sItem As String
    Set fldr = Application.FileDialog(4)
    With fldr
        .Title = "Select a Folder"
        .AllowMultiSelect = False
        If .Show <> -1 Then GoTo NextCode
        sItem = .SelectedItems(1)
    End With
NextCode:
    GetFolder = sItem
    Set fldr = Nothing
End Function
```

## How to Use

GERA builds off of functionality already a part of the GrantEMP Access program. Start by importing data from the HCCA servers into Access. Then use the provided Export to CSV script to get the data ready for GERA.

### Exporting

Open Access and use the Export To CSV button, then select the folder you want to be your workspace. Make sure it [isn't set to sync to OneDrive](#avoiding-conflicts-with-onedrive).

Now you have the opportunity to look through the data in the five input tables and correct values if neccessary, but **it's recommended to wait until GERA has run Step 2: Table Fixer before doing manual corrections.**

### Step 1: Select Workspace and Quarter

Now it's time to open GERA. Select the same folder you exported the tables from Excel into, and pick a quarter. Choosing the right quarter for your data is very important. Below are all the features that take quarters into account:

- [Preserving past manual corrections](#preserving-manual-corrections). Manual corrections will be preserved if they are on a previous quarter's value. Selecting First Quarter will not preserve any corrections, selecting Third will preserve ones in Qtr1 and Qtr2 columns
- Employability Score recalculation. All previous quarters and the current quarter will have their employability scores recalculated. The `OverallEmployabilityScore` column will average the employability scores in previous quarters and the current quarter, but not future ones.
- PDFs. Impacts the naming of PDFs, the number of bars each graph has, and the number of rows the table has.

Even if your tables have more data filled in than your selected quarter would suggest, GERA will handle it and ignore the "future" data. This means you can redo past quarters even when newer data is present. The only problem this could cause is that manual corrections will be overridden. Create a backup of the `CampusDataReport.csv` file if there were manual corrections you don't want to be overridden. For more info, see [manual correction](#preserving-manual-corrections).

### Step 2: Table Fixer

The Table Fixer step is really a collection of processes that build off of GrantEMP's basic views by automating frequent actions. It uses SQL to interface with the tables it recieves from the workspace folder, meaning the process is highly extensible. Understanding how to use Table Fixer and its [Custom Views and Patches system](#custom-views-and-patches) to its fullest requires knowing the **order** these actions will happen in.

1. Create SQL Tables from CSV files.
2. [Process Custom Patches](#custom-views-and-patches)
3. [Copy Manual Corrections from Past Output](#preserving-manual-corrections)
4. Recalculate Employability Scores
5. Create Views and Export them to CSV

##### Preserving Manual Corrections

GERA will preserve manual corrections made to the `CampusDataReport.csv` file, if the changes are made in columns of a past quarter. For example, you run First Quarter and then manually change some scores. Then you run Second Quarter and the changes will remain. However, if you decided to run First Quarter then the changes would be overridden.

##### Employability Score Recalculation

##### Custom Views and Patches

### Avoiding Conflicts with OneDrive

GERA creates and deletes lots of temporary files in the workspace directory while it is running a task. OneDrive will attempt to sync the changes to all of these files, which will quickly overwhelm it. Unfortunately, there isn't much an application like GERA can do to remedy this, so the only solution is to **not select a folder that is backed up by OneDrive** when choosing your workspace.

If any of the following icons below show up next to your folder, then it is being backed up by OneDrive and should **not** be used with GERA.

<img src="https://github.com/purplesprinklesdev/gera_docs/blob/main/resources/onedriveicons.jpg" width="380">

If, for example, your desired workspace folder is in the `Documents` folder but is syncing with OneDrive, you may be at the path:

`C:\Users\USERNAME\OneDrive\Documents\` - Notice the "OneDrive" here

Whereas this path will probably not be set up to sync:

`C:\Users\USERNAME\Documents\` - Notice there's no "OneDrive"

Ultimately, your system will probably differ in some way, so you might just have to explore to find a place that OneDrive isn't syncing to. If all else fails, try creating a workspace at the root. (`C:\Workspace\` for example)
