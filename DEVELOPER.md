# Vellum developer guide

This guide explains how to create, build, and host applications for the Vellum ecosystem.

## 1. Architecture
PocketBook devices run Linux (ARMv7). While they natively support C/C++ apps (InkView), Vellum enables a "Sidecar" architecture. This allows you to write the heavy logic in modern languages (Go, Rust, Python) while using a lightweight C wrapper for the UI.

### Folder structure
To ensure compatibility with Vellum, your app must be packaged as a .zip file with this exact structure:

```text
/applications/
├── MyApp.app            # The launcher (C wrapper)
└── MyApp/               # The container (dependency folder)
    ├── binary_arm       # Your static binary (Go/Rust/etc.)
    ├── config.conf      # User configuration
    ├── icon.bmp         # App icon
    └── icon_f.bmp       # Focused app icon

```

---

## 2. The build system

We use a dockerized build pipeline. You do not need to install the PocketBook SDK on your machine.

### Prerequisites

* **Docker Desktop**
* **PowerShell** (Windows) or pwsh (Linux/Mac)

### The build script (`build.ps1`)

Save this script in your project root. It handles cross-compilation (using the official SDK image) and packaging (using Alpine Linux to preserve permissions).

```powershell
# build.ps1 - Vellum app builder
# Usage: .\build.ps1
# Requirements: Docker

param (
    [string]$AppName = "MyApp",       # Name of your .app file
    [string]$SourceFile = "main.c",   # Your C entry point in /src
    [string]$BinaryName = "engine"    # Name of your sidecar binary (if any)
)

$Image = "5keeve/pocketbook-sdk:6.3.0-b288-v1"
$WorkDir = Get-Location

# Configs
$CompilerPath = "/SDK/usr/bin/arm-obreey-linux-gnueabi-gcc"
$IncludeDirs = @(
    "/SDK/usr/arm-obreey-linux-gnueabi/sysroot/usr/local/include",
    "/SDK/usr/arm-obreey-linux-gnueabi/sysroot/usr/include/freetype2/",
    "/SDK/usr/arm-obreey-linux-gnueabi/sysroot/usr/include"
)
$IncludeFlags = $IncludeDirs | ForEach-Object { "-I$_" }
$IncludeString = $IncludeFlags -join " "

# Ensure output directories exist
New-Item -ItemType Directory -Force -Path "$WorkDir\build_arm\$AppName" | Out-Null

Write-Host ">> Building $AppName.app..." -ForegroundColor Cyan

# 1. Compile C wrapper
# Mount the current folder to /src inside Docker
$Cmd = "$CompilerPath /src/src/$SourceFile -o /src/$AppName.app $IncludeString -linkview -lm"
docker run --rm --entrypoint /bin/bash -v "${WorkDir}:/src" $Image -c $Cmd

if (-not (Test-Path "$WorkDir\$AppName.app")) {
    Write-Error "Compilation failed!"
    exit 1
}

# 2. Package files
Write-Host ">> Packaging..."
Copy-Item "$WorkDir\$AppName.app" "$WorkDir\build_arm\$AppName\"

# Copy the 'Container' folder if it exists (for sidecar binaries)
if (Test-Path "$WorkDir\bin\$BinaryName") {
    New-Item -ItemType Directory -Force -Path "$WorkDir\build_arm\$AppName\$AppName" | Out-Null
    Copy-Item "$WorkDir\bin\$BinaryName" "$WorkDir\build_arm\$AppName\$AppName\"
}

# 3. Zip (via Docker)
# because Windows zip destroys Linux 'executable' permissions
Write-Host ">> Zipping..."
$ZipPath = "$WorkDir\build_arm\$AppName.zip"
if (Test-Path $ZipPath) { Remove-Item $ZipPath }

docker run --rm -v "${WorkDir}:/work" alpine /bin/sh -c "apk add --no-cache zip && cd /work/build_arm/$AppName && chmod +x $AppName.app && zip -r /work/build_arm/$AppName.zip ."

Write-Host ">> DONE: build_arm\$AppName.zip is ready." -ForegroundColor Green

```

---

## 3. Hosting your own repository

Vellum is decentralized. You can host your own repository on GitHub Pages, Vercel, or any other static file server.

### Repository structure example

Create a repo with this layout:

```text
/my-vellum-repo
├── repo.json          # The index file
├── icons/             # Folder for PNG icons
└── apps/              # Folder for ZIP files

```

### The index (`repo.json`)

This is the file Vellum reads. It must list all available apps in your repo.

```json
{
  "repo_name": "My custom repo",
  "maintainer": "Your name",
  "url": "https://yourname.github.io/my-vellum-repo/repo.json",
  "packages": [
    {
      "id": "com.yourname.myapp",
      "name": "MyApp",
      "version": "1.0.0",
      "genre": "Utility",
      "icon": "https://yourname.github.io/my-vellum-repo/icons/myapp.png",
      "download": "https://yourname.github.io/my-vellum-repo/apps/MyApp.zip",
      "description": "Description of what this app does.",
      "compatibility": {
        "firmware": "6.0+",
        "model": "all"
      }
    }
  ]
}

```

### Publishing

1. **Commit** your `.zip` files to `apps/` and `.png` icons to `icons/`.
2. **Update** `repo.json` with the new versions.
3. **Enable GitHub Pages** in your repository settings.
4. Users can now add your URL (`https://yourname.github.io/my-vellum-repo/repo.json`) to Vellum to install your apps.

```

```
