.. _framework:

#########
Framework
#########

In Xcode start a new project: Framework in Objective C; I named it Adder and put it on my Desktop.

Drag ``add1.c`` and ``add2.c`` (from the previous chapters on Libraries) into the Adder folder (in the FileView) on the left. Here is ``add1.c`` again for reference:

.. sourcecode:: c

    #include <stdio.h>
    int f1(int x) {
        printf( "f1: %d;", x );
        return x+1;
    }

Xcode has automatically created a header file for you ``Adder.h``.  Write declarations of the two functions into that file.

.. sourcecode:: c

    int f1(int x);
    int f2(int x);
    
(or add the ``.h`` files to the project and put import statements in ``Adder.h``).
    
With ``Adder.h`` selected in the FileView, at the right-hand side make sure that under "Target Membership", Adder is set to Public.

.. image:: /figures/fw1.png
   :scale: 100 %

Do Build For Running.  The result is hidden away.  To find it, select ``Adder.framework`` (under Products) in the FileView and do File > Show in Finder.  CTL-drag a copy of the Adder.framework file to the Desktop.  

Now we can build ``useadd`` and link it against the Adder framework

.. sourcecode:: bash

    clang -g -o useadd -F. -framework Adder useadd.c

It will run:

.. sourcecode:: bash

    > ./useadd
    f1: 1;  main 2
    f2: 10;  main 12
    >

But it's better to put ``Adder.framework`` where it belongs:

.. sourcecode:: bash

    mv Adder.framework/ ~/Library/Frameworks/Adder.framework

``useadd`` still works

.. sourcecode:: bash

    > ./useadd
    f1: 1;  main 2
    f2: 10;  main 12
    >

Check the libraries as they load:

.. sourcecode:: bash

    > export DYLD_PRINT_LIBRARIES=1
    > ./useadd
    dyld: loaded: /Users/telliott_admin/Desktop/./useadd
    dyld: loaded: /Users/telliott_admin/Library/Frameworks/Adder.framework/Versions/A/Adder
    dyld: loaded: /usr/lib/libSystem.B.dylib
    ..
    dyld: loaded: /usr/lib/libobjc.A.dylib
    dyld: loaded: /usr/lib/libauto.dylib
    dyld: loaded: /usr/lib/libc++abi.dylib
    dyld: loaded: /usr/lib/libc++.1.dylib
    dyld: loaded: /usr/lib/libDiagnosticMessagesClient.dylib
    f1: 1;  main 2
    f2: 10;  main 12
    >

An alternative approach would be to select the listing for ``Adder.framework`` that Xcode puts in the dialog that comes with Show in Finder, and drag that line into TextEdit to reveal something like:

.. sourcecode:: bash

    /Users/telliott_admin/Library/Developer/Xcode\
    /DerivedData/Adder-fmzjjwrmijnhgzcppfwdmxrdiply\
    /Build/Products/Debug/Adder.framework

Then define ``ADDER_FRAMEWORK_PATH={that path}`` 

whatever it is, and then build with 

.. sourcecode:: bash

    clang -g -o useadd -F${ADDER_FRAMEWORK_PATH} -framework Adder useadd.c

But you'll still need to put it somewhere it can be found at runtime.  Maybe the best solution is to build the program with the Framework inside it.  That's the last example from my series of posts:

http://telliott99.blogspot.com/2011/01/os-x-frameworks-bundles-3.html

We could use ``Adder.framework``, but let's follow this example.  We will have a framework with a single very useful class.  The header:

``Stuff.h``

.. sourcecode:: objective-c

    #import <Cocoa/Cocoa.h>
    @interface Stuff : NSObject
    { }
    + (void) doStuff;
    @end

The implementation 

``Stuff.m``

.. sourcecode:: objective-c

    #import "Stuff.h"
    @implementation Stuff
    + (void) doStuff
    { NSLog(@"doing stuff."); }
    @end

Make a new Xcode project: Cocoa Framework and drag in ``Stuff.m`` under Classes. Xcode makes a header for you, so add the code from Stuff.h to the header they give. 

Select the SimpleFramework target, and then click on Stuff.h and make sure it says "public" (upper panel on extreme right-hand side).  Build it.

Make sure it says SimpleFramework

The old instructions say

Then get the Inspector for the target (double-click on SimpleFramework under the Targets pane) and under the Build tab > Deployment > Installation Directory type @executable_path/../Frameworks. Build the framework.

but I can't find this.

Show in Finder..

The third code snippet is for ``main`` in a new Xcode project:  a Cocoa application named SimpleApp.

.. sourcecode:: objective-c

    #import <Cocoa/Cocoa.h>
    #import <SimpleFramework/Stuff.h>

    int main(int argc, const char * argv[]) {
        [Stuff doStuff];
        return NSApplicationMain(argc, argv);
    }

Drag the Simple.framework into this project

.. sourcecode:: bash

    dyld: Library not loaded: @rpath/Stuff.framework/Versions/A/Stuff
      Referenced from: /Users/telliott_admin/Library/Developer/Xcode/DerivedData/SimpleApp-dpctprrfhrnwsvcwwetnwaagsbfx/Build/Products/Debug/SimpleApp.app/Contents/MacOS/SimpleApp
      Reason: image not found


