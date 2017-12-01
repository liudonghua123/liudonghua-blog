---
title: RSA中的public/private加解密
tags:
  - openssl
  - rsa
  - rsautl
  - ssh-keygen
id: 251
categories:
  - 学习
date: 2014-10-20 21:13:44
---

今天在另外一台电脑上想连接我的EC2服务器，由于没有pem文件（私钥private key），重新生成了一个，只是最终还是不能连接上，详见[Amazon EC2 Key Pairs](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)，原因是由于没有服务器上的公钥的对应私钥，所以也就没办法连接上EC2服务器上传另一个公钥，但是还是有其他办法，见上述文档的[Connecting to Your Linux Instance if You Lose Your Private Key](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html#replacing-lost-key-pair)。<!--more-->

所以就看了一下RSA的公私钥加解密这块的内容，以下是在ubuntu下测试的，详见注释即可理解

[shell highlight=""]
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$ # generate plain test file
liudonghua@www:~/rsa_tutorial$ cat &gt; test.txt &lt;&lt; DELIMITER
&gt; This is a ras public/private encrypt/decrypt sample!!!
&gt; DELIMITER
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$ # produce a 1024 length private key named openssl_1024.pri
liudonghua@www:~/rsa_tutorial$ openssl genrsa -out openssl_1024.pri 1024
Generating RSA private key, 1024 bit long modulus
............++++++
....................++++++
e is 65537 (0x10001)
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$ # produce the corresponding public key named openssl_1024.pub
liudonghua@www:~/rsa_tutorial$ openssl rsa -in openssl_1024.pri -pubout -out openssl_1024.pub
writing RSA key
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$ cat openssl_1024.pri
-----BEGIN RSA PRIVATE KEY-----
MIICWwIBAAKBgQC276RD6IMWofi7wupJN4UWvOJ72xQFz/47o9g8QqfMvowa8X/2
QBf0MUuAGkh2TueSPm8hAe/sbCLQajReUaFgNI7zH3Arb72utxu+SezgBXQK+l9S
okwKM/JXKegj32SwmoyzG4aakRq16JIYqeTyKsbhGZsNNvQSa/QVytSOxwIDAQAB
AoGAYLHgwOhYyhDJWe3YSuUm2vLyQAd32O6s8jdTp96PtYCOq/s06SPNxYx83PSH
ksl4S+vmb6sHd49dA47vqV86jasmsXgXMfNC5+yHbSxamDKe8VM2f5Db7f/R7yVk
4NsS8qlmtVLql1y+2hrqM+A1uX3eOJL18AqXkK5CjJetGxECQQDpsWVcDTsB5rcN
zzUYbYDbNY4hzL4+nhF+KXaDuCRPgxIutiFQWTCqOEN1ZudFp2iLTFKcQczO9MZe
gpT348H5AkEAyGXu6FMaF0ukCCm1aOzHJFafxTggagJcJygP5iI8cSHzlP+KzEs+
0pUYnD2UvafvVA7BDwSsCff8fBfr8vMGvwJAEpXFFdkHhFMw46xC8LpksQpFT3LU
/m3bvkjV4AvY92nZHFXnuFgfgqoO01tnsSZrLgjX2Q1ymFLnI8UGy+AVIQJAL7ID
EIxm01CPc9npcVWZeA6d7CSVomV5ZWBlmFJhrFN2U+oWMNVf2GLf/p+xfQoxLgJs
9JQaFi1NjINtBt/MpQJAbS7eNnB3gEL/K4XrtVs/1wS58bxoRy7SpBgzhtC0NpsM
dA5OobN9yI4XsfIBRSU5Bez3VX8scaRpIJ3KYEPxag==
-----END RSA PRIVATE KEY-----
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$ cat openssl_1024.pub
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC276RD6IMWofi7wupJN4UWvOJ7
2xQFz/47o9g8QqfMvowa8X/2QBf0MUuAGkh2TueSPm8hAe/sbCLQajReUaFgNI7z
H3Arb72utxu+SezgBXQK+l9SokwKM/JXKegj32SwmoyzG4aakRq16JIYqeTyKsbh
GZsNNvQSa/QVytSOxwIDAQAB
-----END PUBLIC KEY-----
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$ # use rsautl to encrypt/decrypt, see http://www.tutorialspoint.com/unix_commands/rsautl.htm
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$ openssl rsautl -encrypt -pubin -inkey openssl_1024.pub -in test.txt -out encrypted_test.txt
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$ openssl rsautl -decrypt -inkey openssl_1024.pri -in encrypted_test.txt -out decrypted_test.txt
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$ cat decrypted_test.txt
This is a ras public/private encrypt/decrypt sample!!!
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$ # use ssh-keygen to generate public/private key together interactively
liudonghua@www:~/rsa_tutorial$ # see https://help.github.com/articles/generating-ssh-keys/
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$ ssh-keygen -t rsa -b 1024
Generating public/private rsa key pair.
Enter file in which to save the key (/home/liudonghua/.ssh/id_rsa): ssh_keygen_1024
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in ssh_keygen_1024.
Your public key has been saved in ssh_keygen_1024.pub.
The key fingerprint is:
9b:ed:1f:0b:e4:c5:66:e3:49:49:e4:19:0a:45:1e:8a liudonghua@www.liudonghua.com
The key's randomart image is:
+--[ RSA 1024]----+
|        .o+ o    |
|       . + = o   |
|      E . o +    |
|           o .   |
|        S . O    |
|         * * o   |
|        o + +    |
|         . . o   |
|          ..o    |
+-----------------+
liudonghua@www:~/rsa_tutorial$ ll ssh_keygen_1024*
-rw------- 1 liudonghua liudonghua 1679 Oct 20 20:32 ssh_keygen_1024
-rw-r--r-- 1 liudonghua liudonghua  411 Oct 20 20:32 ssh_keygen_1024.pub
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$ openssl rsautl -encrypt -pubin -inkey ssh_keygen_1024.pub -in test.txt -out encrypted_ssh_keygen_test.txt
unable to load Public Key
liudonghua@www:~/rsa_tutorial$ cat ssh_keygen_1024.pub
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAAgQDcOO3UWGXnxIj3iKe/anB2NgxNJG89OaoL4hBxrY6T+kO2Rm8GNqfHP8d8N6O/WssI3t1oxNiB/EtBgz9qxmgLdu6CABd93hZrPHYZjLNOZlswzydLoYygHuiVbSWGoxcS07nTEi2F4YvA964LfqSS3ZxXQUJJx4N6FP3KJf6r+Q== liudonghua@www.liudonghua.com
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$ cat ssh_keygen_1024
-----BEGIN RSA PRIVATE KEY-----
MIICXAIBAAKBgQDcOO3UWGXnxIj3iKe/anB2NgxNJG89OaoL4hBxrY6T+kO2Rm8G
NqfHP8d8N6O/WssI3t1oxNiB/EtBgz9qxmgLdu6CABd93hZrPHYZjLNOZlswzydL
oYygHuiVbSWGoxcS07nTEi2F4YvA964LfqSS3ZxXQUJJx4N6FP3KJf6r+QIDAQAB
AoGAUCHq8HSGCDKsgB3apD0v1OPP0BYs4ANmi9Jjl2nG0rOjBeVVKtaicE8V4G5C
iDOaY4zs1d2ixpGuNJV0cv7lBnIM4OPwJoaaskhTyeomkfLRyHAkmj4brrrEPrgy
XMWmbWydFhsSR2liirU3SasV5KL2VmqN+eHzboD9MYoMBq0CQQD0ysA2n5pkxjEW
EElY6uXiS2p8oD/iYaycyroKSUfpwFMnZPFAOuqQxhCPFu+O/FsTTtq1cDJK37X5
shzvOkgvAkEA5k4xh9gc8QdkBEJihbq7Dcu+W5OnzlkXqMYlXiWQ7Crc4ginuFRg
eCNqkY4zQnCwL5wxNVhfWTMjoj/1faUcVwJAflea51Zu2G5WbV3QjX0HU0m7V0Fa
V5wz++TCKobx/9pM0LtPrOf7oucnKsMat4DV/NwpE5Ypzu0xvgNI3cwF7wJAR4ez
xJBv9MCp5NTFiumDXXaRihnjPajYO1hHlOUwDNoHPsEXbp3uVIITgF/dNd6QKkll
0z6+ZpMGl0csNTkKAwJBAIV9sS+ggZ3P5WW+uytrSjifsYibzeGLOSP9v+QpA36w
S6+UqzksZ5L+Xt594HaGA8oqC6vAv1AGGIHwQ1+U7Rw=
-----END RSA PRIVATE KEY-----
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$  # compared to the openssl_1024.pub, the structure is different
liudonghua@www:~/rsa_tutorial$  # so we can't use key generated by ssh_keygen directly
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$
[/shell]

附上
通过rsa public/private key免用户名密码登录远程linux终端
[su_accordion]
[su_spoiler title="how-can-i-set-up-password-less-ssh-login" style="fancy"]

1.ssh-keygen
Press Enter key till you get the prompt

2.ssh-copy-id -i root@ip_address
(It will once ask for the password of the host system)
or
cat ~/.ssh/id_rsa.pub | ssh username@hostname 'cat &gt;&gt; .ssh/authorized_keys'
(This requires the folder .ssh to be in the home directory on the targeted hostname, with the authorized_keys file in it)

3.ssh root@ip_address
Now you should be able to login without any password.
[/su_spoiler]
[/su_accordion]

检测public/private key是否匹配
[su_accordion]
[su_spoiler title="check public/private key matches" style="fancy"]

[shell]
liudonghua@www:~/rsa_tutorial$ # test public/private key pair matches
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$ ssh-keygen -y -f ssh_keygen_1024 &gt; public_key_of_ssh_keygen_1024.pub
liudonghua@www:~/rsa_tutorial$ diff public_key_of_ssh_keygen_1024.pub ssh_keygen_1024.pub
1c1
&lt; ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAAgQDcOO3UWGXnxIj3iKe/anB2NgxNJG89OaoL4hBxrY6T+kO2Rm8GNqfHP8d8N6O/WssI3t1oxNiB/EtBgz9qxmgLdu6CABd93hZrPHYZjLNOZlswzydLoYygHuiVbSWGoxcS07nTEi2F4YvA964LfqSS3ZxXQUJJx4N6FP3KJf6r+Q==
---
&gt; ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAAgQDcOO3UWGXnxIj3iKe/anB2NgxNJG89OaoL4hBxrY6T+kO2Rm8GNqfHP8d8N6O/WssI3t1oxNiB/EtBgz9qxmgLdu6CABd93hZrPHYZjLNOZlswzydLoYygHuiVbSWGoxcS07nTEi2F4YvA964LfqSS3ZxXQUJJx4N6FP3KJf6r+Q== liudonghua@www.liudonghua.com
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$ ssh-keygen -y -f openssl_1024.pri &gt; public_key_of_openssl_1024.pub
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$ diff public_key_of_openssl_1024.pub openssl_1024.pub
1c1,6
&lt; ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAAgQC276RD6IMWofi7wupJN4UWvOJ72xQFz/47o9g8QqfMvowa8X/2QBf0MUuAGkh2TueSPm8hAe/sbCLQajReUaFgNI7zH3Arb72utxu+SezgBXQK+l9SokwKM/JXKegj32SwmoyzG4aakRq16JIYqeTyKsbhGZsNNvQSa/QVytSOxw==
---
&gt; -----BEGIN PUBLIC KEY-----
&gt; MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC276RD6IMWofi7wupJN4UWvOJ7
&gt; 2xQFz/47o9g8QqfMvowa8X/2QBf0MUuAGkh2TueSPm8hAe/sbCLQajReUaFgNI7z
&gt; H3Arb72utxu+SezgBXQK+l9SokwKM/JXKegj32SwmoyzG4aakRq16JIYqeTyKsbh
&gt; GZsNNvQSa/QVytSOxwIDAQAB
&gt; -----END PUBLIC KEY-----
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$  # convert openssh public key to pem format
liudonghua@www:~/rsa_tutorial$ ssh-keygen -f public_key_of_openssl_1024.pub -e -m pem
-----BEGIN RSA PUBLIC KEY-----
MIGJAoGBALbvpEPogxah+LvC6kk3hRa84nvbFAXP/juj2DxCp8y+jBrxf/ZAF/Qx
S4AaSHZO55I+byEB7+xsItBqNF5RoWA0jvMfcCtvva63G75J7OAFdAr6X1KiTAoz
8lcp6CPfZLCajLMbhpqRGrXokhip5PIqxuEZmw029BJr9BXK1I7HAgMBAAE=
-----END RSA PUBLIC KEY-----
liudonghua@www:~/rsa_tutorial$
liudonghua@www:~/rsa_tutorial$ cat openssl_1024.pub
-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC276RD6IMWofi7wupJN4UWvOJ7
2xQFz/47o9g8QqfMvowa8X/2QBf0MUuAGkh2TueSPm8hAe/sbCLQajReUaFgNI7z
H3Arb72utxu+SezgBXQK+l9SokwKM/JXKegj32SwmoyzG4aakRq16JIYqeTyKsbh
GZsNNvQSa/QVytSOxwIDAQAB
-----END PUBLIC KEY-----
liudonghua@www:~/rsa_tutorial$
[/shell]

[/su_spoiler]
[/su_accordion]

ssh-keygen常用参数使用
[su_accordion]
[su_spoiler title="ssh-keygen usage" style="fancy"]

OpenSSH key pem和SSH2 key格式互转

ssh-keygen -e -f ~/.ssh/id_rsa.pub &gt; ~/.ssh/id_rsa_pub.pem
ssh-keygen -i -f ~/.ssh/id_rsa_pub.pem -m RFC4716 &gt; ~/.ssh/id_rsa.pub

# Import other format (public/private) key to openssh format key
# read an unencrypted private (or public) key file in the format specified by the -m option and print an OpenSSH compatible private (or public) key to stdout
ssh-keygen -i [-m key_format] [-f input_keyfile]

# Export openssh key (public/private) to other format
# read a private or public OpenSSH key file and print to stdout the key in one of the formats specified by the -m option. The default export format is "RFC4716''.
ssh-keygen -e [-m key_format] [-f input_keyfile]

-m key_format
Specify a key format for the -i (import) or -e (export) conversion options.
"RFC4716'' (RFC 4716/SSH2 public or private key)
"PKCS8'' (PEM PKCS8 public key)
"PEM'' (PEM public key).
The default conversion format is "RFC4716''.

# get openssh format public key from openssh format private key
ssh-keygen -y [-f input_keyfile]

-b bits
Specifies the number of bits in the key to create. For RSA keys, the minimum
size is 768 bits and the default is 2048 bits. Generally, 2048 bits is con-
sidered sufficient. DSA keys must be exactly 1024 bits as specified by FIPS
186-2\. For ECDSA keys, the -b flag determines the key length by selecting
from one of three elliptic curve sizes: 256, 384 or 521 bits. Attempting to
use bit lengths other than these three values for ECDSA keys will fail.
-C comment
Provides a new comment.
-c Requests changing the comment in the private and public key files. This
operation is only supported for RSA1 keys. The program will prompt for the
file containing the private keys, for the passphrase if the key has one, and
for the new comment.
-e This option will read a private or public OpenSSH key file and print to std-
out the key in one of the formats specified by the -m option. The default
export format is "RFC4716''. This option allows exporting OpenSSH keys for
use by other programs, including several commercial SSH implementations.
-f filename
Specifies the filename of the key file.
-i This option will read an unencrypted private (or public) key file in the for-
mat specified by the -m option and print an OpenSSH compatible private (or
public) key to stdout.
-m key_format
Specify a key format for the -i (import) or -e (export) conversion options.
The supported key formats are: "RFC4716'' (RFC 4716/SSH2 public or private
key), "PKCS8'' (PEM PKCS8 public key) or "PEM'' (PEM public key). The
default conversion format is "RFC4716''.
-N new_passphrase
Provides the new passphrase.
-v Verbose mode. Causes ssh-keygen to print debugging messages about its
progress. This is helpful for debugging moduli generation. Multiple -v
options increase the verbosity. The maximum is 3.
-y This option will read a private OpenSSH format file and print an OpenSSH public key to stdout.

