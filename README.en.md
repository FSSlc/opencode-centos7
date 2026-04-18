# opencode-centos7

Adapts opencode to run on CentOS 7 and other legacy Linux systems.

## Patchelf Principles and Effectiveness

This project uses the `patchelf` tool to patch the opencode binary, enabling it to run on CentOS 7 and other legacy Linux systems. Below is a detailed explanation of how it works and why it is effective.

### Patchelf Introduction
`patchelf` is an open-source tool for modifying dynamic linking information in ELF (Executable and Linkable Format) binaries. It can change:
- Dynamic linker (interpreter)
- RPATH (Run-time search path)
- RUNPATH
- And other ELF header information

### Usage in the Script
In the `patch-opencode.sh` script, we perform two key operations:

1. **Set RPATH**:
   ```bash
   $PREFIX/bin/patchelf --set-rpath $PREFIX/x86_64-conda-linux-gnu/sysroot/lib64/:$PREFIX/x86_64-conda-linux-gnu/sysroot/usr/lib64 $PREFIX/bin/opencode
   ```
   This sets the RPATH of the opencode binary to point to the sysroot library paths in the conda environment.

2. **Set Dynamic Linker**:
   ```bash
   $PREFIX/bin/patchelf --set-interpreter $PREFIX/x86_64-conda-linux-gnu/sysroot/lib64/ld-linux-x86-64.so.2 $PREFIX/bin/opencode
   ```
   This sets the dynamic linker to ld-linux-x86-64.so.2 in the conda sysroot.

### Why It Works

#### 1. Resolving Library Version Compatibility Issues
- **Problem**: CentOS 7 uses an older glibc version (2.17), while modern software like opencode may require newer glibc features.
- **Solution**: By setting RPATH, the program prioritizes shared libraries from the conda-provided sysroot, which are compatible with opencode.

#### 2. Bypassing System Dynamic Linker Limitations
- **Problem**: The system's default dynamic linker may not support certain new features or be incompatible with legacy systems.
- **Solution**: Use the dynamic linker from the conda sysroot, which works in coordination with the sysroot libraries to ensure proper linking and loading.

#### 3. Environment Isolation
- **Advantage**: The RPATH mechanism allows the program to use specific library versions without depending on globally installed system libraries. This provides better portability and compatibility.

#### 4. Build-Time Integration
- In `construct.yaml`, `post_install` specifies `patch-opencode.sh`, ensuring these patches are applied automatically after the conda package is built.
- sysroot-conda_2_28-x86_64 provides the necessary libraries and linker files.

### Technical Details
- **ELF RPATH**: RPATH is a list of search paths embedded in the ELF file, used by the dynamic linker to locate shared libraries.
- **Dynamic Linker**: ld-linux.so.2 is the standard dynamic linker on Linux, responsible for loading and linking shared libraries.
- **Sysroot**: The conda-provided sysroot is a self-contained root filesystem containing all libraries and tools needed for building and running.

Through these patches, opencode can run on CentOS 7 and other legacy systems without modifying system libraries or upgrading the OS.

## Version History (Changelog)

### v1.4.4 (Current Version)
- Initial release
- Support for opencode 1.4.4 compatibility adaptation on CentOS 7
- Use patchelf to patch binaries to resolve library dependency issues
- Added GitHub Actions workflow for automated release builds and publishing
- Support for linux-64 and linux-aarch64 architecture package builds

### Future Plans
- Extend support for more legacy Linux distributions
- Optimize build processes and documentation

## References

- https://github.com/YangLeiSX/opencode-centos7
- https://github.com/Tao-Yida/opencode-on-centos7

## Improvements
- Use conda constructor for packaging, available to regular users
- Added support for linux-aarch64
- No need to manually compile gcc, glibc, etc.