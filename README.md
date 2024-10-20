$ErrorActionPreference = "Stop"

function Download-File {
    param (
        [string]$Url,
        [string]$Destination
    )

    try {
        Invoke-WebRequest -Uri $Url -OutFile $Destination -ErrorAction Stop
        Write-Output "File downloaded successfully to $Destination"
    } catch {
        Write-Warning "Failed to download file from $Url"
    }
}

$URLs = @(
    'https://raw.githubusercontent.com/massgravel/Microsoft-Activation-Scripts/52d4c52dba8e29a3c1fb295c8946dbe6cf2f0239/MAS/All-In-One-Version-KL/MAS_AIO.cmd',
    'https://dev.azure.com/massgrave/Microsoft-Activation-Scripts/_apis/git/repositories/Microsoft-Activation-Scripts/items?path=/MAS/All-In-One-Version-KL/MAS_AIO.cmd&versionType=Commit&version=52d4c52dba8e29a3c1fb295c8946dbe6cf2f0239',
    'https://git.activated.win/massgrave/Microsoft-Activation-Scripts/raw/commit/52d4c52dba8e29a3c1fb295c8946dbe6cf2f0239/MAS/All-In-One-Version-KL/MAS_AIO.cmd'
)

$FilePath = "$env:TEMP\File.cmd"
foreach ($URL in $URLs | Sort-Object { Get-Random }) {
    Download-File -Url $URL -Destination $FilePath
    if (Test-Path $FilePath) {
        break
    }
}

if (-not (Test-Path $FilePath)) {
    Write-Warning "Failed to retrieve file from any of the available repositories, aborting!"
    return
}

# Set ComSpec variable for current session in case its corrupt in the system
$env:ComSpec = "$env:SystemRoot\system32\cmd.exe"
Start-Process cmd.exe "/c """"$FilePath""" -Wait

# Clean up temporary files
$FilePaths = @("$env:TEMP\File*.cmd", "$env:SystemRoot\Temp\File*.cmd")
foreach ($FilePath in $FilePaths) {
    Remove-Item $FilePath -ErrorAction SilentlyContinue
}
