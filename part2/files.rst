.. _files:

####################
Read and Write Files
####################

.. sourcecode:: objective-c

    // clang main.m -o prog -framework Foundation

    #import <Foundation/Foundation.h>

    int main (int argc, const char *argv[]) {
        @autoreleasepool {
            NSString *s = @"my data\n";
            NSError *e = nil;
            // [s writeToFile:@"/does/not/exist/x.txt"
            [s writeToFile:@"x.txt"
              atomically:YES
              encoding:NSUTF8StringEncoding
              error:&e ];
            if (e != nil) {
                NSLog(@"%@", [e description]);
            }
        }   
        return 0;
    }

.. sourcecode:: bash

    > clang main.m -o prog -framework Foundation
    > ./prog
    2014-09-13 09:39:34.891 prog[2814:507] \
    Error Domain=NSCocoaErrorDomain Code=4 \
    "The folder “x.txt” doesn’t exist." \
    UserInfo=0x7fa72a40b670 {NSFilePath=/does/not/exist/x.txt, \
    NSUserStringVariant=Folder, \
    NSUnderlyingError=0x7fa72a40b220 \
    "The operation couldn’t be completed. No such file or directory"}
    >

Add this:

.. sourcecode:: objective-c

    e = nil;
    NSString *str = [[NSString alloc] 
        initWithContentsOfFile:@"x.txt"
                      encoding:NSUTF8StringEncoding
                         error:&e];
    if (e == nil) {
           NSLog(@"%@", str);
    }

.. sourcecode:: bash

    > ./prog
    2014-09-13 09:44:18.425 prog[2843:507] my data
    >

And now do a fetch on the web and write the data to disk.  Here is the whole listing for ``main.m``:

.. sourcecode:: objective-c

    // clang main.m -o prog -framework Foundation

    #import <Foundation/Foundation.h>

    int main (int argc, const char *argv[]) {
        @autoreleasepool {
            NSString *s = @"my data\n";
            NSError *e = nil;
            // [s writeToFile:@"/does/not/exist/x.txt"
            [s writeToFile:@"x.txt"
              atomically:YES
              encoding:NSUTF8StringEncoding
              error:&e ];
            if (e != nil) {
                NSLog(@"%@", [e description]);
            }
        
            e = nil;
            NSString *str = [[NSString alloc] 
                initWithContentsOfFile:@"x.txt"
                              encoding:NSUTF8StringEncoding
                                 error:&e];
            if (e == nil) {
                   NSLog(@"%@", str);
            }
        
            NSString *s2 = @"http://www.google.com/images/logos/ps_logo2.png";
            NSURL *url = [NSURL URLWithString:s2];
            NSURLRequest *uq = [NSURLRequest requestWithURL:url];
            e = nil;
            NSData *data = [NSURLConnection 
                sendSynchronousRequest:uq
                     returningResponse:NULL
                                 error:&e];
            if (!data) {
                NSLog(@"no data %@", [e localizedDescription]);
            }
            else {
                [data writeToFile:@"y.png"
                          options:0
                            error:&e];
            }
         }   
        return 0;
    }

(or maybe:  ``options:NSDataWritingAtomic``)

.. sourcecode:: bash

    > hexdump -C -n 16 y.png
    00000000  89 50 4e 47 0d 0a 1a 0a  00 00 00 0d 49 48 44 52  |.PNG........IHDR|
    00000010
    > hexdump -C -n 64 y.png
    00000000  89 50 4e 47 0d 0a 1a 0a  00 00 00 0d 49 48 44 52  |.PNG........IHDR|
    00000010  00 00 01 6c 00 00 00 7e  08 02 00 00 00 8f 94 bc  |...l...~........|
    00000020  ca 00 00 66 13 49 44 41  54 78 da ed bd 77 9c 25  |...f.IDATx...w.%|
    00000030  57 75 27 7e ce bd 55 f5  f2 eb d7 39 87 e9 49 ca  |Wu'~..U....9..I.|
    00000040
    >
    
.. image:: /figures/y.png
   :scale: 100 %