[/su_spoiler]
[/su_accordion]

openssl rsautl常用参数使用
[su_accordion]
[su_spoiler title="rsautl - RSA utility " style="fancy"]
- Unix, Linux Command

&nbsp;
<table class="main" border="0" cellspacing="0" cellpadding="2">
<tbody>
<tr>
<td valign="top">
<table class="middle" cellspacing="0" cellpadding="5">
<tbody>
<tr>
<td>

# NAME

rsautl - RSA utility

# SYNOPSIS

<!-- ignored unsupported tag .tm --> **openssl** **rsautl** [**-in file**] [**-out file**] [**-inkey file**] [**-pubin**] [**-certin**] [**-sign**] [**-verify**] [**-encrypt**] [**-decrypt**] [**-pkcs**] [**-ssl**] [**-raw**] [**-hexdump**] [**-asn1parse**]

# DESCRIPTION

<!-- ignored unsupported tag .tm -->The **rsautl** command can be used to sign, verify, encrypt and decrypt data using theRSA algorithm.

# COMMAND OPTIONS

<!-- ignored unsupported tag .tm -->
<table class="src" border="1" cellspacing="0" cellpadding="5">
<tbody>
<tr>
<th width="25%">Tag</th>
<th>Description</th>
</tr>
<tr valign="top">
<td width="4%">**-in filename**</td>
<td><!-- ignored unsupported tag .tm -->This specifies the input filename to read data from or standard input if this option is not specified.</td>
</tr>
<tr valign="top">
<td width="4%">**-out filename**</td>
<td><!-- ignored unsupported tag .tm -->specifies the output filename to write to or standard output by default.</td>
</tr>
<tr valign="top">
<td width="4%">**-inkey file**</td>
<td><!-- ignored unsupported tag .tm -->the input key file, by default it should be an RSA private key.</td>
</tr>
<tr valign="top">
<td width="4%">**-pubin**</td>
<td><!-- ignored unsupported tag .tm -->the input file is an RSA public key.</td>
</tr>
<tr valign="top">
<td width="4%">**-certin**</td>
<td><!-- ignored unsupported tag .tm -->the input is a certificate containing an RSA public key.</td>
</tr>
<tr valign="top">
<td width="4%">**-sign**</td>
<td><!-- ignored unsupported tag .tm -->sign the input data and output the signed result. This requires and RSA private key.</td>
</tr>
<tr valign="top">
<td width="4%">**-verify**</td>
<td><!-- ignored unsupported tag .tm -->verify the input data and output the recovered data.</td>
</tr>
<tr valign="top">
<td width="4%">**-encrypt**</td>
<td><!-- ignored unsupported tag .tm -->encrypt the input data using an RSA public key.</td>
</tr>
<tr valign="top">
<td width="4%">**-decrypt**</td>
<td><!-- ignored unsupported tag .tm --> decrypt the input data using an RSA private key.</td>
</tr>
<tr valign="top">
<td width="4%">**-pkcs, -oaep, -ssl, -raw**</td>
<td><!-- ignored unsupported tag .tm -->the padding to use: PKCS#1 v1.5 (the default), PKCS#1 OAEP, special padding used in SSL v2 backwards compatible handshakes, or no padding, respectively. For signatures, only **-pkcs** and **-raw** can be used.</td>
</tr>
<tr valign="top">
<td width="4%">**-hexdump**</td>
<td><!-- ignored unsupported tag .tm -->hex dump the output data.</td>
</tr>
<tr valign="top">
<td width="4%">**-asn1parse**</td>
<td><!-- ignored unsupported tag .tm -->asn1parse the output data, this is useful when combined with the **-verify** option.</td>
</tr>
</tbody>
</table>

