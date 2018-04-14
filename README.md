ct_choose: Constant-time choose between two variables
=======================================================
This project contains Clang/LLVM patches/passes for a new built-in function:
```c
Type val = __builtin_ct_choose(bool condition, Type valTrue, Type valFalse);
```

This function returns `valTrue` if `condition` is true, else `valFalse`, and does so in constant time.
The X86_64 backend will compile this function into a `CMOV` after other optimizations have run: this should ensure that 
the transformation is *not* altered. 

For other backends, there is a default 
implementation as `return (trueVal & ~condition) | (falseVal & condition)` with `condition={0,1}`.
Unlike the x86_64, this transformation happens after other IR passes have run, but before the backend passes have: this means IR 
passes will not alter the transformation, but there is a risk the backend optimizations will.

For more information, refer to the paper "[What you get is what you C: Controlling side effects in mainstream C compilers](https://drive.google.com/file/d/1jsOolD1C_Fu9oNVvhkB1_RQ9GlrFSGcN/view)".
 
The setup below was tested on Ubuntu trusty 14.04.5 LTS x86_64. I suggest you install this as a VM before reading further.

Pre-requesites:
---------------
	1. Install a VM running Ubuntu trusty 14.04.5 LTS x86_64. Allocate 32GB of disk.
	2. $sudo apt-get install git cmake g++ binutils-dev

Download, compile and install Clang/LLVM:
-----------------------------------------
	$git clone https://github.com/lmrs2/llvm.git -b ctchoose_38 --single-branch --depth 1 
	$cd llvm/tools
	$git clone https://github.com/lmrs2/clang.git -b ctchoose_38 --single-branch --depth 1 
	$cd ..
	$mkdir build
	$cd build
	$cmake -DLLVM_BINUTILS_INCDIR=/usr/include -DBUILD_SHARED_LIBS=ON -DCMAKE_BUILD_TYPE=Debug -DLLVM_TARGETS_TO_BUILD="X86" ../
	$cmake --build .
	$sudo make install

Now Clang has the new built-in function `__builtin_ct_choose()` which you can call. If you're on x86_64, it should be compiled into a `CMOV`.
