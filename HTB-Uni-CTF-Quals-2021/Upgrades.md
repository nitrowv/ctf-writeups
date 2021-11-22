# Upgrades

**Writeup by:** nitrowv   
**Category:** Reversing   
**Difficulty:** Easy

We're given a macro-enabled Powerpoint presentation with the file name `Upgrades.pptm`. 
As expected, when we open this file in LibreOffice Impress, it finds the macro and disables its execution by default.

Opening the macro in the Edit Macro feature of Impress we see:
```vb
Rem Attribute VBA_ModuleType=VBAModule
Sub Module1
Rem Private Function q(g) As String
Rem q = ""
Rem For Each I In g
Rem q = q & Chr((I * 59 - 54) And 255)
Rem Next I
Rem End Function
Rem Sub OnSlideShowPageChange()
Rem j = Array(q(Array(245, 46, 46, 162, 245, 162, 254, 250, 33, 185, 33)), _
Rem q(Array(215, 120, 237, 94, 33, 162, 241, 107, 33, 20, 81, 198, 162, 219, 159, 172, 94, 33, 172, 94)), _
Rem q(Array(245, 46, 46, 162, 89, 159, 120, 33, 162, 254, 63, 206, 63)), _
Rem q(Array(89, 159, 120, 33, 162, 11, 198, 237, 46, 33, 107)), _
Rem q(Array(232, 33, 94, 94, 33, 120, 162, 254, 237, 94, 198, 33)))
Rem g = Int((UBound(j) + 1) * Rnd)
Rem With ActivePresentation.Slides(2).Shapes(2).TextFrame
Rem .TextRange.Text = j(g)
Rem End With
Rem If StrComp(Environ$(q(Array(81, 107, 33, 120, 172, 85, 185, 33))), q(Array(154, 254, 232, 3, 171, 171, 16, 29, 111, 228, 232, 245, 111, 89, 158, 219, 24, 210, 111, 171, 172, 219, 210, 46, 197, 76, 167, 233)), vbBinaryCompare) = 0 Then
Rem VBA.CreateObject(q(Array(215, 11, 59, 120, 237, 146, 94, 236, 11, 250, 33, 198, 198))).Run (q(Array(59, 185, 46, 236, 33, 42, 33, 162, 223, 219, 162, 107, 250, 81, 94, 46, 159, 55, 172, 162, 223, 11)))
Rem End If
Rem End Sub
Rem 
Rem 
Rem 
End Sub
```

The first thing that grabs our attention is the function `q(g)`.
```vb
Rem Private Function q(g) As String
Rem q = ""
Rem For Each I In g
Rem q = q & Chr((I * 59 - 54) And 255)
Rem Next I
Rem End Function
```

This function iterates through the argument, appending `Chr((I * 59 - 54) And 255)` to the string `q`. This takes the provided value, multiplies it by 59, subtracts 54, and uses a bitwise AND to compare it to 255.

Using the first character in the first instance of `q()`, we have `Chr((245 * 59 - 54) And 255)`. Simplyfing this, we get `Chr(14401 And 255)`, which is `Chr(65)`. Looking at our ASCII table, we see that `65 == 'A'`.

With that, I opted to use Python to run through the instances of `q()`. I took the values in each of the arrays and put them in their own Python list.

```python
list1 = [245,46,46,162,254,250,33,185,33]
list2 = [215,120,237,94,33,162,241,107,33,20,81,198,162,219,159,172,94,33,172,94]
list3 = [245, 46, 46, 162, 89, 159, 120, 33, 162, 254, 63, 206, 63]
list4 = [154, 254, 232, 3, 171, 171, 16, 29, 111, 228, 232, 245, 111, 89, 158, 219, 24, 210, 111, 171, 172, 219, 210, 46, 197, 76, 167, 233]
asciistr = ""
asciistr2 = ""
asciistr3 = ""
asciistr4 = ""
for i in list1:
    numAsc = chr((i*59-54) & 255)
    asciistr += numAsc

for i in list2:
    numAsc2 = chr((i*59-54) & 255)
    asciistr2 += numAsc2

for i in list3:
    numAsc3 = chr((i*59-54) & 255)
    asciistr3 += numAsc3

for i in list4:
    numAsc4 = chr((i*59-54) & 255)
    asciistr4 += numAsc4

print(asciistr)
print(asciistr2)
print(asciistr3)
print(asciistr4)
```

Running this script we get the following output:

```
Add Theme
Write Useful Content        
Add More TODO
HTB{33zy_VBA_M4CR0_3nC0d1NG}
```

Which contains the flag: 

`HTB{33zy_VBA_M4CR0_3nC0d1NG}`