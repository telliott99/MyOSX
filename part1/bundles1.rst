.. _bundles1:

#######
Bundles
#######

This is taken from a post I put here:

http://telliott99.blogspot.com/2011/01/os-x-frameworks-bundles-1.html

It is for a simple "Foundation tool" that loads code from a bundle at runtime.  It makes you appreciate the whole Framework approach, because the natural question is, once I've got it working, where do I put the bundle? 

You can put it in Library/Application Support or something, but it's an issue. The example also introduces the notion of a Protocol.

The code listing for 3 files is given below. The first two, ``BundlePrinter.h`` and ``SimpleMessage.m`` go into a new Xcode project: a Cocoa Bundle named SimpleMessage.

Here is the code for ``BundlePrinter.h``, which simply defines the BundlePrinter Protocol.

.. sourcecode:: objective-c

    @protocol BundlePrinterProtocol
    + (BOOL)activate;
    + (void)deactivate;
    - (NSString *)message;
    @end

What this means is that if you (as a class) are going to follow the protocol, you need to implement three methods with these signatures.  Simple.

The second piece of code is ``SimpleMessage.m``

.. sourcecode:: objective-c

    #import <Foundation/Foundation.h>
    #import "BundlePrinter.h"
    @interface SimpleMessage:NSObject <BundlePrinterProtocol> {}
    @end
    @implementation SimpleMessage
    + (BOOL)activate {
        NSLog(@"SimpleMessage plug-in activated");
        return (YES);
    }
    + (void)deactivate {
        NSLog(@"SimpleMessage plug-in deactivated");
    }
    - (NSString *)message {
        return (@"This is a simple message");
    } @end

The ``<BundlePrinterProtocol>`` says yes, we will follow the protocol.  And the implementation does just that.  To check that we fulfill our obligations, the compiler needs to see ``BundlePrinter.h``.

In Xcode, make a new project that is a bundle called ``SimpleMessage``.  Copy both the header and the implementation  ``SimpleMessage.m`` into the project.  Under Supporting Files, find the ``info.plist`` and set the key: PrincipalClass to SimpleMessage.  Now build it.

Find what we built.  Xcode hides it away these days.  Click on SimpleMessage.bundle in the FileView under Products.  Go to File > Show in Finder, and then Option-drag the bundle to the Desktop.

For the second part of the tutorial, make a new Command Line tool called BundlePrinter.   Copy ``BundlePrinter.h`` into the Project.

Modify ``main.c`` from the project as follows..

Add this to the top of the file:

.. sourcecode:: objective-c

    #import <Foundation/Foundation.h>
    #import "BundlePrinter.h"

    NSString *processPlugin(NSString *path) {
        NSBundle *plugin;
        Class principalClass;
        id pluginInstance;
        NSString *message = nil;
        NSLog(@"processing plug-in: %@", path);
        plugin = [NSBundle bundleWithPath:path];
        if (plugin == nil) {
            NSLog(@"count not load plug-in at path %@",path);
            goto bailout;
        }
        principalClass = [plugin principalClass];
        if (principalClass == nil) {
            NSLog(@"could not load principal class for plug-in at path %@",path);
            goto bailout;
        }
        if (![principalClass conformsToProtocol:
              @protocol(BundlePrinterProtocol)])
            NSLog(@"plug-in must conform to the BundlePrinterProtocol");
        if (![principalClass activate]) {
            NSLog(@"could not activate class for plug-in at path %@", path);
            goto bailout;
        }
        pluginInstance = [[principalClass alloc] init];
        message = [pluginInstance message];
        //[pluginInstance release];
        [principalClass deactivate];
    bailout:
        return message;
    }

It feels a little weird to be using ``goto``  :)

I had to edit the code to get rid of a line near the end.  We handle memory differently these days.

Modify the actual function ``main.m`` to add the following code in ``main``:

.. sourcecode:: objective-c

    NSDirectoryEnumerator *enumerator;
    NSString *path, *message;
    NSString *home = NSHomeDirectory();
    NSString *desktop = [home stringByAppendingString:@"/Desktop/"];
    enumerator = [[NSFileManager defaultManager] enumeratorAtPath:desktop];
    while (path = [enumerator nextObject]) {
        if ([[path pathExtension] isEqualToString:@"bundle"]) {
            NSString *actualPath = [desktop stringByAppendingString:path];
            message = processPlugin(actualPath);
            if (message != nil) {
                //printf("\nmessage is: '%s'\n\n", [message cString]);
                NSLog(@"\nmessage is: '%@'\n\n", message);
            }
        }   
    }

I modified this code too from the previous version, to search the Desktop, which is where we left our bundle.  When I build and run it, I get this:

.. sourcecode:: bash

    014-08-29 18:54:08.172 BundlePrinter[3098:303] processing plug-in: /Users/telliott_admin/Desktop/SimpleMessage.bundle
    2014-08-29 18:54:08.204 BundlePrinter[3098:303] SimpleMessage plug-in activated
    2014-08-29 18:54:08.206 BundlePrinter[3098:303] SimpleMessage plug-in deactivated
    2014-08-29 18:54:08.207 BundlePrinter[3098:303] 
    message is: 'This is a simple message'

    Program ended with exit code: 0

It works!

The complete listing for the ``main`` function is:

.. sourcecode:: objective-c

    int main(int argc, const char * argv[]) {
        @autoreleasepool {
            NSDirectoryEnumerator *enumerator;
            NSString *path, *message;
            NSString *home = NSHomeDirectory();
            NSString *desktop = [home stringByAppendingString:@"/Desktop/"];
            enumerator = [[NSFileManager defaultManager] enumeratorAtPath:desktop];
            while (path = [enumerator nextObject]) {
                //NSLog(@"here %@", path);
                if ([[path pathExtension] isEqualToString:@"bundle"]) {
                    NSString *actualPath = [desktop stringByAppendingString:path];
                    message = processPlugin(actualPath);
                    if (message != nil) {
                        //printf("\nmessage is: '%s'\n\n", [message cString]);
                        NSLog(@"\nmessage is: '%@'\n\n", message);
                    }
                } }
        }
        return 0;
    }
