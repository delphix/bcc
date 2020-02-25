Building
-------

To build libbpf-based tools, simply run `make`. This will build all the listed
tools/applications. All the build artifacts, by default, go into .output
subdirectory to keep source code and build artifacts completely separate. The
only exception is resulting tool binaries, which are put in a current
directory. `make clean` will clean up all the build artifacts, including
generated binaries.

Given libbpf package might not be available across wide variety of
distributions, all libbpf-based tools are linked statically against version of
libbpf that BCC links against (from submodule under src/cc/libbpf). This
results in binaries with minimal amount of dependencies (libc, libelf, and
libz are linked dynamically, though, given their widespread availability).

Tools are expected to follow a simple naming convention:
  - <tool>.c contains userspace C code of a tool.
  - <tool>.bpf.c contains BPF C code, which gets compiled into BPF ELF file.
    This ELF file is used to generate BPF skeleton <tool>.skel.h, which is
    subsequently is included from <tool>.c.
  - <tool>.h can optionally contain any types and constants, shared by both
    BPF and userspace sides of a tool.

For such cases, simply adding <tool> name to Makefile's APPS variable will
ensure this tool is built alongside others.

For more complicated applications, some extra Makefile rules might need to be
created. For such cases, it is advised to put application into a dedicated
subdirectory and link it from main Makefile.

vmlinux.h generation
-------------------

vmlinux.h contains all kernel types, both exported and internal-only. BPF
CO-RE-based applications are expected to include this file in their BPF
program C source code to avoid dependency on kernel headers package.

For more reproducible builds, vmlinux.h header file is pre-generated and
checked in along the other sources. This is done to avoid dependency on
specific user/build server's kernel configuration, because vmlinux.h
generation depends on having a kernel with BTF type information built-in
(which is enabled by `CONFIG_DEBUG_INFO_BTF=y` Kconfig option).

vmlinux.h is generated from upstream Linux version at particular minor
version tag. E.g., `vmlinux_505.h` is generated from v5.5 tag. Exact set of
types available in compiled kernel depends on configuration used to compile
it. To generate present vmlinux.h header, default configuration was used, with
only extra `CONFIG_DEBUG_INFO_BTF=y` option enabled.

Given different kernel version can have incompatible type definitions, it
might be important to use vmlinux.h of a specific kernel version as a "base"
version of header. To that extent, all vmlinux.h headers are versioned by
appending <MAJOR><MINOR> suffix to a file name. There is always a symbolic
link vmlinux.h, that points to whichever version is deemed to be default
(usually, latest).

bpftool
-------

bpftool is a universal tool used for inspection of BPF resources, as well as
providing various extra BPF-related facilities, like code-generation of BPF
program skeletons. The latter functionality is heavily used by these tools to
load and interact with BPF programs.

Given bpftool package can't yet be expected to be available widely across many
distributions, bpftool binary is checked in into BCC repository in bin/
subdirectory. Once bpftool package is more widely available, this can be
changed in favor of using pre-packaged version of bpftool.
