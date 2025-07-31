# LFS Chapter 3 - Downloading Packages and Patches

##  Chapter Objective
Download all source packages and patches required to build Linux From Scratch 12.3-systemd system.

##  Preparing the sources directory

### Creating the directory
```bash
mkdir -v $LFS/sources
```

### Setting permissions (writable + sticky)
```bash
chmod -v a+wt $LFS/sources
```

##  Downloading files

### Downloading download lists
```bash
cd $LFS/sources

# Download wget-list for systemd
wget https://www.linuxfromscratch.org/lfs/downloads/stable-systemd/wget-list

# Download MD5 checksums
wget https://www.linuxfromscratch.org/lfs/downloads/stable-systemd/md5sums
```

### Automatic package download
```bash
wget --input-file=wget-list --continue --directory-prefix=$LFS/sources
```

##  Encountered Issues

### Problem with Expat-2.6.4
- **Symptom:** `expat-2.6.4.tar.xz: No such file or directory`
- **Cause:** Invalid or unavailable link in wget-list
- **Solution:** Manual download from GitHub
```bash
wget -O expat-2.6.4.tar.xz https://github.com/libexpat/libexpat/releases/download/R_2_6_4/expat-2.6.4.tar.xz
```

## 🔧 Patches - manual download

Required patches for LFS 12.3-systemd:

```bash
# Bzip2 - documentation installation
wget https://www.linuxfromscratch.org/patches/lfs/12.3/bzip2-1.0.8-install_docs-1.patch

# Coreutils - internationalization fixes
wget https://www.linuxfromscratch.org/patches/lfs/12.3/coreutils-9.6-i18n-1.patch

# Expect - GCC 14 compatibility
wget https://www.linuxfromscratch.org/patches/lfs/12.3/expect-5.45.4-gcc14-1.patch

# Glibc - FHS compliance
wget https://www.linuxfromscratch.org/patches/lfs/12.3/glibc-2.41-fhs-1.patch

# Kbd - Backspace/Delete key fix
wget https://www.linuxfromscratch.org/patches/lfs/12.3/kbd-2.7.1-backspace-1.patch
```

## ✅ Integrity verification

### Checking MD5 checksums
```bash
pushd $LFS/sources
md5sum -c md5sums
popd
```

### Setting file ownership
```bash
chown root:root $LFS/sources/*
```

##  Final result

- **97 files** in `$LFS/sources` directory
- **~500 MB** of data downloaded
- **All packages verified** by MD5 checksums
- **Ownership set to root**

##  Downloaded components

### Main packages
- **Compilers:** GCC-14.2.0, Binutils-2.44
- **C Library:** Glibc-2.41
- **Compression:** Zlib-1.3.1, Bzip2-1.0.8, Xz-5.6.4
- **Terminal:** Ncurses-6.5, Readline-8.2.13
- **Shell:** Bash-5.2.37
- **Utilities:** Coreutils-9.6, Findutils-4.10.0, Grep-3.11
- **Python:** Python-3.13.2, Setuptools-75.8.1, Wheel-0.45.1
- **Init system:** Systemd-257.3, D-Bus-1.16.0
- **Kernel:** Linux-6.13.4
- **Editor:** Vim-9.1.1166
- **And 70+ other packages**

### Patches
- bzip2-1.0.8-install_docs-1.patch (1.6 KB)
- coreutils-9.6-i18n-1.patch (164 KB)
- expect-5.45.4-gcc14-1.patch (7.8 KB)
- glibc-2.41-fhs-1.patch (2.8 KB)
- kbd-2.7.1-backspace-1.patch (12 KB)

##  Lessons learned

1. **wget-list may contain outdated links** - be prepared for manual downloads
2. **Patches must be downloaded separately** - not always included in wget-list
3. **MD5 verification is crucial** for ensuring file integrity
4. **Proper permissions are important** - sticky bit prevents accidental deletion

## ✅ Status: CHAPTER 3 COMPLETED SUCCESSFULLY

All required packages and patches have been downloaded and verified. System ready to proceed to **Chapter 4 - Final Preparations**.