# NOTES

<!-- ignored unsupported tag .tm --> **rsautl** because it uses theRSA algorithm directly can only be used to sign or verify small pieces of data.

# EXAMPLES

<!-- ignored unsupported tag .tm -->Sign some data using a private key:
<table class="src" cellspacing="5" cellpadding="5">
<tbody>
<tr>
<td>
<pre><!-- ignored unsupported tag .ne -->
 openssl rsautl -sign -in file -inkey key.pem -out sig
</pre>
</td>
</tr>
</tbody>
</table>
Recover the signed data
<table class="src" cellspacing="5" cellpadding="5">
<tbody>
<tr>
<td>
<pre><!-- ignored unsupported tag .ne -->
 openssl rsautl -verify -in sig -inkey key.pem
</pre>
</td>
</tr>
</tbody>
</table>
Examine the raw signed data:
<table class="src" cellspacing="5" cellpadding="5">
<tbody>
<tr>
<td>
<pre><!-- ignored unsupported tag .ne -->
 openssl rsautl -verify -in file -inkey key.pem -raw -hexdump
</pre>
</td>
</tr>
</tbody>
</table>
<table class="src" cellspacing="5" cellpadding="5">
<tbody>
<tr>
<td>
<pre><!-- ignored unsupported tag .ne -->
 0000 - 00 01 ff ff ff ff ff ff-ff ff ff ff ff ff ff ff   ................
 0010 - ff ff ff ff ff ff ff ff-ff ff ff ff ff ff ff ff   ................
 0020 - ff ff ff ff ff ff ff ff-ff ff ff ff ff ff ff ff   ................
 0030 - ff ff ff ff ff ff ff ff-ff ff ff ff ff ff ff ff   ................
 0040 - ff ff ff ff ff ff ff ff-ff ff ff ff ff ff ff ff   ................
 0050 - ff ff ff ff ff ff ff ff-ff ff ff ff ff ff ff ff   ................
 0060 - ff ff ff ff ff ff ff ff-ff ff ff ff ff ff ff ff   ................
 0070 - ff ff ff ff 00 68 65 6c-6c 6f 20 77 6f 72 6c 64   .....hello world
