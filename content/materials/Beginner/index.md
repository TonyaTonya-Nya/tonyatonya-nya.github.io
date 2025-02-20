---
title: Linux 背景知識與基礎指令
weight: 1
date: 2024-09-08 00:00:00+0000
categories:
  - Materials
---

## 🦉Linux 背景知識

### Linux 簡介

Linux，指的是以 Linux Kernel 來開發的眾多作業系統，它們通常是開源的。常見的 Linux 作業系統有 Ubuntu、CentOS、Debian 等。每一種 Linux 的作業系統都對特定用途進行了強化與演變，各自應用在不同的領域。例如 Android 系統也是 Linux 家族的一員。在本課程中，我們最常利用的 Linux 作業系統會是 Ubuntu 以及 Kali Linux。

Ubuntu 是基於 Debian 的一套 Linux 作業系統，由 Canonical Ltd. 開發。它既具備強大的行動及 PC 功能，又有許多的套件以及應用程式用以滿足使用者的各種需求，同時，它也是最知名的 Linux 作業系統之一。

Kali Linux 是基於 Debian 的一套 Linux 作業系統，由 Offensive 所開發。專門用於滲透測試。它通常用於在無線網路、Web 應用程式、伺服器上進行漏洞掃描及攻擊，因此包含了許多資訊安全領域相關的工具。

這兩套作業系統都具有完善的開發團隊來維護及更新，同時也有眾多的使用者社群互相討論，由於 Linux 作業系統都是以 Linux Kernel 為基礎進行開發，學習使用一套 Linux 作業系統後，未來使用其它 Linux 作業系統便不需要從頭開始學習，因此我們鼓勵大家在課程之外的時間多多摸索、學習使用 Linux 作業系統。

### 指令介面

電子計算機(或稱電腦、計算機)。是指根據一系列指令執行任意算術或邏輯操作序列的裝置。我們在使用計算機時，通常會透過圖形化使用者介面 (Graphical User Interface，GUI) 來進行操作。例如：桌面圖示、按鈕、選單、對話框等。這些直觀的操作介面，使得使用者不需要具備專業的計算機技術及技術，也能輕鬆的操作計算機底層的軟硬體並且使用它們提供的服務。然而圖形化使用者介面在直觀且人性化的操作介面背後，卻需要消耗更多的記憶體和計算機的處理能力來產生並輸出圖形化介面的各種元素。

因此，除了透過圖形化使用者介面與計算機互動以外，我們也可以透過指令介面 (Command-Line Interface，CLI) 以純文字的方式與計算機進行互動。在 Linux 上，最普遍的指令介面則是 Linux 自帶的終端機 (Terminal)，你可能聽說過其他相似的名詞，像是 Console 、 shell，它們在廣義上指的都是同樣的一件事 —— CLI。

事實上，shell 泛指任何能擔當起使用者與 kernel 交互的處理程式。而在 Liunx 上有幾個知名的 shell。例如：sh、bash、zsh 等。不同的 shell 會提供不同的功能來加速使用者與 kernel 互動的效率，但它們都具有相同的基本用途，執行程式與解釋指令。我們將在之後的課程學習如何使用基本指令，透過 shell 來與 kernel 進行互動。

## 🦉Linux 基礎指令

### 指令概覽

本小節列舉了 Linux 常用的指令，並且將會在後面的小節一一說明。
| 指令 | 說明 |
| --------| -------- |
| man | 輸出指令的說明文件|
| pwd | 輸出當前目錄 |
| cd | 目錄切換 |
| ls | 輸出當前目錄中的檔案 |
| mkdir | 建立目錄 |
| mv | 移動、重新命名目錄或檔案 |
| rm | 刪除目錄或檔案 |
| touch | 新增檔案 |
| echo | 輸出字串內容 |
| cat | 輸出檔案內容 |
| cp | 複製檔案 |
| file | 顯示檔案類型 |
| grep | 文本搜尋 |
| find | 搜尋 |
| sudo | 超級使用者 |
| clear | 清除指令界面 |
| exit | 關閉指令界面 |

### 絕對路徑與相對路徑

使用 `cd` 指令時，可以使用絕對路徑或相對路徑進入目錄。Linux 的絕對路徑通常是指以斜線（/）開頭的完整路徑，用於指定檔案、目錄或其它資源的位置。例如，`/etc/passwd` 是一個 Linux 的絕對路徑。在 Linux 中，單獨的斜線（/）表示根目錄，所有檔案、資料夾的位置都是從根目錄而來的路徑。

