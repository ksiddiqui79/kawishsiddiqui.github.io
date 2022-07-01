title: VBA MS Excel ConcatIF Function
description: VBA MS Excel ConcatIF Function
keywords: excel, word, ms, utility, vacuum, analyze, data lake, datalake, formation, case study, shell, bash, sh, split, size, console, aws, files, tips, cheat, shell command, bash command, script, shell script, sql, teradata, redshift, vba, database, data warehouse, etl, custom, big data

#VBA MS Excel ConcatIF Function
Since long time I have been thinking to have a function in Excel like SUMIF. Initially I thought Office 2010 will bring this feature/function.
But now I have solution function here for all.

#VBA Code

```VBA
Public Function ConcatIf(ByRef FindInRange As Range, ByRef FindText As String, ByRef StringRange As Range, Optional ByRef strDelimiter As String)
If IsNull(strDelimiter) Then strDelimiter = ","
Dim l As Integer
Dim result As String

For l = 1 To FindInRange.Count
   If FindInRange(l, 1) = FindText Then
      If IsNull(result) Or Trim(result) = "" Then
         result = StringRange(l, 1)
      Else
         If StringRange(l, 1)  "" Then result = result & strDelimiter & StringRange(l, 1)
      End If
   End If
Next
ConcatIf2 = CStr(result)
End Function
```