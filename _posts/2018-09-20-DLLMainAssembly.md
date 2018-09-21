---
layout: post
title: DLL Main API in X86 assembly
category: [Reverse engineering]
tags: [c,programming, reverseengineering, coursera]
---



; BOOL _stdcall DLLMain(HINSTANCE hinstDLL, DWORD fdwReason,
; LPVOID lpvReserved)

_DllMain@12 proc near

push ebp
mov ebp, esp
sub esp, 130h
push edi
sidt fword ptr [ebp-8]
mov eax, [ebp-6]
cmp eax, 8003F400h
jbe short loc_10001C88
cmp eax, 80047400h
jnb short loc_10001C88
xor eax, eax
pop edi
mov esp, ebp
pop ebp
retn 0Ch

==================================== ==================================== ==================================== ==================================== ====================================

Explanation until here:

push ebp
mov ebp, esp

Function prologue Saves the previous stack register (esp) and saves the new one (ebp). We need the ebp to create the frame. We can take the ebp to create local variables and load
arguments of every function.

==================

sub esp, 130h

Reserve 0x130 bytes in the stack. This substract the memory address of the esp register by 130 bytes.

==================

push edi

Saves edi register in the stack.

==================

sidt fword ptr [ebp-8]

Executes sidt instruction and writes the 6 byte IDT register to a specified memory region. In this case, sidt writes the 6-byte IDT register to [ebp-8] => Local variable.

==================

mov eax, [ebp-6]

Reads a double word at [ebp-6] and stores it at eax.

==================

cmp eax, 8003F400h
jbe short loc_10001C88

Compares if eax is below or equal to 0x8003F400. And if it is, it transfers control to loc_1001C88.
jbe stands for Jump if below or equal. The flags that are modified are CF = 1 and ZF = 1.

CF => Carry Flag => Set when the result requires a carry.
ZF => Zero Flag > Set if the result of the previous arithmetic operation is zero.

==================

cmp eax, 80047400h
jnb short loc_10001C88
Compares if eax is above or equal to 0x8004700. And if it is, it transfer control to loc_1001C88
jnb stands for Jump if not above or equal. The flag that is modified is only CF = 0.

==================

xor eax, eax

This instruction clears the value of eax.

==================

pop edi
Restores the saved edi register in line 6.

==================

mov esp, ebp
pop ebp

Restore the previous base frame and stack pointer.

==================

retn 0Ch

Adds 0xC bytes to the stack and then returns to the caller.


==================================== ==================================== ==================================== ==================================== ====================================

xor eax, eax
mov ecx, 49h
lea edi, [ebp-12Ch]
mov dword ptr [ebp-130Ch], 0
push eax
push 2
rep stosd
call CreateToolhelp32Snapshot
mov edi, eax
cmp edi, 0FFFFFFFFh
jnz show loc_10001CB9
xor eax, eax
pop edi
mov esp, ebp
pop ebp
retn 0ch

Explanation until here:

xor eax, eax
mov ecx, 49h

Clear eax register and set ecx to 0x49 value.

==================

lea edi, [ebp-12Ch]

Sets edi to ebp-0x12 value. Since ebp is the frame pointer, ebp-0x12 is a local variable.

==================

mov dword ptr [ebp-130Ch], 0

Writes zero to the double word located at ebp-0x130

==================

push eax
push 2

Push eax and 2 into the stack.

==================

rep stosd

Zeroes a 0x124-byte buffer starting from ebp-0x12.

stosd instruction => Zeroes  0x124 byte buffer starting from ebp-0x12
stosd => 4 byte granularity
ecx => 0x49 times
49h * 4 => 124-bytes

==================

call CreateToolhelp32Snapshot

Calls CreateToolhelp32Snapshot

Handle WINAPI CreateToolhelp32Snapshot {
  _In_ DWORD dwFlags,
  _In_ DWORD th32ProcessID
};

Win32 API functions follow STDCALL calling convention.
STDCALL =>
Parameters: Pusehd on the stack from right to left. Callee must clean the stack.
Return Value: Stored in eax.
Non-Volatile Registers: ebp, esp, ebx, esi, edi.

So in this case the dwFlags parameter will be 0x2 and the th32ProcessID will be 0x0.
This function enumerates all processes on the system and returns a handle to be used in Process32Next.

==================

mov edi, eax
cmp edi, 0FFFFFFFFh

Save the return value in edi and check if it is -1 

==================

xor eax, eax
pop edi
mov esp, ebp
pop ebp
retn 0ch

If value is -1, it is set to 0 and it returns. 

==================

If not, the program continues in the following instruction

jnz show loc_10001CB9


==================================== ==================================== ==================================== ==================================== ====================================

lea eax, [ebp-130h]
push esi
push eax
push edi 
mov dword ptr [ebp-130h], 128h
call Process32First
test eax, eax 
jz short loc_10001D24
mov esi, ds:_stricmp
lea ecx, [ebp-10Ch]
push 10007C50h 
push ecx
call esi; _stricmp 
add esp, 8
test eax, eax
jz short loc_10001D16