Linux 的相對路徑表示相對於當前目錄，目標檔案或目錄的位置。例如， `./test.txt` 是一個 Linux 的相對路徑，表示當前目錄中的 test.txt 檔案。

用於表示相對路徑的常用符號有以下三種:

| 符號 | 代表意義   |
| ---- | ---------- |
| ..   | 上一層目錄 |
| .    | 該層目錄   |
| ~    | 家目錄     |

舉例來說，如果有一目錄架構如下圖所示，我們當前所在的目錄為 `Public/dir2`。

```shell
Public
├── dir1
│    └── dir1-A
│        └── file1-A
└── dir2
    └── file2
```

如果我們想要找到 file2 ，可以透過以下三種方式來找到:
其中第一種是以絕對路徑表示，後兩種則是以不同的符號來表示相對路徑。

```shell
kiwis@ubuntu:~/Public/dir2$ /home/kiwis/Public/dir2/file2
kiwis@ubuntu:~/Public/dir2$ ./file2
kiwis@ubuntu:~/Public/dir2$ ~/Public/dir2/file2
```

同樣的道理，如果我們想要找到 file1-A ，也可以透過以下三種方式來找到:

```shell
kiwis@ubuntu:~/Public/dir2$ /home/kiwis/Public/dir1/1-A/file1-A
kiwis@ubuntu:~/Public/dir2$ ../dir1/1-A/file1-A
kiwis@ubuntu:~/Public/dir2$ ~/Public/dir1/1-A/file1-A
```

### man

在介紹常用的 Linux 基礎指令前，我們要先介紹一個指令的萬事通 —— man。`man` 指令是 manual 的簡稱，是 Linux 的文件管理指令，即操作手冊。它通常用來查看文件的內容、編輯配置文件、查找指令說明、參數等。`man` 指令也可以查看各種函數庫，查找函數的使用方法。

在 Linux 上有許多指令，但我們不太可能將所有指令的所有用法全部背下來。因此，當遇到不熟悉的指令或是指令沒見過的使用方式時，我們就能夠對該指令下 `man`，來查看該指令的說明文件。在課程中，我們鼓勵學員遇到不熟悉的指令時，先嘗試使用 `man` 去了解指令再查看解答。透過這種方式來加強對指令的熟悉度。

### cat

cat (concatenate) 指令是 Linux 系統上用來輸出檔案內容的指令。

`cat` 指令的基礎用法是 `cat <file_name>`

```shell
kiwis@ubuntu:~$ cat TestFile
Hello World!
```

上面的範例中，我們使用 `cat` 指令顯示 TestFile 的內容，它只有一行文字，即 "Hello World!"。

使用 `cat -n` ，則它會在輸出的每一行前加上行號。

```shell
kiwis@ubuntu:~$ cat -n TestFile
     1  Hello World!
```

### cd

cd (change directory) 指令是 Linux 中用來切換當前所在的工作目錄，當我們需要進入到特定的目錄進行操作時，可以使用 `cd` 指令來改變當前目錄。

`cd` 指令的基礎用法是 `cd <directory_name>`

```shell
kiwis@ubuntu:~$ cd Public/
kiwis@ubuntu:~/Public$ pwd
/home/kiwis/Public

kiwis@ubuntu:~/Public$ cd ..
kiwis@ubuntu:~$ pwd
/home/kiwis
```

上面的範例中，我們使用 `cd` 指令將目前的工作目錄切換到 Public，並且使用 `pwd` 指令輸出我們所在的工作目錄。可以看到 `pwd` 來到了 `/home/kiwis/Public` 下。我們再次使用 `cd` 指令回到上一層目錄，Public 的父目錄。並且使用 `pwd` 指令輸出出我們所在的工作目錄，可以看到 `pwd` 成功回到 `/home/kiwis` 下。

### ls

ls (list) 指令是 Linux 中用來列出當前目錄下的檔案及子目錄，若未指定目錄，則預設列出當前工作目錄下的檔案及子目錄。

```shell
kiwis@ubuntu:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  snap  Templates  Videos

kiwis@ubuntu:~$ ls /
bin    dev   lib    libx32      mnt   root  snap      sys  var
boot   etc   lib32  lost+found  opt   run   srv       tmp
cdrom  home  lib64  media       proc  sbin  swapfile  usr
```

