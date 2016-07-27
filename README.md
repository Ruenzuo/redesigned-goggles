## Trying to understand how linking works

* libfoo_dynamic is a dynamic library that depends on CoreFoundation.
* libaba_dynamic is a dynamic library that depends on CoreFoundation and libfoo_dynamic.
* main should be an executable that dynamically links both libfoo_dynamic and libfoo_dynamic.

### Building libfoo_dynamic

```bash
clang -c bar.c -o bar.o # create object file
libtool -dynamic bar.o -o libfoo_dynamic.dylib -framework CoreFoundation -lSystem -macosx_version_min 10.11 # create dynamic library
install_name_tool -id "@rpath/../libfoo_dynamic/libfoo_dynamic.dylib" libfoo_dynamic.dylib # change the install name
```

### Building libaba_dynamic

```bash
clang -c aba.c -o aba.o -I ../libfoo_dynamic/ # create object file
libtool -dynamic aba.o -o libaba_dynamic.dylib -framework CoreFoundation -lSystem -L../libfoo_dynamic/ -lfoo_dynamic -macosx_version_min 10.11 # create dynamic library
install_name_tool -id "@rpath/../libaba_dynamic/libaba_dynamic.dylib" libaba_dynamic.dylib # change the install name
```

### Building main

```bash
clang -c main.c -o main.o -I ../libfoo_dynamic/ -I ../libaba_dynamic/ # create object file
ld main.o -framework CoreFoundation -lSystem -L../libaba_dynamic/ -laba_dynamic -L../libfoo_dynamic -lfoo_dynamic -o main -macosx_version_min 10.11 -rpath . # create binary
```

### Result

```bash
./main 
buzz
buzz
bab

otool -L ./main
./main:
  /System/Library/Frameworks/CoreFoundation.framework/Versions/A/CoreFoundation (compatibility version 150.0.0, current version 1258.1.0)
  /usr/lib/libSystem.B.dylib (compatibility version 1.0.0, current version 1226.10.1)
  @rpath/../libaba_dynamic/libaba_dynamic.dylib (compatibility version 0.0.0, current version 0.0.0)
  @rpath/../libfoo_dynamic/libfoo_dynamic.dylib (compatibility version 0.0.0, current version 0.0.0)

nm main
0000000000001000 A __mh_execute_header
                 U _aba
                 U _fizz
0000000000001f80 T _main
                 U dyld_stub_binder
```