==================


lea eax, [ebp-130h]

Load to eax a local variable set to 0. 

==================

mov dword ptr [ebp-130h], 128h

Initialize the variable to 128h 

==================


push esi
push eax
push edi 

Push the values into the stack 

==================

Calling the function Process32First 

BOOL WINAPI Process32First{
	_In_ Handle hSnapshot, 
	_Inout_ LPPROCESSENTRY32 lppe
}

typedf struct tagProcessENNTRY32 {
	DWORD dwsize;
	DWORD cntUsage; 
	DWORD th32ProcessID; 
	ULONG_PTR th32DefaultHeapID;
	DWORD th32ModuleID;
	DWORD cntThreads; 
	DWORD th32ParentProcessID;
	LONG pcPriClassBase; 
	DOWRD dwFlags; 
	TCHAR szExeFile[MAX_PATH];
} PROCESSENTRY32, *PROCESSENTRY32; 


00000000 PROCESSENTRY32 struc ; (sizeof = 0x128)
00000000 dwSize dd ?
00000004 cntUsage dd ? 
00000008 thProcess32ID dd ? 
0000000C th32DefaultHeapID ? 
00000010 th32ModuleID dd ?
00000014 cntThreads dd ? 
00000018 th32ParentProcessID dd ? 
0000001C pcPriClassBase dd ? 
00000020 dwFlags dd ? 
00000024 szExeFile db 260 dup(?)
00000028 PROCESSENTRY32 ends 

As we can see this API takes two parameters, hSnapShot is edi (The handle of all the process running returned by the function CreateToolhelp32Snapshot). 
lppe is the the value of a local variable. 

Since lppe points to a PROCESSENTRY32 structure we can assume that the variable at [ebp-130h] is of the same type. 

If we see the previous lines: 

xor eax, eax
mov ecx, 49h
lea edi, [ebp-12Ch]
mov dword ptr [ebp-130Ch], 0
push eax
push 2
rep stosd

What they were doing is to initialize the structure to 0. (Remember how stosd works). 

==================

test eax, eax 

Remember that in the convention STDCALL, register eax saves the return value of the function. So here we are checking that eax contains a Not 0 value. 
If zero, we jump to  => jz short loc_10001D24

jz => Jumps if Zero. ZF = 1. 

Loads EIP with the specified address (loc_10001D24). 

If not, we continue. 

==================

mov esi, ds:_stricmp

Saves the address of the stricmp function in ESI. 


==================


lea ecx, [ebp-10Ch]

Load whatever is in [ebp-10Ch] <= Local variable. into ecx. 	

==================

push 10007C50h 

This is a string in the data section. The data section is already defined in the program. 

.data: 10007C50h 65 78 70 6C 6F+Str2 db 'explorer.exe', 0

==================

push ecx

Push ecx into the stack. 

If we remember, ecx is what is in the adress [ebp-10Ch]. This should be the value szExeFile. 

If the struct tagPROCESSENTRY32 starts at 130h and ends at 8h, we have the following: 

130h ProcessEntryStruc; 
130h dwSize
12Ch cntUsage 
128h th32ProcessID
124 th32DefaultHeapID
120 th32ModuleID
11C cntThreads
118 th32ParentProcessID
114 pcPriClassBase
110 dwFlags 
10C szExeFile >>>>>>>>>>>>> 

That is the reason that ecx has the value of szExeFile. 

==================

call esi; _stricmp 

The function _stricmp is called and it accepts two parameters. ECX and 10007C50 (String). 

==================

add esp, 8

This lines clear the stack (moving the esp 8 bytes). Why? Because the function stricmp uses the convention CDECL: 

Parameters: Pushed on the stack from right to left. Caller must clean up the stack after the call. 
Return Value: Return in eax register. 
Non-Volatile Registers: ebp, esp, ebx, esi, edi. 

==================

test eax, eax

Check if the return value of stricmp is not zero. 

==================

jz short loc_10001D16

If zero, jump to the specified address. (This means that the string matched "explorer.exe"). 

==================

==================================== ==================================== ==================================== ==================================== ====================================

loc_10001CF0: 

lea edx, [ebp-130h]
push edx
push edi
call Process32Next
test eax, eax 
jz short loc_10001D24
lea eax, [ebp-10Ch]
push 10007C50h
push eax
call esi;_stricmp
add esp, 8 
test eax, eax 
jnz short loc_10001CF0 

Loop that does the same, with two exit points. 

After this loop, we continue the execution

==================================== ==================================== ==================================== ==================================== ====================================

loc_10001D16: 
mov eax, [ebp-118h]
mov ecx, [ebp-128h]
jmp short loc_10001D2A
loc_10001D24 
mov eax, [ebp+0Ch]
mov ecx, [ebp+0Ch]
loc_10001D2A
cmp eax, ecx
pop esi
jnz short loc_10001D38
xor eax, eax 
pop edi 
mov esp, ebp
pop ebp
retn 0Ch