上面的範例中，我們可以看到 `ls` 指令列出了當前工作目錄下的所有檔案及子目錄。我們也能透過設定參數讓指令為我們做額外的工作。

使用 `ls -a`，可以列出所有檔案及目錄，包含被隱藏的目錄及檔案。在 Linux 中，隱藏的檔案其檔名前面會有 `.` 以表示。

```shell
kiwis@ubuntu:~$ ls -a
.              Desktop      Music            .sudo_as_admin_successful
..             Documents    Pictures         Templates
.bash_history  Downloads    .pki             Videos
.bash_logout   .gnupg       .profile         .vscode
.bashrc        .keras       Public           .wget-hsts
.cache         .local       .python_history
.config        .mongorc.js  snap
.dbshell       .mozilla     .ssh
```

使用 ls -l，可以以詳細資訊的形式列出檔案及目錄，包含權限、擁有者、日期等資訊。

```shell
kiwis@ubuntu:~$ ls -l
total 36
drwxr-xr-x 10 kiwis kiwis 4096 Dec 27 06:40 Desktop
drwxr-xr-x  2 kiwis kiwis 4096 Jul  7  2022 Documents
drwxr-xr-x  2 kiwis kiwis 4096 Dec 27 06:30 Downloads
drwxr-xr-x  2 kiwis kiwis 4096 Jul  7  2022 Music
drwxr-xr-x  2 kiwis kiwis 4096 Jul  7  2022 Pictures
drwxr-xr-x  2 kiwis kiwis 4096 Jul  7  2022 Public
drwx------  3 kiwis kiwis 4096 Sep  6 02:36 snap
drwxr-xr-x  2 kiwis kiwis 4096 Jul  7  2022 Templates
drwxr-xr-x  2 kiwis kiwis 4096 Jul  7  2022 Videos
```

使用 `ls -k`，可以使用人類可讀性較高的方式顯示檔案大小，例如 1K、2M。

```shell
kiwis@ubuntu:~$ ls -lh
total 48K
drwxr-xr-x 10 kiwis kiwis 4.0K Dec 27 06:40 Desktop
drwxr-xr-x  2 kiwis kiwis 4.0K Jul  7  2022 Documents
drwxr-xr-x  2 kiwis kiwis 4.0K Dec 27 06:30 Downloads
drwxr-xr-x  2 kiwis kiwis 4.0K Jul  7  2022 Music
drwxr-xr-x  2 kiwis kiwis 4.0K Jul  7  2022 Pictures
drwxr-xr-x  2 kiwis kiwis 4.0K Jul  7  2022 Public
drwx------  3 kiwis kiwis 4.0K Sep  6 02:36 snap
drwxr-xr-x  2 kiwis kiwis 4.0K Jul  7  2022 Templates
drwxr-xr-x  2 kiwis kiwis 4.0K Jul  7  2022 Videos
```

你可能注意到了，參數是能夠合併使用的，例如我們將 `ls -a` 與 `ls -l` 合併使用，下達 `ls -al`，該指令則會列出目錄下所有檔案及目錄以及其詳細資料。

```shell
kiwis@ubuntu:~$ ls -la
total 112
drwxr-xr-x 20 kiwis kiwis 4096 Jan 15 04:31 .
drwxr-xr-x  3 root  root  4096 Jul  7  2022 ..
-rw-------  1 kiwis kiwis 4154 Dec 27 02:45 .bash_history
-rw-r--r--  1 kiwis kiwis  220 Jul  7  2022 .bash_logout
-rw-r--r--  1 kiwis kiwis 3771 Jul  7  2022 .bashrc
drwxrwxr-x 17 kiwis kiwis 4096 Dec 27 06:43 .cache
drwx------ 16 kiwis kiwis 4096 Dec 27 06:39 .config
-rw-------  1 kiwis kiwis  335 Jul 11  2022 .dbshell
drwxr-xr-x 10 kiwis kiwis 4096 Dec 27 06:40 Desktop
drwxr-xr-x  2 kiwis kiwis 4096 Jul  7  2022 Documents
drwxr-xr-x  2 kiwis kiwis 4096 Dec 27 06:30 Downloads
```

| 參數 | 說明                                       |
| ---- | ------------------------------------------ |
| -a   | 輸出所有目錄及檔案，包含被隱藏的目錄及檔案 |
| -l   | 輸出目錄及檔案的詳細資訊                   |
| -h   | 以人類可讀的方式顯示檔案大小               |

