# VAC 🛡️
This repository contains parts of source code of Valve Anti-Cheat recreated from machine code.

## Introduction
Valve Anti-Cheat (VAC) is user-mode noninvasive anti-cheat system developed by Valve. It is delivered in form of modules (dlls) streamed from remote server. `steamservice.dll` handles module loading.

## Modules
| ID | Purpose | .text section raw size | Source folder |
| --- | --- | --- | --- |
| 1 | Collect information about system configuration.<br>This module is loaded first and sometimes even before any VAC-secured game is launched. | 0x5C00 | Modules/SystemInfo

## Encryption / Hashing
VAC uses several encryption / hashing methods:
- MD5 - hashing data read from process memory
- ICE - decryption of imported functions names and encryption of scan results
- CRC32 - hashing table of WinAPI functions addresses
- Xor - encryption of function names on stack, e.g `NtQuerySystemInformation`. Strings are xor-ed with `^` or `>` or `&` char.

## Module Description

### #1 - SystemInfo
This module is loaded first and sometimes even before any VAC-secured game is launched.

The module calls `GetNativeSystemInfo` function and reads fields from resultant [`SYSTEM_INFO`](https://docs.microsoft.com/en-us/windows/win32/api/sysinfoapi/ns-sysinfoapi-system_info) struct:
- wProcessorArchitecture
- dwProcessorType

Then it calls [`NtQuerySystemInformation`](https://docs.microsoft.com/en-us/windows/win32/api/winternl/nf-winternl-ntquerysysteminformation) API function with following `SystemInformationClass` values (in order they appear in code):
- SystemTimeOfDayInformation - returns undocumented [`SYSTEM_TIMEOFDAY_INFORMATION`](https://www.geoffchappell.com/studies/windows/km/ntoskrnl/api/ex/sysinfo/timeofday.htm) struct, VAC uses two fields:
    - LARGE_INTEGER CurrentTime
    - LARGE_INTEGER BootTime
- SystemCodeIntegrityInformation - returns [`SYSTEM_CODEINTEGRITY_INFORMATION`](https://www.geoffchappell.com/studies/windows/km/ntoskrnl/api/ex/sysinfo/codeintegrity.htm), module saves `CodeIntegrityOptions` field
- SystemDeviceInformation - returns [`SYSTEM_DEVICE_INFORMATION`](https://www.geoffchappell.com/studies/windows/km/ntoskrnl/api/ex/sysinfo/device.htm), module saves `NumberOfDisks` field
- SystemKernelDebuggerInformation - returns [`SYSTEM_KERNEL_DEBUGGER_INFORMATION`](https://www.geoffchappell.com/studies/windows/km/ntoskrnl/api/ex/sysinfo/kernel_debugger.htm), VAC uses whole struct
- SystemBootEnvironmentInformation - returns [`SYSTEM_BOOT_ENVIRONMENT_INFORMATION`](https://www.geoffchappell.com/studies/windows/km/ntoskrnl/api/ex/sysinfo/boot_environment.htm), VAC copies `BootIdentifier` GUID
- SystemRangeStartInformation

For more information about `SYSTEM_INFORMATION_CLASS` enum see [Geoff Chappell's page](https://www.geoffchappell.com/studies/windows/km/ntoskrnl/api/ex/sysinfo/class.htm).
