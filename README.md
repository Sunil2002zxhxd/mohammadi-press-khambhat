' Paste this into a Module in Excel VBA editor.
' Use Developer -> Visual Basic -> Insert -> Module, then paste.
' Save workbook as .xlsm (Macro-enabled).

Function URLEncode(str As String) As String
    Dim i As Long, ch As Integer, out As String
    For i = 1 To Len(str)
        ch = AscW(Mid$(str, i, 1))
        Select Case ch
            Case 48 To 57, 65 To 90, 97 To 122
                out = out & ChrW(ch)
            Case 32
                out = out & "%20"
            Case Else
                out = out & "%" & Hex(ch)
        End Select
    Next i
    URLEncode = out
End Function

Sub SendEstimateToWhatsApp()
    Dim ws As Worksheet
    Set ws = ThisWorkbook.ActiveSheet

    Dim r As Long
    r = ActiveCell.Row  ' message will be taken from the active row

    Dim phone As String
    phone = Trim(ws.Cells(r, "B").Value) ' PhoneNumber column (B)

    If phone = "" Then
        MsgBox "Please enter phone number (digits only) in column B.", vbExclamation
        Exit Sub
    End If

    Dim header As String
    header = ws.Cells(r, "K").Value ' Company Header / Sign (column K)

    Dim details As String
    details = ws.Cells(r, "C").Value ' Details / Type of Work (column C)

    Dim qty As String
    qty = ws.Cells(r, "D").Value

    Dim totalAmt As String
    totalAmt = ws.Cells(r, "E").Value

    Dim adv As String
    adv = ws.Cells(r, "F").Value

    Dim outstanding As String
    outstanding = ws.Cells(r, "G").Value

    Dim proc As String
    proc = ws.Cells(r, "H").Value

    Dim days As String
    days = ws.Cells(r, "I").Value

    Dim delivery As String
    delivery = ws.Cells(r, "J").Value

    ' Build message (Gujarati) - edit as needed
    Dim msg As String
    msg = header & vbCrLf & vbCrLf
    msg = msg & "*ESTIMATE*" & vbCrLf & vbCrLf
    msg = msg & "Particulars: " & details & vbCrLf & vbCrLf
    msg = msg & "Quantity: " & qty & vbCrLf
    msg = msg & "Total Amount: ₹ " & totalAmt & vbCrLf
    msg = msg & "Advance Paid: ₹ " & adv & vbCrLf
    msg = msg & "Outstanding: ₹ " & outstanding & vbCrLf
    msg = msg & "PROCESS: " & proc & "  WORKING DAY/S: " & days & vbCrLf
    msg = msg & "Delivery Time: " & delivery & vbCrLf & vbCrLf
    msg = msg & "મોહંમદી પ્રિન્ટીંગ પ્રેસ (For any queries call: 9825547625)"

    Dim url As String
    url = "https://web.whatsapp.com/send?phone=91" & phone & "&text=" & URLEncode(msg)

    ' Open default browser with WhatsApp Web prefilled message
    ThisWorkbook.FollowHyperlink Address:=url, NewWindow:=True
End Sub