需要特別注意的是，隨著指令不同，相同的參數可能具有不同的意義，並非所有參數接對於不同的指令都會有相同的功能。若要詳細了解指令以及參數的意義，可以使用 `man` 指令查看指令的說明文件。

### pwd

pwd (print working directory) 指令是 Linux 中用來輸出當前所在目錄的絕對路徑。當我們在指令介面工作時，往往會進入不同的目錄進行操作，使用 `pwd` 指令可以讓我們隨時了解當前所在的目錄位置。如果不小心在 Linux 中迷路了，也能夠透過 `pwd` 確認當前在哪個目錄底下。

`pwd` 指令的基礎用法是 `pwd <directory_name>`

```shell
kiwis@ubuntu:~$ pwd
/home/kiwis

kiwis@ubuntu:~$ cd Downloads/

kiwis@ubuntu:~/Downloads$ pwd
/home/kiwis/Downloads
```

### mkdir

mkdir (make directory) 指令是 Linux 中用來建立新目錄的指令。當我們需要在特定位置建立新的目錄時，可以使用 `mkdir` 指令來建立。

`mkdir` 指令的基礎用法是 `mkdir <directory_name>`

```shell
kiwis@ubuntu:~$ mkdir KiwisDir
kiwis@ubuntu:~$ ls
Desktop    Downloads  Pictures  snap       KiwisDir
Documents  Music      Public    Templates  Videos
```

上面的範例中，我們使用 `mkdir` 指令建立一個名為 KiwisDir 的目錄。建立完成後，我們使用 ls 指令來查看當前目錄中的所有項目，可以看到新建的 KiwisDir 目錄。

使用 `mkdir -p`，可以建立多層目錄，在需要一次性建立多層目錄時非常有用。

```shell
kiwis@ubuntu:~$ mkdir -p KiwisDir/KiwisDir2
kiwis@ubuntu:~$ ls
Desktop  Documents  Downloads  Music  KiwisDir  Pictures  Public  snap  Templates  Videos
kiwis@ubuntu:~$ ls KiwisDir/
KiwisDir2
```

上面的範例中，我們使用了 `-p` 選項來建立了 KiwisDir 和其子目錄 KiwisDir2。使用 `ls` 命令可以看到，新建立的 KiwisDir 目錄中還有一個子目錄 KiwisDir2。

### mv

mv (move) 指令是 Linux 系統中用來移動或重命名檔案或目錄的指令。使用 `mv` 指令可以快速地將檔案或目錄移動到另一個位置，或者改變它們的名稱。

`mv` 指令的基礎用法是 `mv <source_directory> <destination_directory>`

`<source_directory>` 代表要移動或重命名的檔案或目錄的路徑，`<destination_directory>` 代表檔案或目錄移動或重命名後的目標路徑或名稱。

```shell
kiwis@ubuntu:~$ mv KiwisDir/ KiwisDir2/
kiwis@ubuntu:~$ ls
Desktop    Downloads  Pictures  snap       KiwisDir2
Documents  Music      Public    Templates  Videos

kiwis@ubuntu:~$ mv KiwisDir2 Public/KiwisDir2
kiwis@ubuntu:~$ ls Public/
KiwisDir2
kiwis@ubuntu:~$ ls
Desktop  Documents  Downloads  Music  Pictures  Public  snap  Templates  Videos
```

上面的範例中，我們使用 `mv` 將工作目錄底下的 KiwisDir 目錄重命名為 KiwisDir2，並且使用 `ls` 輸出出來。可以看到 KiwisDir 已經被我們重新命名為 KiwisDir2 了。接著我們將 KiwisDir2 移動到 `Public/KiwisDir2`，並且同樣使用 `ls` 指令來查看 Public 目錄以及當前目錄。可以發現當前目錄下的 KiwisDir2 已經消失，取而代之的是 Public 目錄下出現了 KiwisDir2 的子目錄。

需要注意，如果 `<destination_directory>` 已經存在，`mv` 指令會直接覆蓋它，而不會二次確認是否要覆蓋。因此，使用 `mv` 指令時要小心謹慎，以免意外刪除檔案或目錄。

### rm

rm (remove) 指令是 Linux 系統中用來刪除檔案或目錄的指令。可以透過使用 `rm <directory_name>` 這個指令來刪除檔案或目錄。

`rm` 指令的基礎用法是 `rm <directory_name>`

