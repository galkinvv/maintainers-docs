# Actions
These are the one step instruction type (which we have seen before), Bottles
supports several and will certainly be extended in the future. However, these
alone are not enough and require other parameters, sometimes common and other
times specific to the action. We will cover them all in this documentation.

## Actions table
The following table lists all the actions that Bottles supports for
dependencies steps.

| Action | Description |
| ------ | ----------- |
| `install_exe` or `install_msi` | Install an executable |
| `download_archive` | Download an archive |
| `archive_extract` | Extract an archive |
| `cab_extract` | Extract a Windows Cabinet |
| `get_from_cab` | Extract a file from a Windows Cabinet |
| `override_dll` | Override a DLL |
| `uninstall` | Call an uninstaller by its name |
| `set_windows` | Set the current Windows version |
| `set_register_key` | Set a Windows registry key |
| `copy_file` or `copy_dll` | Copy a file from one place to another |
| `install_fonts` | Copy one or more fonts to the bottle windows/Fonts path |
| `register_font` | Register a font in the Windows registry |
| `replace_font` | Replace a font replacement in the Windows registry |

## Common definitions
### Downloads and local files
All actions that support a `url` key will download the file from that URL. This
can also be used to define a local file (from the Bottles' temp folder). The
correct syntax to use the local file is:

```yaml
..
url: temp/file...
..
```

the `temp/` part (placeholder) will be automatically replaced by the full path
to the Bottles' temp folder.

## `install_exe` or `install_msi`
This action is used to install an executable.

### Parameters
| Key | Description |
| --- | ----------- |
| `file_name` | The name of the file to be installed |
| `url` | The URL or path of the file to be downloaded |
| `rename` | The new name of the file if it must be renamed |
| `file_checksum` | The checksum of the file (MD5) |
| `arguments` | The arguments to be passed to the executable |
| `environment` | The environment variables to be injected |

### Example
```yaml
- action: install_exe
  environment: 
    WINEDLLOVERRIDES: fusion=b
  file_name: ndp48-x86-x64-allos-enu.exe
  url: https://download.visualstudio.microsoft.com/download/pr/7afca223-55d2-470a-8edc-6a1739ae3252/abd170b4b0ec15ad0222a809b761a036/ndp48-x86-x64-allos-enu.exe
  rename: dotnet48.exe
  file_checksum: 4e7a8586d06ce1f8cebac0d4bc379e16
  arguments: /sfxlang:1027 /q /norestart
```

## `download_archive`
This action is used to download an archive, can also be used to download
any other kind of file. It only downloads the file, it does not extract, install
or anything else.

### Parameters
| Key | Description |
| --- | ----------- |
| `file_name` | The name of the file to be downloaded |
| `url` | The URL or path of the file to be downloaded |
| `rename` | The new name of the file if it must be renamed |
| `file_checksum` | The checksum of the file (MD5) |

### Example
```yaml
- action: download_archive
  file_name: msxml6-KB973686-enu-amd64.exe
  url: https://web.archive.org/web/20190122095451/https://download.microsoft.com/download/1/5/8/158F681A-E595-472B-B15E-62B649B1B6FF/msxml6-KB973686-enu-amd64.exe
  file_checksum: 13a292beb9ebe46ac97b8a4352fe2cd5
```

## `archive_extract`
This action is used to download and extract an archive. The archive will be
extracted to a new directory named as the `file_name` or `rename` in the 
Bottles' temp directory.

### Parameters
| Key | Description |
| --- | ----------- |
| `file_name` | The name of the file to be downloaded |
| `url` | The URL or path of the file to be downloaded |
| `rename` | The new name of the file if it must be renamed |
| `file_checksum` | The checksum of the file (MD5) |

### Example
```yaml
- action: archive_extract
  file_name: faudio-20.07.tar.xz
  rename: faudio_20_07.xz
  url: https://github.com/Kron4ek/FAudio-Builds/releases/download/20.07/faudio-20.07.tar.xz
  file_checksum: 7385a4d276f940e025488e74a967789b
```

## `cab_extract`
This action is used to extract a Windows Cabinet, can also download it
before extracting.

### Parameters
| Key | Description |
| --- | ----------- |
| `file_name` | The name of the file to be downloaded |
| `url` | The URL or path of the file to be downloaded |
| `rename` | The new name of the file if it must be renamed |
| `file_checksum` | The checksum of the file (MD5) |
| `dest` | The destination folder where the files will be extracted, Bottles will automatically create it if does not exist |

#### Supported `dest` values
The destination should start with one of the following placeholders:

| Placeholder | Description |
| ----------- | ----------- |
| temp/ | Will be automatically replaced with the full path to the Bottles' temp directory |

If the destination does not start with one of the supported placeholders, it
will fail with an error. It is not possible to extract a whole Windows Cabinet
to a bottle folder, **hardscripting this will result in a rejection by the team.**

### Example
```yaml
- action: cab_extract
  file_name: directx_Jun2010_redist.exe
  url: https://download.microsoft.com/download/8/4/A/84A35BF1-DAFE-4AE8-82AF-AD2AE20B6B14/directx_Jun2010_redist.exe
  file_checksum: 822e4c516600e81dc7fb16d9a77ec6d4
  dest: temp/directx_Jun2010_redist/
```

## `get_from_cab`
This action is used to extract a file from a Windows Cabinet.

### Parameters
| Key | Description |
| --- | ----------- |
| `source` | The path to the Windows Cabinet file |
| `file_name` | The file to be extracted (support wildcards) |
| `dest` | The destination folder where the files will be extracted, Bottles will automatically create it if does not exist |

#### Supported `dest` values
The destination should start with one of the following placeholders:

| Placeholder | Description |
| ----------- | ----------- |
| temp/ | Will be automatically replaced with the full path to the Bottles' temp directory |
| win64/ | Will be automatically replaced with the full path to the windows/ system 64bit folder of the running bottle |
| win32/ | Will be automatically replaced with the full path to the windows/ system 32bit folder of the running bottle |

> **Note**: The `win64` and `win32` placeholders are extremely important, as 
> they change according to the bottle's architecture. Windows (and WINE) 32bit 
> use the system32 folder to store 32bit resources, while 64bit systems uses 
> the system32 folder to store 64bit resources and syswow64 to store 32bit.

If the destination does not start with one of the supported placeholders, it
will fail with an error. It is not possible to extract a whole Windows Cabinet
to a bottle folder, **hardscripting this will result in a rejection by the team.**

### Example
```yaml
- action: get_from_cab
  source: directx_Jun2010_redist.exe
  file_name: Jun2010_D3DCompiler_43_x86.cab
  dest: temp/d3dcompiler_43_x86/
```

example using the `win64` and `win32` placeholders:

```yaml
- action: get_from_cab
  source: d3dcompiler_43_x86/Jun2010_D3DCompiler_43_x86.cab
  file_name: D3DCompiler_43.dll
  dest: win32
  rename: d3dcompiler_43.dll

- action: get_from_cab
  source: d3dcompiler_43_x64/Jun2010_D3DCompiler_43_x64.cab
  file_name: D3DCompiler_43.dll
  dest: win64/
  rename: d3dcompiler_43.dll
```

> This page is not complete.