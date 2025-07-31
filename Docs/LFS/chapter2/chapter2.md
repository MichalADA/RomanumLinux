# LFS 12.3 Build Log - Chapter 2

## Chapter 2: Preparing the Host System 

### 2.1 Introduction
W tym rozdziale sprawdziliśmy wymagania host system i zainstalowaliśmy wszystkie niezbędne narzędzia deweloperskie dla budowy LFS.

### 2.2 Host System Requirements

#### 2.2.1 Hardware 
- **CPU**: minimum 4 cores  (mamy 4 cores w Proxmox)
- **RAM**: minimum 8 GB  (mamy 8 GB w Proxmox)
- **Dysk**: ~50 GB dla LFS  (mamy dedykowany dysk 50GB)

#### 2.2.2 Software Requirements

**Wymagane narzędzia i wersje minimalne:**
- Bash ≥ 3.2 (/bin/sh → bash)
- Binutils ≥ 2.13.1
- Bison ≥ 2.7 (/usr/bin/yacc → bison)  
- Coreutils ≥ 8.1
- Diffutils ≥ 2.8.1
- Findutils ≥ 4.2.31
- Gawk ≥ 4.0.1 (/usr/bin/awk → gawk)
- GCC ≥ 5.2 (z g++)
- Grep ≥ 2.5.1a
- Gzip ≥ 1.3.12
- Linux Kernel ≥ 5.4
- M4 ≥ 1.4.10
- Make ≥ 4.0
- Patch ≥ 2.5.4
- Perl ≥ 5.8.8
- Python ≥ 3.4
- Sed ≥ 4.1.5
- Tar ≥ 1.22
- Texinfo ≥ 5.0
- Xz ≥ 5.0.0

### 2.3 Wykonane kroki

**1. Instalacja pakietów deweloperskich na Debian 12:**
```bash
# Aktualizacja systemu
sudo apt update && sudo apt upgrade -y

# Instalacja wszystkich wymaganych pakietów dla LFS
sudo apt install -y build-essential bison flex gawk m4 texinfo \
    python3 python3-dev libssl-dev libncurses-dev libelf-dev \
    bc kmod cpio rsync wget curl patch tar xz-utils gzip \
    perl sed diffutils findutils grep binutils gcc g++ make
```

**2. Utworzenie i uruchomienie skryptu sprawdzającego wersje:**
```bash
# Stworzono skrypt version-check.sh z dokumentacji LFS
cat > version-check.sh << "EOF"
#!/bin/bash
# A script to list version numbers of critical development tools

LC_ALL=C
PATH=/usr/bin:/bin

bail() { echo "FATAL: $1"; exit 1; }
grep --version > /dev/null 2> /dev/null || bail "grep does not work"
sed '' /dev/null || bail "sed does not work"
sort /dev/null || bail "sort does not work"

ver_check()
{
    if ! type -p $2 &>/dev/null
    then
        echo "ERROR: Cannot find $2 ($1)"; return 1;
    fi
    v=$($2 --version 2>&1 | grep -E -o '[0-9]+\.[0-9\.]+[a-z]*' | head -n1)
    if printf '%s\n' $3 $v | sort --version-sort --check &>/dev/null
    then
        printf "OK: %-9s %-6s >= $3\n" "$1" "$v"; return 0;
    else
        printf "ERROR: %-9s is TOO OLD ($3 or later required)\n" "$1";
        return 1;
    fi
}

ver_kernel()
{
    kver=$(uname -r | grep -E -o '^[0-9\.]+')
    if printf '%s\n' $1 $kver | sort --version-sort --check &>/dev/null
    then
        printf "OK: Linux Kernel $kver >= $1\n"; return 0;
    else
        printf "ERROR: Linux Kernel ($kver) is TOO OLD ($1 or later required)\n" "$kver";
        return 1;
    fi
}

# Sprawdzenie wszystkich wymaganych narzędzi
ver_check Coreutils sort 8.1 || bail "Coreutils too old, stop"
ver_check Bash bash 3.2
ver_check Binutils ld 2.13.1
ver_check Bison bison 2.7
ver_check Diffutils diff 2.8.1
ver_check Findutils find 4.2.31
ver_check Gawk gawk 4.0.1
ver_check GCC gcc 5.2
ver_check "GCC (C++)" g++ 5.2
ver_check Grep grep 2.5.1a
ver_check Gzip gzip 1.3.12
ver_check M4 m4 1.4.10
ver_check Make make 4.0
ver_check Patch patch 2.5.4
ver_check Perl perl 5.8.8
ver_check Python python3 3.4
ver_check Sed sed 4.1.5
ver_check Tar tar 1.22
ver_check Texinfo texi2any 5.0
ver_check Xz xz 5.0.0
ver_kernel 5.4

if mount | grep -q 'devpts on /dev/pts' && [ -e /dev/ptmx ]
then echo "OK: Linux Kernel supports UNIX 98 PTY";
else echo "ERROR: Linux Kernel does NOT support UNIX 98 PTY"; fi

alias_check() {
    if $1 --version 2>&1 | grep -qi $2
    then printf "OK: %-4s is $2\n" "$1";
    else printf "ERROR: %-4s is NOT $2\n" "$1"; fi
}

echo "Aliases:"
alias_check awk GNU
alias_check yacc Bison
alias_check sh Bash

echo "Compiler check:"
if printf "int main(){}" | g++ -x c++ -
then echo "OK: g++ works";
else echo "ERROR: g++ does NOT work"; fi
rm -f a.out

if [ "$(nproc)" = "" ]; then
    echo "ERROR: nproc is not available or it produces empty output"
else
    echo "OK: nproc reports $(nproc) logical cores are available"
fi
EOF

chmod +x version-check.sh
./version-check.sh
```

