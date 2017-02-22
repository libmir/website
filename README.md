Mir documentation
=================

How to run?
-----------

```
git submodule update --init
make -f doc/Makefile clean html
```

How to add a new package?
-------------------------

### 1) Add it as submodule

```
git submodule add https://github.com/libmir/mir-foo
```

### 2) Setup relative symlinks in `all`

For example:

```
ln -s ../mir-cpuid/source/cpuid all/
```

### 3) Add your modules to the documentation engine in the Makefile


### 4) Commit & enjoy
