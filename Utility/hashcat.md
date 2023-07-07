# hashcat

[TOC]

source code: https://github.com/hashcat/hashcat

hash types: https://hashcat.net/wiki/doku.php?id=example_hashes

rule: https://hashcat.net/wiki/doku.php?id=rule_based_attack
https://blog.csdn.net/wn314/article/details/76697019


`hashcat --example-hashes`: 可以查看有多少种hash类型
`hashcat -I`: 显示 system/evironment/backend API 这些的信息

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

攻击模式： --attack-mode
| 序号 |        模式        |       注释        |
| :--- | :----------------- | :---------------- |
| 0    | Straight           | （字典破解）      |
| 1    | Combination        | （组合破解）      |
| 3    | Brute-force        | （掩码暴力破解）  |
| 6    | Hybrid dict + mask | （混合字典+掩码） |
| 7    | Hybrid mask + dict | （混合掩码+字典） |

Straight: 是给定一个字典，hashcat会逐行读取字典中的内容，计算每行的hash值，与目标hash值相比较。[^hashcat-attack-modes]
Combination: 组合两个密码字典的内容。使用这一攻击模式需要不多不少地指定两个密码字典。
```sh
echo -n abc123 | md5sum | awk '{print $1}' > hashes.txt
cat << EOF > wordlist-alpha.txt
a
abc
EOF

cat << EOF > wordlist-number.txt
1
12
123
EOF

hashcat -m 0 -a 1 hashes.txt wordlist-alpha.txt wordlist-number.txt
```
上面的尝试的字典是 wordlist-alpha.txt左边的2个 以及wordlist-number.txt右边的3个：
```
a1
a12
a123
abc1
abc12
abc123
```

上面的字典可以通过下面的命令生成
```sh
hashcat -m 0 -a 1 wordlist-alpha.txt wordlist-number.txt --stdout
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
echo -n Abc123 | md5sum | awk '{print $1}' > hashes.txt
hashcat -m 0 -a 3 hashes.txt ?u?l?l?d?d?d
```

`Hybrid dict + mask`: 将一个字典和一个mask进行组合破解
```sh
echo -n abc123 | md5sum | awk '{print $1}' > hashes.txt
cat << EOF > wordlist-alpha.txt
a
ab
abc
EOF

hashcat -m 0 -a 6 hashes.txt wordlist-alpha.txt ?d?d?d
```

它尝试破解的组合如下
```
a00
a01
a02
...
abc99
```

`Hybrid mask + dict`: 这个和上面的交换了位置
```sh
echo -n 123abc | md5sum | awk '{print $1}' > hashes.txt
cat << EOF > wordlist-alpha.txt
a
ab
abc
EOF

hashcat -m 0 -a 7 hashes.txt ?d?d?d wordlist-alpha.txt
```

Workload profile
|  #   | Performance | Runtime | Power Consumption | Desktop Impact |
| :--- | :---------- | :------ | :---------------- | :------------- |
| 1    | Low         | 2 ms    | Low               | Minimal        |
| 2    | Default     | 12 ms   | Economic          | Noticeable     |
| 3    | High        | 96 ms   | High              | Unresponsive   |
| 4    | Nightmare   | 480 ms  | Insane            | Headless       |


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


[^hashcat-attack-modes]: https://blog.werner.wiki/hashcat-attack-modes/


