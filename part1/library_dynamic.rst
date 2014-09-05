.. _library_dynamic:

###############
Dynamic Library
###############

Alternatively, we can make a dynamic library.

.. sourcecode:: bash

    > clang -dynamiclib -current_version 1.0 add*.o -o libadd.dylib
    > file libadd.dylib
    libadd.dylib: Mach-O dynamically \
    linked shared library i386
    > otool -L libadd.dylib
    libadd.dylib:
    	libadd.dylib (compatibility version 0.0.0, \
    	current version 1.0.0)
    	/usr/lib/libSystem.B.dylib (compatibility version 1.0.0, \
    	current version 125.2.11)
    >

Find out more about ``otool`` by doing

.. sourcecode:: bash

    man otool | col -b > result.txt

So try some options

.. sourcecode:: bash

    > otool -tvV "libadd.a(add1.o)"
    libadd.a(add1.o):
    (__TEXT,__text) section
    _f1:
    00000000	pushl	%ebp
    00000001	movl	%esp,%ebp
    00000003	subl	$0x18,%esp
    00000006	calll	0x0000000b
    0000000b	popl	%eax
    0000000c	movl	0x08(%ebp),%ecx
    0000000f	movl	%ecx,0xfc(%ebp)
    00000012	movl	0xfc(%ebp),%ecx
    00000015	movl	%esp,%edx
    00000017	movl	%ecx,0x04(%edx)
    0000001a	leal	0x1fe-0xb(%eax),%eax
    00000020	movl	%eax,(%edx)
    00000022	calll	_printf+0x100000000
    00000027	movl	0xfc(%ebp),%ecx
    0000002a	addl	$0x01,%ecx
    0000002d	movl	%eax,0xf8(%ebp)
    00000030	movl	%ecx,%eax
    00000032	addl	$0x18,%esp
    00000035	popl	%ebp
    00000036	ret
    >

Now that we have a dynamic library, use it like this:

.. sourcecode:: bash

    > clang -c useadd.c
    > clang useadd.o ./libadd.dylib -o useadd
    > ./useadd
    f1: 1;  main 2
    f2: 10;  main 12

Perhaps you are suspicious that we are still using the old ``useadd`` (I deleted it) or ``libadd.a`` somehow.  Snoop on the loader:

.. sourcecode:: bash

    > export DYLD_PRINT_LIBRARIES=1
    > ./useadd
    dyld: loaded: /Users/telliott_admin/Desktop/./useadd
    dyld: loaded: /Users/telliott_admin/Desktop/libadd.dylib
    dyld: loaded: /usr/lib/libSystem.B.dylib
    dyld: loaded: /usr/lib/system/libmathCommon.A.dylib
    f1: 1;  main 2
    f2: 10;  main 12
    >

We can see that ``libadd.dylib`` is loaded.  Also

.. sourcecode:: bash

    > nm useadd
    00001f72 s  stub helpers
    0000202c D _NXArgc
    00002030 D _NXArgv
    00002038 D ___progname
    00001000 A __mh_execute_header
    00002034 D _environ
             U _exit
             U _f1
             U _f2
    00001ec0 T _main
             U _printf
    00002000 s _pvars
             U dyld_stub_binder
    00001e80 T start
    >

``_f1`` and ``_f2`` are now listed as ``U`` rather than ``T``.  ``T`` means the symbol is in the text (code) section.  ``U`` means that it is undefined (it will be loaded when needed). 