```shell
kiwis@ubuntu:~$ ls
file1.txt file2.txt file3.txt
kiwis@ubuntu:~$ rm file3.txt
kiwis@ubuntu:~$ ls
file1.txt file2.txt
```

上面的範例中，我們使用 `rm` 指令刪除 file3.txt。再使用 `ls` 指令確認該文件已被刪除。

使用 `rm -r` ，則它會連同所有子目錄和檔案一起刪除。

```shell
kiwis@ubuntu:~$ rm -r Public/KiwisDir2/
kiwis@ubuntu:~$ ls Public/
```

需要注意，使用 `rm` 指令刪除檔案後沒有任何辦法恢復被刪除的檔案。因此，使用 `mv` 指令時務必要小心謹慎。

### touch

touch 指令是 Linux 系統中用來更改檔案或目錄的時間戳的指令，但通常我們更傾向使用它的另一個功能，建立空檔案。

`touch` 指令的基礎用法是 `touch <file_name>`

```shell
kiwis@ubuntu:~$ touch TestFile
kiwis@ubuntu:~$ ls
Desktop    Downloads  Pictures  snap       TestFile
Documents  Music      Public    Templates  Videos
kiwis@ubuntu:~$
```

上面的範例中，我們使用 `touch` 指令建立了一個名為 TestFile 的檔案。接著使用 `ls` 指令查看當前目錄下的所有文件，可以看到 TestFile 文件已經建立成功了。

`touch` 指令也可以用來更新文件的時間戳。當我們需要更改文件的時間戳以便進行後續操作時，可以使用 `touch` 指令。

```shell
kiwis@ubuntu:~$ ls -l TestFile
-rw-rw-r-- 1 kiwis kiwis 0 Apr 21 14:18 TestFile

kiwis@ubuntu:~$ touch TestFile
kiwis@ubuntu:~$ ls -l TestFile
-rw-rw-r-- 1 kiwis kiwis 0 Apr 21 14:20 TestFile
```

上面的範例中，我們使用 `ls -l` 指令查看 TestFile 的詳細資訊。可以看到檔案的時間戳是 14:18。接著使用 `touch` 指令更新檔案的時間戳，再次使用 `ls -l` 指令查看 TestFile 檔案的詳細資訊，可以看到時間戳已經更新為 14:20。

### echo

echo 指令是 Linux 系統上用來輸出字串的指令。

`echo` 指令的基礎用法是 `echo <some_string>`

```shell
kiwis@ubuntu:~$ echo 'Hello World!'
Hello World!
```

我們很少會單獨使用 `echo` 指令，我們通常會搭配運算子 (operators) 來使用，例如以下範例中，大於（`>`）這個運算子代表著重定向。代表我們能夠將前一個指令的輸出重定向到其他地方。因此下面的指令將會執行把 Hello World! 這個字串寫入 TestFile 檔案中的操作。我們將在下一個章節更詳細的介紹運算子相關的內容。

```shell
kiwis@ubuntu:~$ echo 'Hello World!' > TestFile
```

### cp

cp 是 Linux 系統上將檔案或目錄複製到指定位置的指令。通常使用 `cp` 指令來備份資料、將檔案複製到另一個位置。

`cp` 指令的基礎用法是 `cp <source_directory> <destination_directory>`

`cp` 指令的常用參數有以下三個
使用 `cp -i` 會在目標檔案已經存在時詢問使用者是否要覆蓋它。
使用 `cp -r` 會遞迴複製所有目錄和檔案。
使用 `cp -a` 會複製檔案的所有屬性，如檔案權限、擁有者、時間戳等。

```shell
kiwis@ubuntu:~$ cp -i file1.txt /home/kiwis/Public/
kiwis@ubuntu:~$ cp -r dir1/ /home/kiwis/Public/
kiwis@ubuntu:~$ cp -a file1.txt /home/kiwis/Public/
```

| 參數 | 說明                     |
| ---- | ------------------------ |
| -i   | 目標存在時，詢問是否覆蓋 |
| -r   | 複製目錄及其子目錄和內容 |
| -n   | 複製檔案的所有屬性       |

### file

file 是 Linux 系統上用來確定檔案格式、推測檔案類型的指令。它可以從檔案的內容推斷出其類型和格式，也可以在檔案中查找特定字串以確定檔案的類型。`file` 指令會對檔案進行兩種檢測，第一種是依靠檔案副檔名來推斷檔案類型，第二種是依據檔案內容來判斷檔案類型。使用 `file` 指令可以快速確定檔案類型，以便選擇正確的工具進行編輯、查看。

