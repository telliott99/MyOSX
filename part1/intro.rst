.. _intro:

############
Command Line
############

The typical way to begin programming applications on OS X is to dive into Xcode.

But it's certainly possible to fiddle with Objective C constructs and test them on the command line.  And I like the clarity that brings, since it's absolutely clear what you've done.  Xcode has a lot of magic.  Later on we will have a lot more to say about Cocoa from Xcode.

Let's go back even further, and start with pure C.

Here is ``test.c``:

.. sourcecode:: bash

    #include <stdio.h>
    int main(int argc, char* argv []) {
        printf("Hello\n");
    return 0;
    }

.. sourcecode:: bash

    > gcc test.c -o prog
    > ./prog
    Hello
    >

We're using the standard C ``include`` to gain access to an input/output library.  We invoke the ``gcc`` compiler, which is a bit of a sham, because if you look

Since we're on OS X, we really should use the preferred compiler, which is ``clang``.  (I read somewhere, and by following the libraries that are loaded it appears to be the case, that the stock ``gcc`` somehow redirects to ``clang`` under Mavericks).

.. sourcecode:: bash

    > clang test.c -o prog
    > ./prog
    Hello
    >

If you're in a hurry you can combine them, but I usually don't.

.. sourcecode:: bash

    > clang test.c -o prog && ./prog
    Hello
    >

To do Objective C stuff, we change the file suffix to ``.m``, and switch to using ``#import`` (though ``#include`` will still work at this point).

``test.m``

.. sourcecode:: objective-c

    #import <stdio.h>
    int main(int argc, char *argv[]) {
        printf("Hello\n");
    return 0;
    }

.. sourcecode:: bash

    > clang test.m -o prog
    > ./prog
    Hello
    >

We can get access classes from the Objective C frameworks with ``import``.  For example, modify ``prog.m``

.. sourcecode:: objective-c

    #import <Foundation/Foundation.h>
    int main(int argc, char * argv[]) {
        NSLog(@"Hello %s", argv[1]);
        return 0;
    }

Now, we have to remember to tell the compiler what these new symbols (``NSLog``) mean.  The previous compilation call will get an error "Undefined symbols ..".  So modify it:

.. sourcecode:: bash

    > clang test.m -o prog -framework Foundation
    > ./prog
    2014-09-05 10:36:07.358 prog[4636:507] Hello (null)
    >

In the old days (see my old blog posts), it was optional whether you used garbage collection or managed memory manually.  Now the flag ``-fobjc-gc-only`` will give an error.

What we are supposed to do is to wrap nearly all of ``main`` in ``@autoreleasepool``, like this

#import <Foundation/Foundation.h>
int main(int argc, char * argv[]) {
    @autoreleasepool {
        NSLog(@"Hello %s", argv[1]);
    }
    return 0;
}

It doesn't matter for this simple program, but it would make a difference if we allocated an ``NSString``, for example.  

How about something a little more complicated, that also uses AppKit:

.. sourcecode:: objective-c

    #import <Foundation/Foundation.h>
    #import <AppKit/AppKit.h>

    int main(int argc, char * argv[]) {
        @autoreleasepool {
            NSLog(@"Hello %s", argv[0]);

            NSFont* font = [NSFont fontWithName:@"Arial" size:24.0];
            NSLog(@"%@", [font description]);
            NSMutableDictionary* D = [[NSMutableDictionary alloc] init];
            [D setObject:font forKey:@"NSFontAttributeName" ];
            NSColor* purple = [NSColor purpleColor];
            [D setObject:purple forKey:@"NSForegroundColorAttributeName" ];
            NSString* s = @"abc";
            NSLog(@"%@", [D description]);
        }
        return 0;
    }

.. sourcecode:: bash

    > clang test.m -o prog -framework Foundation \
     -framework AppKit
    > ./prog
    2014-09-05 10:41:52.063 prog[4690:507] Hello ./prog
    2014-09-05 10:41:52.081 prog[4690:507] "ArialMT 24.00 \
        pt. P [] (0x7ff9cac12cc0) fobj=0x7ff9cac15b80, spc=6.67"
    2014-09-05 10:41:52.082 prog[4690:507] {
        NSFontAttributeName = "\"ArialMT 24.00 pt. P [] \
            (0x7ff9cac12cc0) fobj=0x7ff9cac15b80, spc=6.67\"";
        NSForegroundColorAttributeName = \
           "NSCalibratedRGBColorSpace 0.5 0 0.5 1";
    }
    >

Now, we are ready to make something even more complicated.  Actually, some applications with a GUI can be built from the command line, but it's easier on everyone if we just use Xcode.  

So fire it up, and make a new project (Cocoa Application) called Hello.  Add a new Cocoa class called MyView (subclass of NSView) and paste this into the ``MyView.m`` file for the ``drawRect`` method that we're given:

.. sourcecode:: objective-c

    - (void)drawRect:(NSRect)dirtyRect {
        NSLog(@"drawRect");
        [super drawRect:dirtyRect];
        [[NSColor lightGrayColor] set];
        NSRectFill ( [self bounds] );
    
        NSRect r = NSMakeRect(50,50,50,50);
        NSBezierPath* p = [NSBezierPath bezierPathWithRect:r];
        [[NSColor redColor] set];
        [p fill];
    
        NSString *s = @"abc";
        NSMutableDictionary* D = [[NSMutableDictionary alloc] init];
        NSFont* font = [NSFont fontWithName:@"Arial" size:48.0];
        [D setObject:font forKey:NSFontAttributeName];
        [D setObject:[NSColor whiteColor] 
            forKey:NSForegroundColorAttributeName];
        [s drawAtPoint:NSMakePoint(50,150) withAttributes:D];
    }

The one other thing to do is go to the ``xib`` and see the layout.  Click on the window icon on the dock, then on the window itself until the description shows the window's view.  Change the class to ``MyView``.

Build and run (CMD-R) and you should get this:

.. image:: /figures/text.png
   :scale: 75 %

One can invoke the clang compiler from ``xcrun`` on the command line.  Here we instruct the compiler which SDK to use:

.. sourcecode:: objective-c

    > xcrun -sdk macosx clang test.m -o prog -framework Foundation -framework AppKit

It actually works without ``-sdk macosx``.  But one can specify an alternate SDK this way.