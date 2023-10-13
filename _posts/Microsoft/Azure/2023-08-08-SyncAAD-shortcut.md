---
title: Sync AAD Button (Interactive)
author: Jacky
date: 2023-08-08 11:33:00 +0800
categories: [Microsoft, Entra]
tags: [Microsoft, Entra]
pin: false
math: false
mermaid: false
---

# Sync AAD (Interactive)

Shortcut:

target: C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -noexit -ExecutionPolicy Bypass -File syncAD.ps1 -NoNewWindow

Start in: C:\custom\Utility

Icon: %ProgramFiles%\Microsoft Azure AD Sync\UIShell\miisclient.exe


Script: 

```ps
Write-Host "2023 JLangisch.de"
Add-Type -AssemblyName System.Windows.Forms

$form = New-Object System.Windows.Forms.Form
$form.Text = "Sync Type Selection"
$form.Width = 600
$form.Height = 200

$label = New-Object System.Windows.Forms.Label
$label.Text = "Please select a sync type:"
$label.AutoSize = $true
$label.Location = New-Object System.Drawing.Size(10,10)
$form.Controls.Add($label)

$deltaButton = New-Object System.Windows.Forms.RadioButton
$deltaButton.Text = "Delta"
$deltaButton.AutoSize = $true
$deltaButton.Location = New-Object System.Drawing.Size(10,30)
$form.Controls.Add($deltaButton)

$initialButton = New-Object System.Windows.Forms.RadioButton
$initialButton.Text = "Initial"
$initialButton.AutoSize = $true
$initialButton.Location = New-Object System.Drawing.Size(10,50)
$form.Controls.Add($initialButton)

$warning = New-Object System.Windows.Forms.Label
$warning.Text = "WARNING: Initial sync will replace all data in the AAD with data from the AD source."
$warning.AutoSize = $true
$warning.Location = New-Object System.Drawing.Size(10,80)
$warning.ForeColor = "Red"
$form.Controls.Add($warning)

$okButton = New-Object System.Windows.Forms.Button
$okButton.Text = "OK"
$okButton.AutoSize = $true
$okButton.Location = New-Object System.Drawing.Size(10,120)
$okButton.Add_Click({
    if ($initialButton.Checked) {
        $confirmation = [System.Windows.Forms.MessageBox]::Show("Are you sure you want to run an initial sync?", "Confirmation", [System.Windows.Forms.MessageBoxButtons]::YesNo)
        if ($confirmation -eq "Yes") {
            try {
                Start-ADSyncSyncCycle -PolicyType Initial
                Write-Host "Success with the Initial sync. Cancel relates to the form."
            } catch {
                Write-Host "Error: $($_.Exception.Message)"
            }
        }
    } elseif ($deltaButton.Checked) {
        try {
            Start-ADSyncSyncCycle -PolicyType Delta
            Write-Host "Success with the Delta sync. Cancel relates to the form."
        } catch {
            Write-Host "Error: $($_.Exception.Message)"
        }
    } else {
        Write-Host "Please select a sync type."
    }
    $form.Close()
})
$form.Controls.Add($okButton)

$form.ShowDialog()
[Environment]::Exit(1)

````