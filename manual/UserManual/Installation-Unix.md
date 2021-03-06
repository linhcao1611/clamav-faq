# Installing ClamAV on Unix / Linux / macOS from Source

## Step-by-Step Instructions

- [Debian & Ubuntu](https://www.clamav.net/documents/installation-on-debian-and-ubuntu-linux-distributions)
- [Redhat & CentOS](https://www.clamav.net/documents/installation-on-debian-and-ubuntu-linux-distributions)
- [macOS](https://www.clamav.net/documents/installation-on-macos-mac-os-x)

---

## Requirements

The following is an overview of the tools, libraries, and steps needed to build ClamAV.

Required tools:

- `gcc` or `clang`
- GNU Make (`gmake` on UNIX systems)

Recommended tools:

- `check` unit testing framework

***Required*** libraries (including development sources (i.e. `...-dev` or `...-devel`)):

- zlib
- openssl version 0.9.8 or higher

**Recommended** libraries (including development sources (i.e. `...-dev` or `...-devel`)):

- pcre2
- bzip2
- libxml2

Optional libraries (including development sources (i.e. `...-dev` or `...-devel`)):

- curl library:     _required for clamsubmit_
- json-c library:   _required for clamsubmit_
- ncurses library:  _required for clamdtop_

ClamAV may execute Bytecode signatures using:

- ClamAV's built-in bytecode interpreter
- LLVM for Just-In-Time (JIT) compilation*
  - System-installed LLVM library (3.2-3.6)
  - ClamAV's built-in version of LLVM 2.8

    *The performance difference between using LLVM and using the interpeter is negligible. If you prefer to use LLVM / JIT for bytecode signature execution, be advised that we presently only support up to LLVM version 3.6.

The following are thus optional, but *required* to use LLVM in place of the bytecode interpeter:

- LLVM 3.2 - 3.6
- A supported CPU for LLVM JIT, either of: X86, X86-64, PowerPC, PowerPC64

The following are optional, but needed for the LLVM JIT unit tests:

- GNU Make (version 3.79, recommended 3.81 or newer)
- Python (version 2.5.4)

---

### Known Library Compatibility Issues

#### zlib

Certain versions on certain OSes will cause failures loading virus database:

- CentOS 6 32bit: zlib 1.2.3-29
  - Solution: Update to newer version.
- AIX 5.3: zlib 1.2.11-1
  - Solution: Try different version, downgrade may be required.

---

## Installing ClamAV

---

### Private installation on local shell account

To install ClamAV locally on an unprivileged shell account you need not create any additional users or groups. Assuming your home directory is `/home/gary` you should build it as follows:

<pre>
    ./configure --prefix=/home/gary/clamav --disable-clamav
    make; make install
</pre>

The `--disable-clamav` switch disables the check for existence of the `clamav` user and group but `clamscan` would still require an unprivileged account to work in a superuser mode.

---

### Global installation in system-owned directories

#### Adding new system user and group

If installing to the system, it is recommended to set up at least one special user account to run `freshclam` and `clamd`. You may choose to set up two separate accounts, one for each. You only need to create these accounts the first time you install ClamAV.

These are instructions specific to some popular operating systems:

- [Debian, Ubuntu, etc](https://www.clamav.net/documents/installation-on-debian-and-ubuntu-linux-distributions#Users-and-on-user-privileges)
- [Redhat, CentOS, etc](https://www.clamav.net/documents/installation-on-debian-and-ubuntu-linux-distributions#Users-and-on-user-privileges)
- [macOS](https://www.clamav.net/documents/installation-on-macos-mac-os-x#Users-and-on-user-privileges)

If your operating system isn't specified above, and your OS does not have the `groupadd` and `useradd` utilities, consult a system manual. **Don’t forget to lock access to the account!**

#### Compiling ClamAV for global installation

Once you have created the clamav user and group, please extract the archive:

<pre>
    tar xzf clamav-[ver].tar.gz
    cd clamav-[ver]
</pre>

Assuming you want to install the configuration files in `/etc`, configure and build the software as follows:

<pre>
    ./configure --sysconfdir=/etc
    make
    su -c "make install"
</pre>

In the last step, the software is installed into the `/usr/local` directory and the config files into `/etc`. **WARNING: Never enable the SUID or SGID bits for Clam AntiVirus binaries.**

---

### First-time set-up

First, create a database directory. This would be located under the install path `share/clamav`. For example:

- `/usr/local/share/clamav`
- `~/clamav/share/clamav`

You will need to create `freshclam.conf` and `clamd.conf` files in the config directory. In the above example, we chose `/etc`, so run the following.

<pre>
    sudo cp /etc/freshclam.conf.sample /etc/freshclam.conf
    sudo cp /etc/clamd.conf.sample /etc/clamd.conf
</pre>

At a minimum, you will need to edit each file and remove or comment-out the `Example` line. In addition, for `clamd.conf` you will need to enable either `LocalSocket` or `TCPSocket`.

For additional recommendations, please read:

- [Debian, Ubuntu, etc](https://www.clamav.net/documents/installation-on-debian-and-ubuntu-linux-distributions#First-time-set-up)
- [Redhat, CentOS, etc](https://www.clamav.net/documents/installation-on-debian-and-ubuntu-linux-distributions#First-time-set-up)
- [macOS](https://www.clamav.net/documents/installation-on-macos-mac-os-x#First-time-set-up)

---

### Test your installation

To test your local installation execute:

<pre>
    ~/clamav/bin/freshclam
    ~/clamav/bin/clamscan ~
</pre>

To test your system installation execute:

<pre>
    sudo freshclam
    sudo clamscan ~
</pre>

---

## Compilation with clamav-milter enabled

The `libmilter` package and its development files are required. To enable clamav-milter, configure ClamAV with

<pre>
    ./configure --enable-milter
</pre>

---

## Using a system-installed LLVM library

To configure ClamAV to use a system-installed LLVM library:

<pre>
    ./configure --with-system-llvm=/myllvm/bin/llvm-config
    make
    sudo make install
</pre>

The argument to `--with-system-llvm` indicates the path name of the LLVM configuration utility (llvm-config). Alternatively, you may use `--enable-llvm` and `./configure` will search for LLVM in /usr/local/ and then /usr.

Recommended versions of LLVM are 3.2 - 3.6. Some installations have reported problems using earlier LLVM versions. Versions of LLVM beyond 3.6 are not currently supported in ClamAV.

---

## Running unit tests

ClamAV includes unit tests that allow you to test that the compiled binaries work correctly on your platform.

The first step is to use your OS’s package manager to install the `check` package. If your OS doesn’t have that package, you can download it from [SourceForge](http://check.sourceforge.net/), build it and install it.

To help clamav’s configure script locate `check`, it is recommended that you install `pkg-config`, preferably using your OS’s package manager, or from [FreeDesktop.org](http://pkg-config.freedesktop.org).

The recommended way to run unit-tests is the following, which ensures you will get an error if unit tests cannot be built:

<pre>
    ./configure --enable-check
    make
    make check
</pre>

When `make check` is finished, you should get a message similar to this:

<pre>
    ==================
    All 8 tests passed
    ==================
</pre>

If a unit test fails, you get a message similar to the following. Note that in older versions of make check may report failures due to the absence of optional packages. Please make sure you have the latest versions of the components noted in section /refsec:components. See the next section on how to report a bug when a unit test fails.

<pre>
    ========================================
    1 of 8 tests failed
    Please report to https://bugzilla.clamav.net/
    ========================================
</pre>

If unit tests are disabled (and you didn’t use -–enable-check), you will get this message:

<pre>
    *** Unit tests disabled in this build
    *** Use ./configure --enable-check to enable them

    SKIP: check_clamav
    PASS: check_clamd.sh
    PASS: check_freshclam.sh
    PASS: check_sigtool.sh
    PASS: check_clamscan.sh
    ======================
    All 4 tests passed
    (1 tests were not run)
    ======================
</pre>

Running `./configure --enable-check` should tell you why.

---

## Reporting a unit test failure bug

If `make check` reports failed tests, we encourage you to report a bug on [bugzilla](https://bugzilla.clamav.net).

When writing a bug report regarding failed unit tests, please provide the following:

- The exact output from `make check`
- Output of `uname -mrsp`
- your `config.log`
- The following files from the `unit_tests/` directory:
  - `test.log`
  - `clamscan.log`
  - `clamdscan.log`

- `/tmp/clamd-test.log` if it exists
- where and how you installed the check package
- Output of `pkg-config check --cflags --libs`
- Optionally if `valgrind` is available on your platform, the output of the following:
    <pre>
        make check
        CK_FORK=no ./libtool --mode=execute valgrind unit_tests/check_clamav
    </pre>

---

## Obtain Latest ClamAV anti-virus signature databases

Before you can run `clamd`, `clamdscan`, or `clamscan`, you must have ClamAV Virus Database (.cvd) file(s) installed in the appropriate location on your system. The default location for these database files are `/usr/local/share/clamav`.

Here is a listing of currently available ClamAV Virus Database Files:

- bytecode.cvd (signatures to detect bytecode in files)
- main.cvd (main ClamAV virus database file)
- daily.cvd (daily update file for ClamAV virus databases)
- safebrowsing.cvd (virus signatures for safe browsing)

These files should be downloaded using the `freshclam` utility on a periodic basis. While using HTTPS to directly download the CVDs is possible, using `freshclam` is the preferred method of keeping the ClamAV virus database files up to date. `freshclam` can download database difference files (`.cdiff`) to get the latest signature definitions without downloading whole CVD files. This saves a considerable amount of bandwidth.

For more information on how to configure `freshclam` to do automatic/scheduled updates, see the [freshclam configuration section](Configuration.md#Setting-up-auto\-updating) of our Configuration guide.

Please see the [freshclam usage section](Usage.md#freshclam) for additional details on freshclam).

---

## Binary packages

As an alternative to building and installing from source, most Linux package managers provide pre-compiled ClamAV packages.

For more information about installing ClamAV via a Package Manager, please visit
the ["other versions" section on the ClamAV.net Downloads page](https://www.clamav.net/download.html#otherversions).
