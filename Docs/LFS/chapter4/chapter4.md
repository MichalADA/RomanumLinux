# LFS Chapter 4 - Final Preparations - Complete Summary

##  Chapter Objective
Prepare the build environment by creating directory structure, adding unprivileged user, and configuring the build environment for LFS construction.

##  Step 1: Create LFS Directory Structure

### Basic directory layout
```bash
# As root user
mkdir -pv $LFS/{etc,var} $LFS/usr/{bin,lib,sbin}

# Create symbolic links for compatibility
for i in bin lib sbin; do
    ln -sv usr/$i $LFS/$i
done

# For 64-bit systems (x86_64)
case $(uname -m) in
    x86_64) mkdir -pv $LFS/lib64 ;;
esac

# Create tools directory for cross-compiler
mkdir -pv $LFS/tools
```

## 👤 Step 2: Add LFS User

### Create lfs group and user
```bash
# Create lfs group
groupadd lfs

# Create lfs user with bash shell
useradd -s /bin/bash -g lfs -m -k /dev/null lfs

# Set password for lfs user
passwd lfs
```

**Command options explained:**
- `-s /bin/bash`: Set bash as default shell
- `-g lfs`: Add user to lfs group
- `-m`: Create home directory
- `-k /dev/null`: Prevent copying from skeleton directory

##  Step 3: Grant Permissions to LFS User

### Set ownership of LFS directories
```bash
# Grant lfs full access to all LFS directories
chown -v lfs $LFS/{usr{,/*},var,etc,tools}

# For 64-bit systems
case $(uname -m) in
    x86_64) chown -v lfs $LFS/lib64 ;;
esac
```

##  Step 4: Environment Configuration

### Switch to lfs user
```bash
su - lfs
```

### Create .bash_profile
```bash
cat > ~/.bash_profile << "EOF"
exec env -i HOME=$HOME TERM=$TERM PS1='\u:\w\$ ' /bin/bash
EOF
```

**Purpose:** Creates clean environment without host system contamination.

### Create .bashrc with full configuration
```bash
cat > ~/.bashrc << "EOF"
set +h
umask 022
LFS=/mnt/lfs
LC_ALL=POSIX
LFS_TGT=$(uname -m)-lfs-linux-gnu
PATH=/usr/bin
if [ ! -L /bin ]; then PATH=/bin:$PATH; fi
PATH=$LFS/tools/bin:$PATH
CONFIG_SITE=$LFS/usr/share/config.site
export LFS LC_ALL LFS_TGT PATH CONFIG_SITE
EOF
```

**Environment variables explained:**
- `set +h`: Disable bash hash function (use fresh tools immediately)
- `umask 022`: Set safe file permissions (644 for files, 755 for directories)
- `LFS=/mnt/lfs`: LFS mount point
- `LC_ALL=POSIX`: Set POSIX locale for consistency
- `LFS_TGT=$(uname -m)-lfs-linux-gnu`: Target architecture for cross-compilation
- `PATH`: Prioritize tools from $LFS/tools/bin
- `CONFIG_SITE`: Prevent host contamination in configure scripts

### Add parallel compilation support
```bash
cat >> ~/.bashrc << "EOF"
export MAKEFLAGS=-j$(nproc)
EOF
```

**Purpose:** Enable parallel compilation using all CPU cores.

##  Step 5: Clean Host Environment

### Remove potentially problematic bash.bashrc (as root)
```bash
# Exit lfs user, return to root
exit

# Check and remove problematic file
[ ! -e /etc/bash.bashrc ] || mv -v /etc/bash.bashrc /etc/bash.bashrc.NOUSE
```

**Purpose:** Prevent host system's bash configuration from interfering with LFS build.

## 🔄 Step 6: Activate Environment

### Return to lfs user and load environment
```bash
# Switch back to lfs user
su - lfs

# Load the new environment
source ~/.bash_profile
```

##  Step 7: Verification

### Check environment variables
```bash
echo $LFS          # Should show: /mnt/lfs
echo $LFS_TGT      # Should show: x86_64-lfs-linux-gnu
echo $PATH         # Should include: /mnt/lfs/tools/bin
echo $MAKEFLAGS    # Should show: -j<number_of_cores>
```

### Example correct output:
- **LFS:** `/mnt/lfs`
- **LFS_TGT:** `x86_64-lfs-linux-gnu`
- **PATH:** `/mnt/lfs/tools/bin:/usr/bin` (and possibly `/bin`)
- **MAKEFLAGS:** `-j4` (or number of your CPU cores)

##  Understanding SBUs (Standard Build Units)

### What is SBU?
- **SBU** = Time to compile first package (binutils in Chapter 5)
- Used as relative measure for other packages
- Example: if binutils = 4 minutes, then "4.5 SBU" package ≈ 18 minutes

### Performance optimization:
- Use `powerprofilesctl set performance` for consistent timing
- Or `tuned-adm profile throughput-performance` on some distros
- Parallel compilation (`-j<cores>`) speeds up builds significantly

##  About Test Suites

### Importance levels:
- **Critical:** GCC, binutils, glibc test suites
- **Chapter 5-6:** Tests pointless (cross-compiled programs can't run on host)
- **Chapter 8:** Tests become meaningful and recommended

### Common issues:
- May fail due to insufficient PTYs (pseudo terminals)
- Some failures are known and documented at build logs
- Expected failures listed at: https://www.linuxfromscratch.org/lfs/build-logs/12.3/

##  Chapter 4 Completion Checklist

- ✅ **Directory structure created** in $LFS
- ✅ **LFS user created** with proper shell and permissions
- ✅ **Environment configured** with clean, isolated settings
- ✅ **Parallel compilation enabled** for faster builds
- ✅ **Host contamination prevented** via environment isolation
- ✅ **All variables verified** and working correctly

##  Status: CHAPTER 4 COMPLETED SUCCESSFULLY

The build environment is now properly configured and isolated. Ready to proceed to **Chapter 5 - Compiling a Cross-Toolchain**.

##  Next Steps
Chapter 5 will build:
1. **Binutils Pass 1** (cross-linker and assembler)
2. **GCC Pass 1** (cross-compiler)
3. **Linux API Headers** (kernel interface)
4. **Glibc** (C library)
5. **Libstdc++** (C++ standard library)

The cross-toolchain will be installed in `$LFS/tools/` directory and used to build temporary tools in Chapter 6.