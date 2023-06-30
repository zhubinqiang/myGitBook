# hashcat

[TOC]

source code: https://github.com/hashcat/hashcat

hash types: https://hashcat.net/wiki/doku.php?id=example_hashes

rule: https://hashcat.net/wiki/doku.php?id=rule_based_attack

攻击模式： --attack-mode
| 序号 |        模式        |       注释        |
| :--- | :----------------- | :---------------- |
| 0    | Straight           | （字典破解）      |
| 1    | Combination        | （组合破解）      |
| 3    | Brute-force        | （掩码暴力破解）  |
| 6    | Hybrid dict + mask | （混合字典+掩码） |
| 7    | Hybrid mask + dict | （混合掩码+字典） |


`hashcat --example-hashes`: 可以查看有多少种hash类型

the cracked passwords are stored in ~/.local/share/hashcat/hashcat.potfile

```sh
hashcat -m 0 -a 0 -O -w 3 hashes.txt 10-million-password-list-top-100000.txt
```
-m: --hash-type 0为md5
-a: --attack-mode 0为straight 字典破解
-w: Workload profile 3 = High
-O: Optimised kernel - speeds up attack on supported modes
hashes.txt: Text file containing hashes we’re trying to crack
10_million_pass...: Wordlist we’re using for the attack, which is from https://github.com/danielmiessler/SecLists

```sh
hashcat -m 0 -a 0 -O -w 3 hashes.txt 10-million-password-list-top-100000.txt -r rules/best64.rule
```


brute-force

| 符号 |        含义        |
| :--- | :----------------- |
| ?d   | Digit              |
| ?l   | Lowercase Letter   |
| ?u   | Uppercase Letter   |
| ?s   | Special Character  |
| ?a   | All Character Sets |

```sh
hashcat -m 0 -a 3 hashes.txt ?l?l?l?d?d?d
```


### MS office wold
https://github.com/openwall/john/blob/bleeding-jumbo/run/office2john.py

```sh
python office2john.py 'test.docx'

test.docx:$office$*2013*100000*256*16*d5f728f5b3a2a346d05e1e59544d9807*b4826abb7c5be17bdf4ad7066a472d9b*157e9d7872948027ed299d1bb09c2d3d03819635b87a2b7fe5e44c0519189d26
```

```sh
hashcat -m 9600 -a 3 '$office$*2013*100000*256*16*d5f728f5b3a2a346d05e1e59544d9807*b4826abb7c5be17bdf4ad7066a472d9b*157e9d7872948027ed299d1bb09c2d3d03819635b87a2b7fe5e44c0519189d26' ?d?d?d?d?d
```

### 7-zip
```sh
sudo apt install -y p7zip p7zip-full
```

```sh
7z a -ppass123 test.7z 7z2john.pl

7z x -y -ppass123 test.7z -o./now

./7z2john.pl ../test.7z
```

7z2john.pl is from https://raw.githubusercontent.com/openwall/john/bleeding-jumbo/run/7z2john.pl