</pre>
</td>
</tr>
</tbody>
</table>
The PKCS#1 block formatting is evident from this. If this was done using encrypt and decrypt the block would have been of type 2 (the second byte) and random padding data visible instead of the 0xff bytes.

It is possible to analyse the signature of certificates using this utility in conjunction with **asn1parse**. Consider the self signed example in certs/pca-cert.pem . Running **asn1parse** as follows yields:
<table class="src" cellspacing="5" cellpadding="5">
<tbody>
<tr>
<td>
<pre><!-- ignored unsupported tag .ne -->
 openssl asn1parse -in pca-cert.pem
</pre>
</td>
</tr>
</tbody>
</table>
<table class="src" cellspacing="5" cellpadding="5">
<tbody>
<tr>
<td>
<pre><!-- ignored unsupported tag .ne -->
    0:d=0  hl=4 l= 742 cons: SEQUENCE         
    4:d=1  hl=4 l= 591 cons:  SEQUENCE         
    8:d=2  hl=2 l=   3 cons:   cont [ 0 ]       
   10:d=3  hl=2 l=   1 prim:    INTEGER           :02
   13:d=2  hl=2 l=   1 prim:   INTEGER           :00
   16:d=2  hl=2 l=  13 cons:   SEQUENCE         
   18:d=3  hl=2 l=   9 prim:    OBJECT            :md5WithRSAEncryption
   29:d=3  hl=2 l=   0 prim:    NULL             
   31:d=2  hl=2 l=  92 cons:   SEQUENCE         
   33:d=3  hl=2 l=  11 cons:    SET              
   35:d=4  hl=2 l=   9 cons:     SEQUENCE         
   37:d=5  hl=2 l=   3 prim:      OBJECT            :countryName
   42:d=5  hl=2 l=   2 prim:      PRINTABLESTRING   :AU
  ....
  599:d=1  hl=2 l=  13 cons:  SEQUENCE         
  601:d=2  hl=2 l=   9 prim:   OBJECT            :md5WithRSAEncryption
  612:d=2  hl=2 l=   0 prim:   NULL             
  614:d=1  hl=3 l= 129 prim:  BIT STRING
