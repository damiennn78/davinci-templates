---
description: Package a DaVinci Resolve Fusion template — zips the Edit/ folder and renames it <FolderName>.drfx
---

Package the DaVinci Resolve Fusion template whose folder name is: $ARGUMENTS

Steps to execute (PowerShell, working directory is the Template root):

1. Validate that the folder `$ARGUMENTS` exists directly under the current working directory. If not, try to find the closest folder if you have doubts then stop and report the error.
2. Validate that `$ARGUMENTS\Edit` exists. If not, stop and report the error.
3. Delete `$ARGUMENTS\$ARGUMENTS.drfx` and `$ARGUMENTS\Edit.drfx` if they already exist (clean rebuild).
4. Crée le ZIP manuellement avec `ZipArchive` pour garantir les forward slashes et les entrées de dossiers explicites (requis par DaVinci Resolve) :
   ```powershell
   Add-Type -AssemblyName System.IO.Compression.FileSystem
   Add-Type -AssemblyName System.IO.Compression

   $templateName = "$ARGUMENTS"
   $editPath     = (Resolve-Path (Join-Path $templateName "Edit")).Path
   $editParent   = Split-Path $editPath -Parent
   $drfxPath     = Join-Path (Resolve-Path $templateName).Path "$templateName.drfx"
   $zipTemp      = $drfxPath -replace '\.drfx$', '.zip'

   if (Test-Path $drfxPath) { Remove-Item $drfxPath -Force }
   if (Test-Path $zipTemp)  { Remove-Item $zipTemp  -Force }
   $oldDrfx = Join-Path (Resolve-Path $templateName).Path "Edit.drfx"
   if (Test-Path $oldDrfx)  { Remove-Item $oldDrfx  -Force }

   $stream  = [System.IO.File]::Open($zipTemp, [System.IO.FileMode]::Create)
   $archive = New-Object System.IO.Compression.ZipArchive($stream, [System.IO.Compression.ZipArchiveMode]::Create)

   # Entrées de dossiers avec forward slash (DaVinci l'exige)
   $archive.CreateEntry(((Split-Path $editPath -Leaf) + '/')) | Out-Null
   Get-ChildItem -Path $editPath -Recurse -Directory | ForEach-Object {
       $rel = $_.FullName.Substring($editParent.Length + 1).Replace('\', '/') + '/'
       $archive.CreateEntry($rel) | Out-Null
   }

   # Fichiers avec forward slashes
   Get-ChildItem -Path $editPath -Recurse -File | ForEach-Object {
       $rel   = $_.FullName.Substring($editParent.Length + 1).Replace('\', '/')
       $entry = $archive.CreateEntry($rel, [System.IO.Compression.CompressionLevel]::Optimal)
       $es    = $entry.Open()
       $fs    = [System.IO.File]::OpenRead($_.FullName)
       $fs.CopyTo($es)
       $fs.Close(); $es.Close()
   }

   $archive.Dispose(); $stream.Close()
   Rename-Item -Path $zipTemp -NewName "$templateName.drfx"

   $size = [math]::Round((Get-Item $drfxPath).Length / 1KB, 1)
   Write-Host "Packaged: $drfxPath ($size KB)"
   ```
5. Confirm success by printing the output path and file size.
