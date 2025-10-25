# 💠 PowerShell Cheatsheet

[← Back to Main ToC](../../README.md)

## Core Concepts

### Execution Policy
```powershell
# Get current execution policy
Get-ExecutionPolicy
# Set execution policy (run as Administrator)
Set-ExecutionPolicy RemoteSigned -Scope CurrentUser
```

### Variables and Types
```powershell
# Variables
$Name = "World"
$Number = 42
$Array = 1,2,3
$Hash = @{ Key = 'Value'; Foo = 'Bar' }

# Types
[System.Int32]$i = 10
[datetime]::Now
[Guid]::NewGuid()
```

### Pipeline and Objects
```powershell
# Everything is an object
Get-Process | Select-Object Name, Id, CPU | Sort-Object CPU -Descending | Format-Table -AutoSize

# Property expansion
(Get-Service | Where-Object {$_.Status -eq 'Running'}).Count

# Export/Import
Get-Service | Export-Csv services.csv -NoTypeInformation
Import-Csv services.csv | Where-Object {$_.Status -eq 'Running'}
```

## Filesystem and Registry
```powershell
# Files
Get-ChildItem -Path C:\ -Recurse -Filter *.log -ErrorAction SilentlyContinue
New-Item -Path . -Name 'file.txt' -ItemType File
Set-Content -Path file.txt -Value 'Hello'
Add-Content -Path file.txt -Value 'World'
Get-Content -Path file.txt

# Directories
New-Item -ItemType Directory -Path .\logs
Remove-Item -Recurse -Force .\logs

# Registry
Get-ItemProperty 'HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion' | Select-Object -First 1
```

## Networking
```powershell
# HTTP requests
Invoke-WebRequest -Uri https://example.com -OutFile index.html
Invoke-RestMethod -Uri https://api.github.com/repos -Headers @{ 'User-Agent'='ps' }

# DNS and TCP
Resolve-DnsName google.com
Test-NetConnection -ComputerName github.com -Port 443
```

## Processes and Services
```powershell
# Processes
Get-Process | Sort-Object CPU -Descending | Select-Object -First 5 Name,CPU,Id
Stop-Process -Id 1234 -Force

# Services
Get-Service | Where-Object {$_.Status -eq 'Running'}
Restart-Service -Name 'Spooler'
Set-Service -Name 'Spooler' -StartupType Automatic
```

## Remoting and Sessions
```powershell
# Enable remoting (as admin)
Enable-PSRemoting -Force

# Create session
$session = New-PSSession -ComputerName server01
Invoke-Command -Session $session -ScriptBlock { Get-Process }
Remove-PSSession $session
```

## Modules and Packages
```powershell
# Modules
Get-Module -ListAvailable
Install-Module -Name Pester -Scope CurrentUser -Force
Import-Module Pester

# Packages (winget/choco)
winget install Git.Git
choco install 7zip -y
```

## JSON/YAML and Files
```powershell
# JSON
$json = Get-Content .\config.json -Raw | ConvertFrom-Json
$json.Property = 'NewValue'
$json | ConvertTo-Json -Depth 10 | Set-Content .\config.json

# YAML via PSYaml module
Install-Module -Name powershell-yaml -Scope CurrentUser -Force
Import-Module powershell-yaml
$yaml = Get-Content .\config.yaml -Raw | ConvertFrom-Yaml
$yaml.property = 'value'
$yaml | ConvertTo-Yaml | Set-Content .\config.yaml
```

## Error Handling & Logging
```powershell
# Try/Catch/Finally
try {
  1/0
} catch [DivideByZeroException] {
  Write-Warning "Divide by zero"
} catch {
  Write-Error $_.Exception.Message
} finally {
  Write-Host "Cleanup"
}

# Transcript logging
Start-Transcript -Path .\session.log
Stop-Transcript
```

## Scripting Patterns
```powershell
param(
  [Parameter(Mandatory=$true)]
  [string]$Name,
  [ValidateSet('dev','staging','prod')]
  [string]$Environment = 'dev',
  [switch]$Verbose
)

function Write-Info($Message) {
  Write-Host "[INFO] $Message" -ForegroundColor Cyan
}

Write-Info "Deploying to $Environment"
if ($Verbose) { $PSStyle.OutputRendering = 'Ansi' }
```

## Automation Examples

### Invoke external commands
```powershell
# Git
& git --version
& git status

# Docker
& docker ps
& docker run --rm -it alpine:3.19 echo hello
```

### Windows Task Scheduler
```powershell
$action = New-ScheduledTaskAction -Execute 'powershell.exe' -Argument '-File C:\scripts\job.ps1'
$trigger = New-ScheduledTaskTrigger -Daily -At 3am
Register-ScheduledTask -Action $action -Trigger $trigger -TaskName 'NightlyJob' -Description 'Nightly maintenance job'
```

## Best Practices & Gotchas

### ✅ Do's
- Use cmdlets over legacy EXEs when possible
- Prefer `Where-Object`/`ForEach-Object` over piping to `where`/`foreach`
- Use `-ErrorAction` and `-ErrorVariable` for robust error handling
- Use `Try/Catch` around risky operations
- Sign scripts in enterprise environments

### ❌ Don'ts
- Don't disable execution policy globally
- Don't rely on string parsing when objects are available
- Don't swallow errors silently

### 🔧 Common Gotchas
- PowerShell versions differ between Windows PowerShell 5.1 and PowerShell 7+
- JSON conversion needs `-Depth` for nested structures
- Remoting requires firewall and service configuration

### 📝 Script Template
```powershell
<#
.SYNOPSIS
  Example advanced function template
.DESCRIPTION
  Demonstrates parameters, pipeline, and error handling
#>
[CmdletBinding()]
param(
  [Parameter(Mandatory, ValueFromPipeline)]
  [string[]]$InputObject,
  [switch]$WhatIf
)

begin { $count = 0 }
process {
  foreach ($item in $InputObject) {
    $count++
    Write-Verbose "Processing $item"
  }
}
end {
  Write-Output "Processed $count items"
}
```

This cheatsheet covers essential PowerShell commands, patterns, and automation recipes for DevOps.