</pre>
</td>
</tr>
</tbody>
</table>
The final BIT STRING contains the actual signature. It can be extracted with:
<table class="src" cellspacing="5" cellpadding="5">
<tbody>
<tr>
<td>
<pre><!-- ignored unsupported tag .ne -->
 openssl asn1parse -in pca-cert.pem -out sig -noout -strparse 614
</pre>
</td>
</tr>
</tbody>
</table>
The certificate public key can be extracted with:
<table class="src" cellspacing="5" cellpadding="5">
<tbody>
<tr>
<td>
<pre><!-- ignored unsupported tag .ne -->
 openssl x509 -in test/testx509.pem -pubkey -noout &gt;pubkey.pem
</pre>
</td>
</tr>
</tbody>
</table>
The signature can be analysed with:
<table class="src" cellspacing="5" cellpadding="5">
<tbody>
<tr>
<td>
<pre><!-- ignored unsupported tag .ne -->
 openssl rsautl -in sig -verify -asn1parse -inkey pubkey.pem -pubin
</pre>
</td>
</tr>
</tbody>
</table>
<table class="src" cellspacing="5" cellpadding="5">
<tbody>
<tr>
<td>
<pre><!-- ignored unsupported tag .ne -->
    0:d=0  hl=2 l=  32 cons: SEQUENCE         
    2:d=1  hl=2 l=  12 cons:  SEQUENCE         
    4:d=2  hl=2 l=   8 prim:   OBJECT            :md5
   14:d=2  hl=2 l=   0 prim:   NULL             
   16:d=1  hl=2 l=  16 prim:  OCTET STRING     
      0000 - f3 46 9e aa 1a 4a 73 c9-37 ea 93 00 48 25 08 b5   .F...Js.7...H%..
