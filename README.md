# Grant Employability Report Application (GERA)

<img src="https://github.com/purplesprinklesdev/gera_docs/blob/main/resources/logo.png" width="250" align="right">

A lightweight desktop application that generates Visual Employability Reports from student data
* Highly performant and compatible with all major operating systems (just in case)
* Powerful Views and Patches system for custom automation
* Simple design



## Table of Contents
* For Users
  * [Installation](#installation)
    * [Setting up MS Access Macro](#set-up-ms-access-macro)
  * How to Use
  * Configuration
    * Custom Views and Patches system
  * FAQ
  * Troubleshooting
  * FERPA Compliance
* For Developers
  * Tech Stack
  * Cloning & Build Process
  * Code Documentation
    * Understanding the Configuration System
    * Table Fixer Process
    * PDF Generation Process


## Installation
**TODO: add stuff here**

## Set Up MS Access Macro

Copy the following code and put it into Access
**TODO: explain**

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