`file` 指令的基礎用法是 `file <file_name>`

```shell
kiwis@ubuntu:~$ file TestFile
TestFile: ASCII text

kiwis@ubuntu:~$ file image.jpg
image.jpg: JPEG image data, JFIF standard 1.01, resolution (DPI), density 72x72, segment length 16, Exif Standard: [TIFF image data, big-endian, direntries=7, description=...], baseline, precision 8, 1620x1080, components 3

kiwis@ubuntu:~$ file archive.tar.gz
archive.tar.gz: gzip compressed data, last modified: Sun Oct 17 12:53:55 2021, from Unix

kiwis@ubuntu:~$ file program
program: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0
```

上面的範例中，我們使用 `file` 指令來查看 TestFile 的類型。可以看到 TestFile 是一個 ASCII text，這表示它是一個文字文件。`file` 指令也可以分辨許多其他文件類型，例如圖片、壓縮檔、可執行檔等等。

### grep

grep （globally search for RE and print it）是 Linux 系統上能夠用來搜尋檔案文本中滿足指定條件內容的指令。它可以在龐大的檔案或資料流中快速找到所需的內容。`grep` 指令可以根據正則表達式 (Regular expression)、文本字串等來搜尋內容。

`grep` 指令的基礎用法是 `grep <search_pattern> <file_name>`

`<search_pattern>` 是我們要搜索的模式，可以是一個字、一句話或是正則表達式，`<file_name>` 是要搜尋的檔案名稱。

```shell
kiwis@ubuntu:~$ grep 'Hello World!' TestFile
Here is a example, show how to find "Hello World!".
```

上面的範例中，`grep` 在 TestFile 文件中搜索 'Hello World!' 的模式，找到了包含 'Hello World!' 的字串，將結果輸出。

### find

find 是 Linux 系統上用來尋找檔案或目錄的指令。它可以用來搜尋特定目錄下特定檔案名的檔案，透過修改日期、大小、擁有者等屬性尋找。`find` 有非常多的參數，如：`-name`，`-mtime`，`-user` 等。

`find` 指令的基礎用法是 `find <find_directory> <search_expression>`

`<find_directory>` 是要搜尋的目錄，`<search_expression>` 是搜尋的條件表達式。

```shell
kiwis@ubuntu:~$ find ./ -name TestFile
./TestFile
```

上面的範例中，我們使用 find 指令，在當前目錄下搜尋名稱為 TestFile 的檔案。

此外，我們也能搭配萬用字元 `*` 進行搜尋。

```shell
kiwis@ubuntu:~$ find ./ -name *.txt
TestFile.txt
```

上面的範例中，我們想要找到當前目錄中的 txt 檔案，我們透過 `*` 表示匹配任何任意長度字元。

`find` 指令還有許多其他參數及用法

| 參數   | 說明                   |
| ------ | ---------------------- |
| -name  | 搜尋檔案或資料夾的名稱 |
| -size  | 搜尋檔案大小           |
| -user  | 搜尋屬於某用戶的檔案   |
| -type  | 搜尋檔案類型           |
| -mtime | 設定搜尋的時間長度     |
| -iname | 忽略檔案名稱的大小寫   |

### sudo

sudo 是 Linux 系統上非常重要的指令。它的功能是在使用者權限不足時，暫時提升使用者權限，以執行需要 root 權限才能執行的指令。

`sudo` 指令的基礎用法是 `sudo <command>`

```shell
kiwis@ubuntu:~$ sudo apt-get update
kiwis@ubuntu:~$ sudo apt-get install <package_name>
```

上面的範例中，我們使用 `apt-get` 指令來安裝軟體時，使用 `sudo` 指令以 root 權限執行安裝。

需要注意，使用 `sudo` 指令要非常謹慎，以避免不必要的風險，因為一旦使用 `sudo` 指令，就可以在系統上執行需要 root 權限的任何指令。

### clear

clear 是 Linux 系統清除指令介面上的所有內容的指令。使用指令介面時，有時候畫面會變得非常雜亂，這時可以使用 `clear` 指令來清除。

```shell
kiwis@ubuntu:~$ clear
```

上面的範例中，輸入完指令後，指令介面上的所有內容都會被清除。`clear` 指令只會清除當前指令介面上的內容，並不會影響歷史指令紀錄，所以還是可以使用方向鍵來查看之前輸入的指令。
