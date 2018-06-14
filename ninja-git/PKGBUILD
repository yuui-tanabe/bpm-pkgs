pkgname=('llvm33')
pkgver=3.3
pkgrel=2
arch=('i686' 'x86_64')
url="http://llvm.org/"
pkgdesc="LLVM 3.3 (installed in /opt/llvm33/)"
license=('custom:University of Illinois/NCSA Open Source License')
makedepends=('libffi' 'python2' 'ocaml' 'python-sphinx')
source=(http://llvm.org/svn/llvm-project/llvm/branches/release_33/)
md5sum=('SKIP')


# Python site-packages dir (relative to ${pkgdir})
_py_sitepkg_dir="/usr/lib/python2.7/site-packages"

# Determine the installed OCaml package version
# Arguments: NONE
_ocamlver() {
    { pacman -Q ocaml 2>/dev/null || pacman -Sp --print-format '%n %v' ocaml ;} \
        | awk '{ print $2 }' | cut -d - -f 1 | cut -d . -f 1,2,3
}

# Fix the Python interpreter path in .py files to point to python2
# Arguments: py_file_to_patch [py_file_to_patch ...]
_fix_python_exec_path() {
    sed -i \
        -e 's|^#!/usr/bin/python$|&2|' \
        -e 's|^#!/usr/bin/env python$|&2|' \
        ${@}
}

# Compile the Python files in a directory
# Arguments: directory_to_operate_on
_compile_python_files() {
    python2 -m compileall "${1}"
    python2 -O -m compileall "${1}"
}

# Install the Python bindings of a package
# Arguments: source_directory_to_install_from
_install_python_bindings() {
    install -m 0755 -d "${pkgdir}${_py_sitepkg_dir}"
    cp -r "${1}" "${pkgdir}${_py_sitepkg_dir}/"
    _compile_python_files "${pkgdir}${_py_sitepkg_dir}/${1##*/}"
}

# Install the license files for a package
# Arguments: source_directory_to_install_from
# Notes: We prune some directories that are inserted into the tree in prepare() 
#        in order to eliminate possible duplicates. We also use NULL-terminated
#        strings, just in case we have paths including spaces. Finally, we opt
#        for a flat directory structure, so all license files in subdirectories
#        get their names from the relative path with '/'s replaced by dashes.
#        Not the most elegant solution, but should be working well enough.
_install_licenses() {
    find "${1}" \
        \( \
            -path "${srcdir}/${_pkgname}/tools/lld" -o \
            -path "${srcdir}/${_pkgname}/tools/clang" -o \
            -path "${srcdir}/${_pkgname}/tools/lldb" -o \
            -path "${srcdir}/${_pkgname}/projects/compiler-rt" \
        \) -prune -o \
        \( \
            -iname 'license*' -o \
            -iname 'credits*' -o \
            -iname 'copyright*' \
        \) -printf '%P\0' \
        | while read -d $'\0' license_file; do
            install -D -m 0644 \
                "${1}/${license_file}" \
                "${pkgdir}/usr/share/licenses/${pkgname}/${license_file//\//-}"
        done
}


pkgver() {
    cd "${srcdir}/${_pkgname}"

    # This will almost match the output of `llvm-config --version` when the
    # LLVM_APPEND_VC_REV cmake flag is turned on. The only difference is
    # dash being replaced with underscore because of Pacman requirements.
    echo $(awk -F 'MAJOR |MINOR |PATCH |SUFFIX |)' \
            'BEGIN { ORS="." ; i=0 } \
             /set\(LLVM_VERSION_/ { print $2 ; i++ ; if (i==2) ORS="" } \
             END { print "\n" }' \
        CMakeLists.txt)_r$(svnversion | tr -d [A-z])
}
build() {
    cd "${srcdir}/build"

    # Building with any already installed on the system LLVM OCaml bindings is very error-prone.
    # The problems almost certainly arise from incompatibilities between the installed system-wide
    # bindings and the newly built ones. Unfortunately, the OCAMLPATH environment variable doesn't
    # allow overriding the search path set in the system configuration file, only adding to it.
    # Even same version bindings cause problems in certain circumstances, so let's play safe.
    ocamlfind query llvm >/dev/null 2>&1 && {
        error 'Incompatible LLVM OCaml bindings installed.'
        plain 'Building with already installed on the system LLVM OCaml bindings is not supported.'
        plain 'Please either uninstall any currently installed llvm-ocaml* package before building,'
        plain 'or, __preferably__, build in a clean chroot, as described on the Arch Linux wiki:'
        plain 'https://wiki.archlinux.org/index.php/DeveloperWiki:Building_in_a_Clean_Chroot'
        exit 1
    }

    export PKG_CONFIG_PATH='/usr/lib/pkgconfig'

    # LLVM_BUILD_LLVM_DYLIB: Build the dynamic runtime libraries (e.g. libLLVM.so).
    # LLVM_LINK_LLVM_DYLIB:  Link our own tools against the libLLVM dynamic library, too.
    # LLVM_BINUTILS_INCDIR:  Set to binutils' plugin-api.h location in order to build LLVMgold.
    cmake -G 'Unix Makefiles' \
        -DCMAKE_BUILD_TYPE:STRING=Release \
        -DCMAKE_INSTALL_PREFIX:PATH=/usr \
        -DLLVM_APPEND_VC_REV:BOOL=ON \
        -DLLVM_ENABLE_RTTI:BOOL=ON \
        -DLLVM_ENABLE_FFI:BOOL=ON \
        -DFFI_INCLUDE_DIR:PATH="$(pkg-config --variable=includedir libffi)" \
        -DFFI_LIBRARY_DIR:PATH="$(pkg-config --variable=libdir libffi)" \
        -DLLVM_BUILD_DOCS:BOOL=ON \
        -DLLVM_ENABLE_SPHINX:BOOL=ON \
        -DSPHINX_OUTPUT_HTML:BOOL=ON \
        -DSPHINX_OUTPUT_MAN:BOOL=ON \
        -DSPHINX_WARNINGS_AS_ERRORS:BOOL=OFF \
        -DLLVM_BUILD_LLVM_DYLIB:BOOL=ON \
        -DLLVM_LINK_LLVM_DYLIB:BOOL=ON \
        -DLLVM_BINUTILS_INCDIR:PATH=/usr/include \
        "../${_pkgname}"

    make
    make ocaml_doc
}

check() {
    cd "${srcdir}/build"
    # Dirty fix for unittests failing because the shared lib is not in the library path.
    # Also, disable the LLVM tests on i686 as they seem to fail too often there.
    [[ "${CARCH}" == "i686" ]] || LD_LIBRARY_PATH="${srcdir}/build/lib" make check
    make check-clang
}
package_llvm-svn() {
    pkgdesc='The LLVM Compiler Infrastructure'
    depends=(
        "llvm-libs-svn=${pkgver}-${pkgrel}"
    )
    groups=('llvm-toolchain-svn')
    provides=('llvm')
    conflicts=('llvm')

    cd "${srcdir}/build"

    # Disable automatic installation of components that go into subpackages
    sed -i '/\(clang\|lld\|lldb\)\/cmake_install.cmake/d' tools/cmake_install.cmake

    make DESTDIR="${pkgdir}" install

    # The runtime libraries get installed in llvm-libs-svn
    rm -f "${pkgdir}"/usr/lib/lib{LLVM,LTO}{,-*}.so{,.*}
    mv -f "${pkgdir}"/usr/lib/{BugpointPasses,LLVMgold,LLVMHello}.so "${srcdir}/"

    # Clang libraries and OCaml bindings go to separate packages
    rm -rf "${srcdir}"/{clang,ocaml.{doc,lib}}
    mv "${pkgdir}/usr/lib/clang" "${srcdir}/clang"
    mv "${pkgdir}/usr/lib/ocaml" "${srcdir}/ocaml.lib"
    mv "${pkgdir}/usr/share/doc/llvm/ocaml-html" "${srcdir}/ocaml.doc"

    if [[ "${CARCH}" == "x86_64" ]]; then
        # Needed for multilib (https://bugs.archlinux.org/task/29951)
        # Header stubs are taken from Fedora
        mv "${pkgdir}/usr/include/llvm/Config/llvm-config"{,-64}.h
        cp "${srcdir}/llvm-Config-llvm-config.h" "${pkgdir}/usr/include/llvm/Config/llvm-config.h"
    fi

    # Clean up documentation
    # TODO: This may not be needed any more.
    rm -rf "${pkgdir}/usr/share/doc/llvm/html/_sources"

    _install_python_bindings "${srcdir}/llvm/bindings/python/llvm"

    _install_licenses "${srcdir}/llvm"
}

package_llvm-libs-svn() {
    pkgdesc='The LLVM Compiler Infrastructure (runtime libraries)'
    depends=(
        'libffi'
        'zlib'
    )
    groups=('llvm-toolchain-svn')
    provides=('llvm-libs')
    conflicts=('llvm-libs')

    cd "${srcdir}/build"

    make DESTDIR="${pkgdir}" install-{LLVM,LTO}

    # Moved from the llvm-svn package here
    mv "${srcdir}"/{BugpointPasses,LLVMgold,LLVMHello}.so "${pkgdir}/usr/lib/"

    # Ref: https://llvm.org/docs/GoldPlugin.html
    install -m755 -d "${pkgdir}/usr/lib/bfd-plugins"
    ln -s {/usr/lib,"${pkgdir}/usr/lib/bfd-plugins"}/LLVMgold.so

    # Since r262066 lto.h is also installed, but we don't need it in the -libs package.
    rm -rf "${pkgdir}/usr/include"

    # Must have a symlink that corresponds to the output of `llvm-config --version`.
    # Without it, some builds, e.g. Mesa, might fail for "lack of shared libraries".
    _sover="$(echo ${pkgver} | cut -d . -f -1)svn"
    # libLLVM.so.3.8.0svn-r123456
    ln -s "libLLVM-${_sover}.so" "${pkgdir}/usr/lib/libLLVM.so.$(echo ${pkgver} | tr _ -)"
    # libLLVM-3.8.0svn-r123456.so
    ln -s "libLLVM-${_sover}.so" "${pkgdir}/usr/lib/libLLVM-$(echo ${pkgver} | tr _ -).so"

    _install_licenses "${srcdir}/llvm"
}

package_llvm-ocaml-svn() {
    pkgdesc='OCaml bindings for LLVM'
    depends=(
        "llvm-svn=${pkgver}-${pkgrel}"
        "ocaml=$(_ocamlver)"
        'ocaml-ctypes'
    )
    provides=('llvm-ocaml')
    conflicts=('llvm-ocaml')

    cd "${srcdir}/build"

    install -m755 -d "${pkgdir}/usr/lib"
    install -m755 -d "${pkgdir}/usr/share/doc/llvm"
    cp -a "${srcdir}/ocaml.lib" "${pkgdir}/usr/lib/ocaml"
    cp -a "${srcdir}/ocaml.doc" "${pkgdir}/usr/share/doc/llvm/ocaml-html"

    _install_licenses "${srcdir}/llvm"
}