**3. Naprawiono problemy z pierwszego uruchomienia:**
```bash
# Zainstalowano brakujące pakiety
sudo apt install -y gcc g++ bison m4 make texinfo

# Naprawiono linki symboliczne
sudo ln -sf /usr/bin/bison /usr/bin/yacc  # yacc → bison
sudo ln -sf /bin/bash /bin/sh             # sh → bash
```

**4. Finalna weryfikacja version-check.sh:**
```
OK: Coreutils 9.1      >= 8.1
OK: Bash      5.2.15   >= 3.2
OK: Binutils  2.40     >= 2.13.1
OK: Bison     3.8.2    >= 2.7
OK: Diffutils 3.8      >= 2.8.1
OK: Findutils 4.9.0    >= 4.2.31
OK: Gawk      5.2.1    >= 4.0.1
OK: GCC       12.2.0   >= 5.2
OK: GCC (C++) 12.2.0   >= 5.2
OK: Grep      3.8      >= 2.5.1a
OK: Gzip      1.12     >= 1.3.12
OK: M4        1.4.19   >= 1.4.10
OK: Make      4.3      >= 4.0
OK: Patch     2.7.6    >= 2.5.4
OK: Perl      5.36.0   >= 5.8.8
OK: Python    3.11.2   >= 3.4
OK: Sed       4.9      >= 4.1.5
OK: Tar       1.34     >= 1.22
OK: Texinfo   6.8      >= 5.0
OK: Xz        5.4.1    >= 5.0.0
OK: Linux Kernel 6.1.0 >= 5.4
OK: Linux Kernel supports UNIX 98 PTY
Aliases:
OK: awk  is GNU
OK: yacc is Bison
OK: sh   is Bash
Compiler check:
OK: g++ works
OK: nproc reports 4 logical cores are available
```

### Status Chapter 2:  COMPLETED
- [x] 2.1 Introduction - przeczytane
- [x] 2.2 Host System Requirements - wszystkie pakiety zainstalowane i zweryfikowane
- [x] 2.3 Building LFS in Stages - zanotowane
- [x] 2.4 Creating a New Partition - wykonane wcześniej
- [x] 2.5 Creating a File System - wykonane wcześniej  
- [x] 2.6 Setting $LFS Variable and Umask - wykonane wcześniej
- [x] 2.7 Mounting the New Partition - wykonane wcześniej

**Host system jest w pełni przygotowany do budowy LFS!**