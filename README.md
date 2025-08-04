# Grant Employability Report Application (GERA)

<img src="https://github.com/purplesprinklesdev/gera_docs/blob/main/resources/logo.png" width="120" align="right">

A lightweight desktop application that generates Visual Employability Reports from student data
- Highly performant and compatible with all major operating systems (just in case)
- [Powerful Views and Patches system](#views-and-patches-system) for custom automation
- Simple design



## Table of Contents
- For Users
  - [Installation](#installation)
    - [Setting up MS Access Macro](#set-up-ms-access-macro)
  - [How to Use](#how-to-use)
    - [Avoiding Conflicts with OneDrive](#avoiding-conflicts-with-onedrive)
  - Configuration
    - Custom Views and Patches system
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

Simply run the GERA installer exe and follow the wizard's instructions

## Set Up MS Access Macro

#### Import Method

Use the modified Access Database which should include a button named "Export To CSV"

#### Manual Method

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

Open Access and use the Export To CSV button, selecting the folder to be your workspace. Make sure it [isn't set to sync to OneDrive](#avoiding-conflicts-with-onedrive)

### Avoiding Conflicts with OneDrive

GERA creates and deletes lots of temporary files in the workspace directory while it is running a task. OneDrive will attempt to sync the changes to all of these files, which will quickly overwhelm it. Unfortunately, there isn't much an application like GERA can do to remedy this, so the only solution is to **not select a folder that is backed up by OneDrive** when choosing your workspace.

If any of the following icons below show up next to your folder, then it is being backed up by OneDrive and should **not** be used with GERA.

<img src="https://github.com/purplesprinklesdev/gera_docs/blob/main/resources/onedriveicons.jpg" width="120">

If, for example, your desired workspace folder is in the `Documents` folder but is syncing with OneDrive, you may be at the path:

`C:\Users\USERNAME\OneDrive\Documents\` - Notice the "OneDrive" here

Whereas this path will probably not be set up to sync:

`C:\Users\USERNAME\Documents\` - Notice there's no "OneDrive"

Ultimately, your system will probably differ in some way, so you might just have to explore to find a place that OneDrive isn't syncing to. If all else fails, try creating a workspace at the root. (`C:\Workspace\` for example)
