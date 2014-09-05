.. _simple_class:

#################
Objective-C Class
#################

Let's look at exactly how one defines a simple Objective-C class.

Although it's not absolutely required, the usual thing is to split the class into an ``@interface`` defined in a header file with the ``.h`` extension, and an ``@implementation`` defined in a file with the ``.m`` extension.  Here is:

``SimpleClass.h``

.. sourcecode:: objective-c

    #import <Foundation/Foundation.h>
    @interface SimpleClass: NSObject
    { NSString* name; }
    
    @property (retain) NSString *name;
    - (void)report;
    @end
    
    
I've scrunched it up to take less space.  Usually there would be more empty lines between items.  Anyway, it begins with an ``#import``, which is to obtain the definitions of ``NSObject`` and ``NSString``.  Then we have the rest of the code between ``@interface`` and ``@end``.

The first line declares the class name and its superclass ``NSObject``.

Then there are three sections.  First, a place where the variables that are visible to users of the class are declared.  Notice the brackets.  Deleting both of them leads to this error:

.. sourcecode:: bash

    In file included from SimpleClass.m:3:
    ./SimpleClass.h:4:12: error: cannot declare variable inside
          @interface or @protocol
     NSString* name; 
               ^
    1 error generated.

where it is not immediately obvious to me what is wrong.  Then we have:

.. sourcecode:: objective-c

    @property (retain) NSString *name;

Which declares that the variable ``name`` is a ``@property``, and that in memory management we want to ``retain`` it.

The last part is a pretty standard ``C``-like prototype for a single function.

.. sourcecode:: objective-c

    - (void)report;

I always forget the correct syntax here.  If there was an argument for this function, then it would look something like this:

.. sourcecode:: objective-c

    - (void)report:(NSString *) input;

The class definition is a bit longer.

``SimpleClass.m``

.. sourcecode:: objective-c

    #import "SimpleClass.h"
    @implementation SimpleClass : NSObject
    @synthesize name;

    - (id)init {
        self = [super init];
        if (self) {
            [self setName:@"SimpleClass"];
        }
        return self;
    }

    - (void)report { NSLog(@"%@", self.name); }
    @end

It is OK to repeat ``SimpleClass : NSObject``, but not necessary.

We have to import *our* header, and then between ``@implementation`` and ``@end`` is the actual implementation.  Notice the lack of brackets ``{ }`` around the class implementation.

Then, before the method implementations, there is this:

.. sourcecode:: objective-c

    @synthesize name;
    
which asks the compiler to construct a "getter" and a "setter" for the ``name`` variable.  Then we have an ``init`` method, which technically speaking, we don't need.  But this just shows how to do it.  I take the opportunity to give ``name`` a value and I use the synthesized method to do it.

And then we have the single method ``report`` which does something predictable.

When I first wrote this class I did

.. sourcecode:: objective-c

    - (void)report { NSLog(@"%@", self.name); }

but the self is not actually required.  I will have to investigate what difference this might make.

Finally, we want to actually use our class.  Usually, we would put the code into a separate file and ``import`` it.  Here, I add a ``main`` function outside of the class declaration, but in the same file ``SimpleClass.m``, and ``main`` will exercise the code.

.. sourcecode:: objective-c

    int main(void)
    {
        NSAutoreleasePool * pool;
        pool = [[NSAutoreleasePool alloc] init];
        id c = [[SimpleClass alloc] init];
        [c report];
        [pool drain];
        return 0;
    }

One detail is that more modern usage is to wrap the two lines we really need inside ``@autorelease { }``

.. sourcecode:: objective-c

    int main(void)
    {
        @autoreleasepool {
            id c = [[SimpleClass alloc] init];
            [c report];
        }
        return 0;
    }

I call:

.. sourcecode:: objective-c

    > clang SimpleClass.m -o prog -framework Foundation
    > ./prog
    2014-08-28 15:43:08.602 prog[2840:507] SimpleClass
    >
    
And that's it.

If you forget the ``-framework Foundation`` part you'll get a long error message about missing symbols from the linker.

To make this a little more realistic, let's put the main function in a separate file ``main.m``, compile our class first, then import it and instantiate it from main.

To do *that*, compile ``SimpleClass`` first like this:

.. sourcecode:: bash

    > clang SimpleClass.m -c

This will generate an object file ``SimpleClass.o``.  Now do this:

.. sourcecode:: bash

    > ar crl libsimple.a SimpleClass.o
    > ranlib libsimple.a
    > clang main.m -o prog -framework Foundation -L. -lsimple
    
You need first to use the archive tool to make a library ``.a`` class and then do ``ranlib`` on it.  Next we call the compiler and tell it to use our library, which is referred to by the shorthand name ``-lsimple`` (leaving out the ``ib``, if you will) and then to be sure to look for it in the current directory ``L.``.  

It works.

.. sourcecode:: bash

    > ./prog
    2014-08-28 15:58:31.646 prog[3006:507] SimpleClass
    >

This works, but these days it is considered bad form to use a static library.

For testing usage of Foundation and AppKit classes, it can be handier to have everything in one file.  So here it is:

``test.m``

.. sourcecode:: bash

    #import <Foundation/Foundation.h>
    @interface SimpleClass: NSObject
    { NSString* name; }

    @property (retain) NSString *name;
    - (void)report;
    @end

    @implementation SimpleClass
    @synthesize name;

    - (id)init {
        self = [super init];
        if (self) {
            [self setName:@"SimpleClass"];
        }
        return self;
    }

    - (void)report { NSLog(@"%@", self.name); }
    @end

    int main(int argc, char * argv[]) {
        @autoreleasepool {
            SimpleClass *sc = [[SimpleClass alloc] init];
            NSLog(@"%@", sc);
        }
        return 0;
    }
    