</pre>
</td>
</tr>
</tbody>
</table>
This is the parsed version of an ASN1 DigestInfo structure. It can be seen that the digest used was md5\. The actual part of the certificate that was signed can be extracted with:
<table class="src" cellspacing="5" cellpadding="5">
<tbody>
<tr>
<td>
<pre><!-- ignored unsupported tag .ne -->
 openssl asn1parse -in pca-cert.pem -out tbs -noout -strparse 4
</pre>
</td>
</tr>
</tbody>
</table>
and its digest computed with:
<table class="src" cellspacing="5" cellpadding="5">
<tbody>
<tr>
<td>
<pre><!-- ignored unsupported tag .ne -->
 openssl md5 -c tbs
 MD5(tbs)= f3:46:9e:aa:1a:4a:73:c9:37:ea:93:00:48:25:08:b5
</pre>
</td>
</tr>
</tbody>
</table>
which it can be seen agrees with the recovered value above.

# SEE ALSO

<tt><tt><tt><tt><tt><tt><tt><tt><tt><tt><tt><tt>_dgst_(1), _rsa_(1), _genrsa_(1)
</tt></tt></tt></tt></tt></tt></tt></tt></tt></tt></tt></tt></td>
</tr>
</tbody>
</table>
</td>
</tr>
</tbody>
</table>
[/su_spoiler]
[/su_accordion]

