.. _NSProxy:

#######
NSProxy
#######

Here is some code adapted from the blog.  With help from


http://www.informit.com/articles/article.aspx?p=1438422&seqNum=2

I learned about ``inout``, which silenced a warning.

``talk.m``:

.. sourcecode:: objective-c

    // clang talk.m -o talk -framework Foundation
    #import <Foundation/Foundation.h>

    @interface VendedObject : NSObject {}
    -(inout NSString *) speak;
    @end

    @implementation VendedObject
    -(inout NSString *) speak { 
        return @"woof"; 
    }
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
            NSConnection *c;
            c = [NSConnection connectionWithRegisteredName:@"my_server"
                                                     host:nil];
            NSDistantObject *proxy_obj = [c rootProxy];
            if (!proxy_obj) {
                NSLog(@"Did not get an object from the server.");
            }
            else {
                NSLog(@"Received:  %@", [proxy_obj description]);
                NSLog(@"%@", [proxy_obj speak]);
            }
        }
    }

The warning came in compiling ``listen.m``:

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

The modification is to ``talk.m``, adding ``inout``:

.. sourcecode:: objective-c

    -(inout NSString *) speak;

Run the two in separate tabs of Terminal:

.. sourcecode:: bash

    > ./talk
    2014-09-10 07:07:37.927 talk[2267:507] \
    <VendedObject: 0x7fcdd2c08050>
    2014-09-10 07:07:37.931 talk[2267:507] \
    (** NSConnection 0x7fcdd2e033c0 
    receivePort <NSMachPort: 0x7fcdd2e03800> \
    sendPort <NSMachPort: 0x7fcdd2e03800> \
    refCount 1 remoteUsesKeyedDO: 0 **)
    

.. sourcecode:: bash

    > ./listen
    2014-09-10 07:07:42.510 listen[2272:507] \
    Received:  <VendedObject: 0x7fcdd2c08050>
    2014-09-10 07:07:42.512 listen[2272:507] woof
    >
