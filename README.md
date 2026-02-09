# Windows Token Impersonation POC

A Proof-of-Concept (POC) demonstration of Windows token impersonation techniques for educational and security research purposes. This tool demonstrates privilege escalation from Administrator (High Integrity) to NT AUTHORITY\SYSTEM by duplicating and impersonating process tokens.

## ⚠️ Disclaimer

**FOR EDUCATIONAL PURPOSES ONLY**

This tool is intended solely for:
- Security research and education
- Authorized penetration testing
- Understanding Windows security mechanisms
- Developing defensive security measures

**Unauthorized use of this tool may violate laws.** Only use this tool on systems you own or have explicit written permission to test.

## Overview

This POC demonstrates the following Windows security concepts:

1. **Process Token Access** - Opening handles to process tokens
2. **Token Duplication** - Creating duplicate tokens with elevated privileges
3. **Privilege Escalation** - Elevating from High Integrity Admin to SYSTEM
4. **Process Creation** - Spawning processes with impersonated tokens

## Prerequisites

- Windows operating system (tested on Windows 10/11)
- Administrator privileges (High Integrity Level)
- Visual Studio 2022 or later (for C++20 support with `std::format`)
- Target process running as SYSTEM (e.g., `winlogon.exe`)

## Compilation

### Visual Studio
```bash
# Open solution in Visual Studio 2022
# Ensure C++20 standard is enabled
# Build in Release or Debug configuration
```
## Usage
```bash
TokenTheft.exe  
```

### Parameters

- `<ProcessID>` - Process ID of a SYSTEM-level process to impersonate
- `<CommandToExecute>` - Full path to the executable to run as SYSTEM

### Examples

**Spawn CMD as SYSTEM:**
```bash
# Find a SYSTEM process ID (e.g., winlogon.exe)
tasklist /v /fi "username eq SYSTEM"

# Execute token theft
TokenTheft.exe 560 C:\Windows\System32\cmd.exe
```
## How It Works

### Step-by-Step Process

1. **Open Target Process**
   - Obtains a handle to the target SYSTEM process using `OpenProcess()`
   - Requires `PROCESS_QUERY_INFORMATION` access rights

2. **Access Process Token**
   - Opens the primary token of the target process with `OpenProcessToken()`
   - Requires `TOKEN_QUERY | TOKEN_DUPLICATE` permissions

3. **Duplicate Token**
   - Creates a duplicate primary token using `DuplicateTokenEx()`
   - New token has `TOKEN_ADJUST_PRIVILEGES | TOKEN_ASSIGN_PRIMARY` rights

4. **Enable Privileges**
   - Enables `SeAssignPrimaryTokenPrivilege` in the duplicated token
   - Required for `CreateProcessAsUserA()` API call

5. **Impersonate Token**
   - Temporarily impersonates the duplicated token with `ImpersonateLoggedOnUser()`
   - RAII pattern ensures automatic reversion via `RevertToSelf()`

6. **Create Process**
   - Spawns the specified command as SYSTEM using `CreateProcessAsUserA()`

## Technical Details

### Required Privileges

The attacking process must have:
- **Administrator rights** (High Integrity Level minimum)
- Ability to open handles to SYSTEM processes

### Windows API Functions Used

- `OpenProcess()` - Open process handle
- `OpenProcessToken()` - Access process token
- `DuplicateTokenEx()` - Duplicate access token
- `AdjustTokenPrivileges()` - Modify token privileges
- `ImpersonateLoggedOnUser()` - Impersonate security context
- `CreateProcessAsUserA()` - Create process with token
- `RevertToSelf()` - Restore original security context

### RAII Implementation

The code uses modern C++ RAII patterns to prevent resource leaks:
```cpp
class HandleGuard        // Automatic handle cleanup
class ImpersonationGuard // Automatic token reversion
```

## Code Highlights

- **Modern C++20** with `std::format`
- **RAII patterns** for automatic resource management
- **Exception safety** with proper error handling
- **Move semantics** for efficient handle management

## License

MIT License - See LICENSE file for details

## Author

t3nb3w 

---

**Remember:** Use responsibly and ethically. Always obtain proper authorization before testing.
