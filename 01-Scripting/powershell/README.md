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
Get-Process | Select-Object Name, Id, CPU |
  Sort-Object CPU -Descending | Format-Table -AutoSize

# Property expansion
$procs = Get-Process
$procs.Name
```

---

## Filesystem and Registry

```powershell
# File operations
Get-ChildItem -Path C:\\Users -Recurse
Copy-Item source.txt destination.txt
Move-Item file.txt newlocation\\
Remove-Item file.txt
New-Item -ItemType Directory -Path C:\\NewFolder

# Registry
# Read registry value
Get-ItemProperty -Path 'HKLM:\\Software\\Microsoft\\Windows\\CurrentVersion' -Name ProgramFilesDir

# Set registry value
Set-ItemProperty -Path 'HKCU:\\Software\\MyApp' -Name Version -Value '1.0'
```

---

## Essential Cmdlets

| Cmdlet | Description | Common Parameters |
| :--- | :--- | :--- |
| `Get-Command` | List available commands | `-Name`, `-Module` |
| `Get-Help` | Display help | `-Examples`, `-Online` |
| `Get-Member` | Object properties/methods | N/A |
| `Where-Object` | Filter objects | `{$_.Property -eq value}` |
| `Select-Object` | Select properties | `-First`, `-Last`, `-Property` |
| `ForEach-Object` | Loop through objects | `{$_ \| Command}` |
| `Measure-Object` | Calculate stats | `-Sum`, `-Average`, `-Maximum` |

---

## Conditionals and Loops

```powershell
# If statement
if ($condition) {
    # code
} elseif ($condition2) {
    # code
} else {
    # code
}

# Switch
switch ($variable) {
    value1 { # code }
    value2 { # code }
    default { # code }
}

# ForEach loop
foreach ($item in $collection) {
    Write-Host $item
}

# While loop
while ($condition) {
    # code
}
```

---

## Functions and Modules

```powershell
# Simple function
function Get-Greeting {
    param(
        [Parameter(Mandatory=$true)]
        [string]$Name
    )
    return "Hello, $Name!"
}

# Advanced function
function Invoke-CustomScript {
    [CmdletBinding()]
    param(
        [Parameter(ValueFromPipeline=$true)]
        [string[]]$InputObject,
        
        [switch]$Verbose
    )
    
    begin {
        Write-Verbose "Starting..."
    }
    
    process {
        foreach ($item in $InputObject) {
            Write-Output "Processing: $item"
        }
    }
    
    end {
        Write-Verbose "Complete"
    }
}

# Import module
Import-Module ModuleName
```

---

## Error Handling

```powershell
# Try-Catch-Finally
try {
    # Code that might fail
    Get-Item "NonExistentFile.txt" -ErrorAction Stop
} catch {
    Write-Error "An error occurred: $_"
    Write-Host $_.Exception.Message
} finally {
    # Always runs
    Write-Host "Cleanup complete"
}

# Error variables
$Error[0]              # Most recent error
$Error.Clear()         # Clear error buffer
```

---

## Working with JSON and API

```powershell
# Parse JSON
$json = Get-Content config.json | ConvertFrom-Json

# Create JSON
$data = @{
    Name = "Test"
    Value = 123
}
$data | ConvertTo-Json | Out-File output.json

# REST API
$response = Invoke-RestMethod -Uri 'https://api.example.com/data' -Method Get
$response = Invoke-WebRequest -Uri 'https://example.com' -Method Post -Body $body
```

---

## Remoting

```powershell
# Execute command on remote computer
Invoke-Command -ComputerName Server01 -ScriptBlock { Get-Process }

# Start session
$session = New-PSSession -ComputerName Server01
Invoke-Command -Session $session -ScriptBlock { Get-Service }
Remove-PSSession $session

# Copy files to/from remote
Copy-Item C:\\Local\\file.txt -Destination C:\\Remote\\ -ToSession $session
```

---

## Best Practices Template

```powershell
#Requires -Version 5.1
<#
.SYNOPSIS
    Brief description
.DESCRIPTION
    Detailed description
.PARAMETER Name
    Parameter description
.EXAMPLE
    PS> .\\Script.ps1 -Name "Test"
.NOTES
    Author: Your Name
    Date: 2024-01-01
#>

[CmdletBinding()]
param(
    [Parameter(Mandatory=$true,
               ValueFromPipeline=$true,
               HelpMessage="Enter a name")]
    [ValidateNotNullOrEmpty()]
    [string]$Name,
    
    [Parameter()]
    [switch]$Force
)

begin {
    $ErrorActionPreference = 'Stop'
    Write-Verbose "Starting script execution"
}

process {
    try {
        # Main logic here
        Write-Output "Processing: $Name"
    } catch {
        Write-Error "Failed to process: $_"
        throw
    }
}

end {
    Write-Verbose "Script execution completed"
}
```

This cheatsheet covers essential PowerShell commands and patterns for
DevOps workflows.
