.. _NSProxy:

#######
NSProxy
#######

Here is some code adapted from the blog.  It works, with a warning that I don't know how to fix yet.  There is method for letting the objects know what they respond to, but I have still to read about it.

``talk.m``:

.. sourcecode:: objective-c

    // clang talk.m -o talk -framework Foundation
    #import <Foundation/Foundation.h>

    @interface VendedObject : NSObject {}
    -(NSString *) speak;
    @end

    @implementation VendedObject
    -(NSString *) speak { return @"woof"; }
    @end

    int main(){
        @autoreleasepool{
            VendedObject *obj;
            obj = [[VendedObject alloc ] init];
            NSLog(@"%@", [obj description]);
    
            NSConnection *conn;
            conn = [[NSConnection alloc] init];
            [conn setRootObject:obj];
            BOOL result;
            result = [conn registerName:@"my_server"];
            if (!result) {
                NSLog(@"Failed to register Name");
            }
            else {
                NSLog(@"%@", [conn description]);
            }
            [[NSRunLoop mainRunLoop] run];
        }
    }

``listen.m``:

.. sourcecode:: objective-c

    // clang listen.m -o listen -framework Foundation
    #import <Foundation/Foundation.h>

    int main(){
        @autoreleasepool{
            NSDistantObject *proxy_obj;
            proxy_obj = [NSConnection 
                rootProxyForConnectionWithRegisteredName:@"my_server" 
                                                    host:nil];
            if (!proxy_obj) {
                NSLog(@"Did not get an object from the server.");
            }
            else {
                NSLog(@"Received:  %@", [proxy_obj description]);
                NSLog(@"%@", [proxy_obj speak]);
            }
        }
    }

And the warning:

.. sourcecode:: bash

    > clang listen.m -o listen -framework Foundation
    listen.m:14:37: warning: instance method '-speak' not found
          (return type defaults to 'id') [-Wobjc-method-access]
                NSLog(@"%@", [proxy_obj speak]);
                                        ^~~~~
    /System/Library/Frameworks/Foundation.framework/Headers/NSDistantObject.h:9:12: note: 
          receiver is instance of class declared here
    @interface NSDistantObject : NSProxy <NSCoding> {
               ^
    1 warning generated.
    > 


.. sourcecode:: bash

    > ./talk
    2014-09-09 07:32:33.733 test[11838:507] <VendedObject: 0x7fc9c24062e0>
    2014-09-09 07:32:33.736 test[11838:507] \
    (** NSConnection 0x7fc9c27036b0 \
    receivePort <NSMachPort: 0x7fc9c2703af0> \
    sendPort <NSMachPort: 0x7fc9c2703af0> \
    refCount 1 remoteUsesKeyedDO: 0 **)

.. sourcecode:: bash

    > ./listen
    2014-09-09 07:32:37.873 listen[11843:507] \
    Received:  <VendedObject: 0x7fc9c24062e0>
    2014-09-09 07:32:37.875 listen[11843:507] woof
    >
