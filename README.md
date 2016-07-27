## Trying to understand how linking works

* libfoo_dynamic is a dynamic library that depends on CoreFoundation.
* libaba_dynamic is a dynamic library that depends on CoreFoundation and libfoo_dynamic.
* main should be an executable that dynamically links both libfoo_dynamic and libfoo_dynamic.

### Building libfoo_dynamic

```bash
clang -c bar.c -o bar.o # create object file
libtool -dynamic bar.o -o libfoo_dynamic.dylib -framework CoreFoundation -lSystem -macosx_version_min 10.11 # create dynamic library
```

### Building libaba_dynamic

```bash
clang -c aba.c -o aba.o -I ../libfoo_dynamic/ # create object file
libtool -dynamic aba.o -o libaba_dynamic.dylib -framework CoreFoundation -lSystem -L../libfoo_dynamic/ -lfoo_dynamic -macosx_version_min 10.11 # create dynamic library
```

### Building main

```bash
clang -c main.c -o main.o -I ../libfoo_dynamic/ -I ../libaba_dynamic/ # create object file
ld main.o -framework CoreFoundation -lSystem -L../libaba_dynamic/ -laba_dynamic -L../libfoo_dynamic -lfoo_dynamic -o main -macosx_version_min 10.11 # create binary
```

### Result

```bash
./main 
dyld: Library not loaded: libfoo_dynamic.dylib
  Referenced from: /Users/renzo.crisostomo/Documents/XING/Ruenzuo/dynamic-linking/main/./main
  Reason: image not found
Trace/BPT trap: 5
```
