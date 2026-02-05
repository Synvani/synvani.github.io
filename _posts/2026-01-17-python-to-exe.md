---
published: true
title: Python ➝ Exe with Minimal False Positives
author: synvan
date: 2026-01-17 14:10:00 +0200
categories: [Code, Python]
tags: [Python, Pyinstaller, Executables]
render_with_liquid: false
description: How to compile executables and minimize those endless antivirus false positives
media_subpath: '/assets/img/posts/07_python-to-exe/'
image:
  path: 00_python-to-exe.jpg
  alt: 
  in_post: true
---

I recently finished building a small desktop app in Python – a parser tool for an old action RPG game. It worked well and I figured it was time to share it with others who might also enjoy generating insights out of their play time. Simple enough, right? Just package it into an executable with PyInstaller and distribute the `.exe` file.

That's when things got annoying – some antivirus vendors decided my little Python app was malware. VirusTotal showed **4~19** (out of 71) engines flagging it as suspicious. Users were getting warnings, Windows Defender was quarantining files, and I was getting messages asking if I was trying to install a virus on a friend's computer. Fun times.

After some digging and experimentation, I managed to cut those false positives in half by rebuilding PyInstaller's bootloader from source. This post documents the journey and the steps that actually worked.

## Why Does This Happen?

Legitimate Python executables get flagged as malware for a few reasons:

1. **Known Bootloader Signatures** – PyInstaller's default bootloader is widely recognized by AV vendors and often flagged heuristically. It's the same binary that millions of apps (including actual malware) use.
2. **UPX Compression** – The UPX packer is commonly used by malware authors, so AV engines treat it with suspicion.
3. **Onefile Extraction** – Onefile executables extract to temp folders at runtime, which mimics behavior seen in malware.

The key insight here is that the biggest culprit is the bootloader signature. If you're using the same pre-compiled bootloader as everyone else, your app shares a fingerprint with every PyInstaller-built executable out there – including the malicious ones.

## Comparing Build Methods

Before settling on a solution, I tested a few different approaches:

| Method                | Mode     | VirusTotal Score | Notes                                              |
| :-------------------- | :------- | :--------------- | :------------------------------------------------- |
| Nuitka Compiler       | One-file | 19/71            | Surprisingly worse despite being a "true" compiler |
| Nuitka Compiler       | One-dir  | 5/71             | Not significantly better                           |
| Standard PyInstaller  | One-file | 4/71             | Baseline – extracts to temp at runtime             |
| Standard PyInstaller  | One-dir  | 4/71             | Same score, no temp extraction                     |
| **Custom Bootloader** | **One-file** | **1/71  ✓**       | **75% reduction** – unique binary signature            |

Nuitka was a bit of a letdown. I had high hopes since it compiles Python to C, but it actually performed worse than PyInstaller for this use case. The custom bootloader approach turned out to be the winner.

## The Solution: Rebuild PyInstaller's Bootloader

The idea is simple – instead of using PyInstaller's pre-compiled bootloader (which AV vendors have fingerprinted), we compile it ourselves from source. This creates a unique binary signature that isn't in any AV database.

### Prerequisites

- Windows 10/11
- Python 3.12+ installed
- An active virtual environment for your project
- Git for Windows

I've ran this on my Windows machine, but you can tweak some of the steps to make it work just as well on other operating systems.

### Step 1: Install MSYS2 and GCC Compiler

We need a C compiler to rebuild the bootloader from source. MSYS2 provides an easy way to get GCC on Windows.

