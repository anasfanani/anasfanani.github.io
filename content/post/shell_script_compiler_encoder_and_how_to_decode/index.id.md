---
language: id
title: Meng-encode Script BASH Dan Cara Men-decode
tags:
  - Bash
categories:
  - Linux
date: 2024-04-01 08:24:00+07:00
description: Encode dan Decode Shell Script / Bash Script Menggunakan SHC, Lalu
  Mendecodenya Menggunakan SHX-DUMPER
---

## Shell Compiler

Shell compiler source code can be found at this repo [https://github.com/neurobin/shc](https://github.com/neurobin/shc "https\://github.com/neurobin/shc").

You may need to read the [README.md](https://github.com/neurobin/shc/blob/master/README) at their repo carefully .

### Quick Start

A quickstart for usage the `shc` a.k.a shell compiler from me:

#### Build from source using GitHub codespace

Go to github codespace https://github.com/codespaces and create new, or you may go to https://github.com/neurobin/shc and select create codespace.

Then input this command to configure.

```bash
git clone https://github.com/neurobin/shc
cd shc 
./configure 
make
sudo make install
```

Create simple file named as `test.sh` :

```bash
#!/bin/sh
echo "hello world"
id
uname -a
```

Then use `shc` with this command :

```
shc -f test.sh
```

We got two file, `test.sh.x` and `test.sh.c`, for now while type this article, i dont know at much what is `test.sh.c` file for.

Lets chmod for allow executing the binary.

```
chmod +x ./test.sh.x
```

Then execute the binary :

```
./test.sh.x
```

Sample output :

```
@anasfanani ➜ /workspaces/ (master) $ ./test.sh.x 
hello world
uid=1000(codespace) gid=1000(codespace) groups=1000(codespace),106(ssh),107(docker),988(pipx),989(python),990(oryx),991(golang),992(sdkman),993(rvm),994(php),995(conda),996(nvs),997(nvm),998(hugo),999(dotnet)
Linux codespaces-67d0c2 5.15.0-1041-azure #48-Ubuntu SMP Tue Jun 20 20:34:08 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux
```

Trying to read the file with `cat ./test.sh.x | head -n 2` :

```
@@@@��3@8
      �
��kX���O�퐖��GNU��@��ݣk�e�mĹ�@9��a >S-Z�rLy��; �(����Q�tdR�td-==��/lib64/ld-linux-x86-64.so.2GNU�GNU��
                                                    �J E��!��!��"����libc.so.6exitsprintf__isoc99_sscanftime__stack_chk_failgetpidstrdupcallocstrlenmemset__errno_locationmemcmpputenvmemcpymallocgetenvs�����iieuipfwrit@�?��?�?�?�B�B(?0?8?@?H?P?X?trer`?r__libc_start_main__environ__xstatGLIBC_2.7GLIBC_2.14GLIBC_2.4GLIBC_2.2.5_ITM_deregisterTMCloneTable__gmon_start___ITM_registerTMCloneTableii
h?
  p?
�?�?�?�?�?�?�?�?�?�?�?��H�H��/H��t��H���5�.��%�.��h���������h���������h���������h��������h��������h��������h��������h��q��������a������h        ��Q������h
```

Then using `file test.sh.x` :

```
test.sh.x: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=bcdc0dde1319c86b5802a7b9c34f9eed909684e2, for GNU/Linux 3.2.0, stripped
```

- - -
## Deobfuscate / Decompile

Found how to decompile the binary from [https://github.com/neurobin/shc/issues/99](https://github.com/neurobin/shc/issues/99 "https\://github.com/neurobin/shc/issues/99").

### Quick Start

Lets try the quickstart from me using this snippet :

```
git clone https://github.com/niansa/bash-shxdumper
cd bash-shxdumper
./configure
make -j$(nproc) || make
```

At this point, we got new modified `./bash` file, now move this bash file to the system :

```
sudo mv /bin/bash /bin/bash.bak && sudo cp ./bash /bin/bash
```

Dont worry, your original bash is backup to `/bin/bash.bak`, I dont need told you about this.

Now we try to deobfuscate `./test.sh.x` files :

```
OUTFILE=./decrypted.sh ./test.sh.x
```

And result is failed to deobfuscate ( -_- ).

It seems the shc update their code, shxdumper doesnt work again.

I have another trick to show the running code from process, while you run a shx code, you can see the source with `ps -aux | grep ./test.sh.x` the code is showing in process list, but you cant see again if process already killed, thats all I know.

I wont write much more because I'm boring and this blog is for my personal note.