其他
[su_accordion]
[su_spoiler title="PEM and OpenSSH v2 public key structure" style="fancy"]

see [converting-keys-between-openssl-and-openssh](http://security.stackexchange.com/questions/32768/converting-keys-between-openssl-and-openssh)
**Generate an RSA pair**
All the following methods give an RSA key pair in the same format
1\. With openssl (man genrsa)
openssl genrsa -out dummy-genrsa.pem 2048
In OpenSSL v1.0.1 genrsa is superseded by genpkey so this is the new way to do it (man genpkey):
openssl genpkey -algorithm RSA -out dummy-genpkey.pem -pkeyopt rsa_keygen_bits:2048
2\. With ssh-keygen
ssh-keygen -t rsa -b 2048 -f dummy-ssh-keygen.pem -N '' -C "Test Key"

**Converting DER to PEM**
If you have an RSA key pair in DER format, you may want to convert it to PEM to allow the format conversion below:
Generation:
openssl genpkey -algorithm RSA -out genpkey-dummy.cer -outform DER -pkeyopt rsa_keygen_bits:2048
Conversion:
openssl rsa -inform DER -outform PEM -in genpkey-dummy.cer -out dummy-der2pem.pem

**Extract the public key from the PEM formatted RSA pair**
1\. in PEM format:
openssl rsa -in dummy-xxx.pem -pubout
2\. in OpenSSH v2 format see:
ssh-keygen -y -f dummy-xxx.pem

[/su_spoiler]
[/su_accordion]

其他一些有用链接
[su_accordion]
[su_spoiler title="Other useful links" style="fancy"]
[Public – Private key encryption using OpenSSL](http://www.devco.net/archives/2006/02/13/public_-_private_key_encryption_using_openssl.php)
[Convert keys between GnuPG, OpenSsh and OpenSSL](http://sysmic.org/dotclear/index.php?post/2010/03/24/Convert-keys-betweens-GnuPG%2C-OpenSsh-and-OpenSSL)
[sshpub-to-rsa](https://gist.github.com/thwarted/1024558)
[/su_spoiler]
[/su_accordion]