1. Download and install MSYS2 from [msys2.org](https://www.msys2.org/) (default location is `C:\msys64`).

2. Open **MSYS2 UCRT64** terminal from the Start Menu and install GCC:
   ```bash
   pacman -S mingw-w64-ucrt-x86_64-gcc
   ```
   {: .nolineno }

3. Verify it worked:
   ```bash
   gcc --version
   ```
   {: .nolineno }

   You should see something like `gcc.exe (Rev8, Built by MSYS2 project) 15.2.0`.

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> Optionally, add `C:\msys64\ucrt64\bin` to your System PATH if you want GCC available in PowerShell. Not required if you're building from the MSYS2 terminal.
{: .prompt-info }
<!-- markdownlint-restore -->


### Step 2: Clone PyInstaller Source

We need the bootloader source code to compile it ourselves. The easiest way is to clone the PyInstaller repository:

```powershell
# Create a dedicated folder for the rebuild
mkdir C:\Users\$env:USERNAME\Python\pyinstaller_rebuild
cd C:\Users\$env:USERNAME\Python\pyinstaller_rebuild

# Clone PyInstaller repository
git clone https://github.com/pyinstaller/pyinstaller.git
```
{: .nolineno }


### Step 3: Rebuild the Bootloader

This is where the magic happens. We're going to compile the bootloader from source, creating a unique binary that AV vendors haven't fingerprinted.

1. Open **MSYS2 UCRT64** terminal (not PowerShell – the build script expects a Unix-like environment).

2. Navigate to the bootloader directory using Unix-style paths:
   ```bash
   cd /c/Users/$USER/Python/pyinstaller_rebuild/pyinstaller/bootloader
   ```
   {: .nolineno }

3. Run the build using your project's Python interpreter:
   ```bash
   /c/Users/$USER/Python/your_venv/Scripts/python.exe ./waf distclean all --target-arch=64bit
   ```
   {: .nolineno }

   Replace `your_venv` with the path to your virtual environment.

4. If everything went well, you should see:
   ```
   'all' finished successfully
   ```
   {: .nolineno }

5. Verify the compiled executables exist:
   ```bash
   find build -name "*.exe"
   ```
   {: .nolineno }

   You should see four files:
   - `build/debug/run_d.exe`
   - `build/debugw/runw_d.exe`
   - `build/release/run.exe`
   - `build/releasew/runw.exe`


### Step 4: Replace PyInstaller's Default Bootloader

Now we need to copy our freshly compiled bootloader into the virtual environment, replacing the pre-compiled one that came with PyInstaller.

From PowerShell:

```powershell
# Copy all rebuilt bootloader executables
Get-ChildItem -Path "C:\Users\$env:USERNAME\Python\pyinstaller_rebuild\pyinstaller\bootloader\build" -Recurse -Filter "*.exe" | Copy-Item -Destination "C:\Users\$env:USERNAME\Python\your_venv\Lib\site-packages\PyInstaller\bootloader\Windows-64bit-intel\" -Force
```
{: .nolineno }

Verify the copy worked by checking the timestamps:

```powershell
Get-ChildItem "C:\Users\$env:USERNAME\Python\your_venv\Lib\site-packages\PyInstaller\bootloader\Windows-64bit-intel\*.exe"
```
{: .nolineno }

The files should show today's date, confirming they were replaced.


### Step 5: Configure Your Spec File

If you're using a `.spec` file (recommended for any non-trivial project), there are a few key settings to get right.

Here's a minimal example for a onefile build:

```python
exe = EXE(
    pyz,
    a.scripts,
    a.binaries,
    a.zipfiles,
    a.datas,
    [],
    exclude_binaries=False,  # False = onefile
    name='MyApp',
    debug=False,
    upx=False,  # IMPORTANT: Disable UPX
    console=False,
    icon='assets/icon.ico',
)
```
{: .nolineno }

The key settings:
- **`upx=False`** – UPX compression is known to trigger false positives. Just don't use it.
- **`exclude_binaries=False`** – This creates a single executable (onefile mode).

<!-- markdownlint-capture -->
<!-- markdownlint-disable -->
> For onefile mode, do not include a `COLLECT` block in your spec file. That's only needed for onedir builds.
{: .prompt-warning }
<!-- markdownlint-restore -->

If PyInstaller isn't detecting all your imports automatically, you may need to add hidden imports:

```python
hiddenimports=[
    'tkinter',
    'tkinter.ttk',
    'tkinter.filedialog',
    'tkinter.messagebox',
    'ctypes',  # Often needed but not auto-detected
],
```
{: .nolineno }

My app was built with `tkinter` (old but gold!) per the example above – just make sure you're not missing any required packages you built your app with.

### Step 6: Build and Test

With everything in place, run the build:

```powershell
pyinstaller --clean MyApp.spec
```
{: .nolineno }

Your executable will be in the `dist` folder. Test it locally first to make sure everything works, then upload to [VirusTotal](https://www.virustotal.com/) to check the false positive rate.


## Troubleshooting

A few issues I ran into along the way:

#### Bootloader build fails with "could not configure a C compiler"

Make sure you're running the build from the **MSYS2 UCRT64 terminal**, not PowerShell. The build script expects a Unix-like environment with GCC in the path.

#### ModuleNotFoundError for various modules

PyInstaller's auto-detection isn't perfect. Add missing modules to the `hiddenimports` list in your spec file. Common culprits include `ctypes`, `tkinter` submodules, and `queue`.

#### PermissionError when running PyInstaller

Close any running instances of your app and delete the `build` and `dist` folders manually. Then run PyInstaller with the `--clean` flag.


## Things to Keep in Mind

**Rebuild after PyInstaller updates.** If you upgrade PyInstaller via pip, it will overwrite your custom bootloader with the default one. You'll need to copy your compiled bootloader again (or rebuild it).

**Onefile vs onedir didn't matter for false positives in my case.** I tested both and got the same detection rate. Use whichever is more convenient for your use case.

**Skip Nuitka for this problem.** I had high hopes for Nuitka since it's a "real" compiler, but it actually performed worse than PyInstaller in terms of false positives.

**Consider code signing if this isn't enough.** I haven't tested it myself, but signing your executable with a code signing certificate might help further. It's not cheap (few $100s/year), but it's an option if you're distributing to a lot of users.


## Wrap-up

That's it – by rebuilding PyInstaller's bootloader from source, I managed to cut VirusTotal false positives from 19/71 down to 1/71. It's not perfect, but it's a significant improvement for about an hour of one-time setup.

The key insight is that the default bootloader is the main culprit. It's the same binary that millions of apps use, including actual malware. By compiling your own, you get a unique signature that hasn't been fingerprinted by AV vendors.

If you're distributing Python desktop apps and getting complaints about false positives, give this approach a try. It's worked well for me.
