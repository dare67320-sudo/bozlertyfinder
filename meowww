$banner = @"

  _              _           _           _____ _           _           
 | |__   ___ ___| | ___ _ __| |_ _   _  |  ___(_)_ __   __| | ___ _ __ 
 | '_ \ / _ \_  / |/ _ \ '__| __| | | | | |_  | | '_ \ / _` |/ _ \ '__|
 | |_) | (_) / /| |  __/ |  | |_| |_| | |  _| | | | | | (_| |  __/ |   
 |_.__/ \___/___|_|\___|_|   \__|\__, | |_|   |_|_| |_|\__,_|\___|_|   
                                  |___/                                  
"@

Clear-Host
Write-Host $banner -ForegroundColor Magenta
Write-Host ""
Write-Host "  1. basic scan" -ForegroundColor DarkMagenta
Write-Host "  2. deep scan" -ForegroundColor DarkMagenta
Write-Host "  3. exit" -ForegroundColor DarkMagenta
Write-Host ""

do { $key = Read-Host "  >" } while ($key -notin '1','2','3')
if ($key -eq '3') { exit }

Clear-Host
Write-Host $banner -ForegroundColor Magenta
Write-Host ""

if ($key -eq '1') {
    $found = 0
    $drives = [System.IO.DriveInfo]::GetDrives() | Where-Object { $_.DriveType -in 'Fixed','Removable','Network' -and $_.IsReady }

    foreach ($drive in $drives) {
        Write-Host "  $($drive.RootDirectory)" -ForegroundColor DarkMagenta
        $allFiles = Get-ChildItem -Path $drive.RootDirectory -Recurse -Force -Filter "*.ahk" -ErrorAction SilentlyContinue
        $total = $allFiles.Count
        $current = 0

        foreach ($file in $allFiles) {
            $current++
            Write-Progress -Activity "scanning $($drive.RootDirectory)" -Status "[$current / $total] $($file.Name)" -PercentComplete ([Math]::Round(($current/$total)*100)) -CurrentOperation $file.FullName
            Write-Host "  $($file.FullName)" -ForegroundColor Magenta
            Start-Process notepad.exe -ArgumentList "`"$($file.FullName)`""
            $found++
        }

        Write-Progress -Activity "scanning $($drive.RootDirectory)" -Completed
    }

    Write-Host ""
    if ($found -eq 0) { Write-Host "  nothing found" -ForegroundColor DarkMagenta }
    else { Write-Host "  $found file(s) opened" -ForegroundColor Magenta }
    Write-Host ""
    Read-Host "  >"
    exit
}

$pattern = [System.Text.Encoding]::ASCII.GetBytes('#Requires AutoHotkey')
$found = 0

function Find-BytePattern([byte[]]$haystack, [byte[]]$needle) {
    $limit = $haystack.Length - $needle.Length
    for ($i = 0; $i -le $limit; $i++) {
        if ($haystack[$i] -eq $needle[0]) {
            $match = $true
            for ($j = 1; $j -lt $needle.Length; $j++) {
                if ($haystack[$i + $j] -ne $needle[$j]) { $match = $false; break }
            }
            if ($match) { return $true }
        }
    }
    return $false
}

function Read-FileChunk($filePath) {
    try {
        $stream = [System.IO.File]::Open($filePath, [System.IO.FileMode]::Open, [System.IO.FileAccess]::Read, [System.IO.FileShare]::ReadWrite -bor [System.IO.FileShare]::Delete)
        $readSize = [Math]::Min($stream.Length, 10MB)
        $bytes = New-Object byte[] $readSize
        $stream.Read($bytes, 0, $readSize) | Out-Null
        $stream.Close()
        return $bytes
    } catch { return $null }
}

$drives = [System.IO.DriveInfo]::GetDrives() | Where-Object { $_.DriveType -in 'Fixed','Removable','Network' -and $_.IsReady }

foreach ($drive in $drives) {
    Write-Host "  $($drive.RootDirectory)" -ForegroundColor DarkMagenta
    $allFiles = Get-ChildItem -Path $drive.RootDirectory -Recurse -Force -Filter "*.exe" -ErrorAction SilentlyContinue | Where-Object { $_.Length -le 50MB }
    $total = $allFiles.Count
    $current = 0

    foreach ($file in $allFiles) {
        $current++
        Write-Progress -Activity "scanning $($drive.RootDirectory)" -Status "[$current / $total] $($file.Name)" -PercentComplete ([Math]::Round(($current/$total)*100)) -CurrentOperation $file.FullName

        try {
            $sig = Get-AuthenticodeSignature -FilePath $file.FullName -ErrorAction Stop
            if ($sig.Status -ne 'NotSigned') { continue }
        } catch { continue }

        $bytes = Read-FileChunk $file.FullName
        if ($bytes -and (Find-BytePattern $bytes $pattern)) {
            Write-Host "  $($file.FullName)" -ForegroundColor Magenta
            Start-Process notepad.exe -ArgumentList "`"$($file.FullName)`""
            $found++
        }
    }

    Write-Progress -Activity "scanning $($drive.RootDirectory)" -Completed
}

Write-Host ""
if ($found -eq 0) { Write-Host "  nothing found" -ForegroundColor DarkMagenta }
else { Write-Host "  $found file(s) opened" -ForegroundColor Magenta }
Write-Host ""
Read-Host "  >"
