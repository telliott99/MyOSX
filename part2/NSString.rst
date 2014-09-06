.. _NSString:

########
NSString
########

Here is a working example showing basic exercise of NSString.

Here is the code (in a single file).  We'll go through the details after we see the whole thing.

``NSString.m``

.. sourcecode:: objective-c

    #import <Foundation/Foundation.h>

    int main(int argc, char * argv[]) {
        @autoreleasepool {
            char *cstr = "abc\0";
            NSStringEncoding ascii = NSASCIIStringEncoding;
            NSString* cs = [NSString stringWithCString:cstr
                                              encoding:ascii];
            NSString* fs = [NSString stringWithFormat:@"%4.2f", 3.1415];
            NSString* us = [NSString stringWithUTF8String:"\u2665"];
            NSString* us2 = [NSString stringWithUTF8String:"\xf0\x9f\x98\x83"];
            NSLog(@"%@ %@ %@ %@", cs, fs, us, us2);

            NSLog(@"%@", [cs stringByAppendingString:heart]);
            NSMutableString *ms = [[NSMutableString alloc]initWithCapacity:100];
            [ms appendString:cs];
            [ms insertString:heart atIndex:0];
            NSLog(@"%@", ms);

            [ms replaceOccurrencesOfString:@"B" 
                                     withString:@"*"
                                        options:NSCaseInsensitiveSearch
                                          range:NSMakeRange(1,3) ];
            NSLog(@"%@", ms);

            uint utf8 = NSUTF8StringEncoding;
            char b[] = { 0x61, 0x62, 0x63, 0x64, 0x65 };
            NSData *d = [NSData dataWithBytes:b length:5];
            s = [[NSString alloc] initWithData:d encoding:utf8];
            NSLog(@"%@", s);

            NSString* s = @"abcdefghijklmnopqrstuvwxyz";
            NSString* t = @"efg";
            NSString* heart = @"â™¥";
        }
        return 0;
    }
    
Here is what we see from the command line:

.. sourcecode:: bash

    > clang NSString.m -o prog -framework Foundation
    > ./prog
    2014-09-05 20:24:21.043 prog[8555:507] abc 3.14 â™¥ í ½í¸ƒ
    2014-09-05 20:24:21.046 prog[8555:507] abcâ™¥
    2014-09-05 20:24:21.046 prog[8555:507] â™¥abc
    2014-09-05 20:24:21.047 prog[8555:507] â™¥a*c
    2014-09-05 20:24:21.048 prog[8555:507] abcde
    >
    
So what did we do?  In the first part, we had

.. sourcecode:: objective-c

    char *cstr = "abc\0";
    NSStringEncoding ascii = NSASCIIStringEncoding;
    NSString* cs = [NSString stringWithCString:cstr
                                      encoding:ascii];
    NSString* fs = [NSString stringWithFormat:@"%4.2f", 3.1415];
    NSString* us = [NSString stringWithUTF8String:"\u2665"];
    NSString* us2 = [NSString stringWithUTF8String:"\xf0\x9f\x98\x83"];
    NSLog(@"%@ %@ %@ %@", cs, fs, us, us2);

We made a null-terminated C string using ``char *cstr`` and then converted it to an NSString.  We used ``stringWithFormat`` with a floating point number.  And then we used the decimal value for a Unicode code point or the actual UTF-8 encoding to make string for two unusual characters.  Printing gives us:

.. sourcecode:: objective-c

    2014-09-05 20:50:51.653 prog[8765:507] abc 3.14 â™¥ í ½í¸ƒ

We see that the Unicode character ``U+2665`` is a "spade".  This is actually its hex value, the decimal value is 9829.  And the value as encoded in UTF-8 would be ``e2 99 a5``.

http://www.fileformat.info/info/unicode/char/2665/index.htm

After that we do a smiley face. 

In the middle part we exercise ``NSMutableString`` with ``insertString:heart atIndex:0`` and replacing part of it.  We convert bytes into a string and print it:

.. sourcecode:: bash

    2014-09-05 20:56:30.096 prog[8800:507] abcde

Here is a selection of NSString methods:

    - ``compare:options:range:``
    - ``hasPrefix:``
    - ``hasSuffix:``
    - ``isEqualToString:``
    
    - ``rangeOfString:``
    - ``rangeOfString:options:range:``
    
    - ``componentsSeparatedByString:``
    - ``componentsSeparatedByCharactersInSet:``
    - ``substringFromIndex:``
    - ``substringWithRange:``

For the most part, usage is pretty clear.

.. sourcecode:: objective-c

    enum {
       NSCaseInsensitiveSearch = 1,
       NSLiteralSearch = 2,
       NSBackwardsSearch = 4,
       NSAnchoredSearch = 8,
       NSNumericSearch = 64,
       NSDiacriticInsensitiveSearch = 128,
       NSWidthInsensitiveSearch = 256,
       NSForcedOrderingSearch = 512,
       NSRegularExpressionSearch = 1024
    };
    
.. sourcecode:: objective-c

    enum {
       NSOrderedAscending = -1,
       NSOrderedSame,
       NSOrderedDescending
    };
    typedef NSInteger NSComparisonResult;
    
Let's do a few:

.. sourcecode:: objective-c

    #import <Foundation/Foundation.h>

    int main(void)
    {
        @autoreleasepool {
            NSString* s;
            s = [NSString stringWithFormat:@"abcd\nefghi"];
            NSArray *a = [s componentsSeparatedByString:@"\n"];
            for (NSString *s in a) { NSLog(@"%@", s); }

            NSCharacterSet *cs;
            cs = [NSCharacterSet characterSetWithCharactersInString:@"#"];
            s = @"abc#def#ghi";
            a = [s componentsSeparatedByCharactersInSet:cs];
            for (NSString *s in a) { NSLog(@"%@", s); }
        }
        return 0;
    }
    
Results are as you would expect:

.. sourcecode:: bash

    > clang main.m -o prog -framework Foundation -framework AppKit
    > ./prog
    2014-09-06 08:55:21.118 prog[10662:507] abcd
    2014-09-06 08:55:21.120 prog[10662:507] efghi
    2014-09-06 08:55:21.121 prog[10662:507] abc
    2014-09-06 08:55:21.121 prog[10662:507] def
    2014-09-06 08:55:21.121 prog[10662:507] ghi
    >

Except, I couldn't get this one to work:

    - ``stringByTrimmingCharactersInSet:``

A bit more:

.. sourcecode:: objective-c

    NSStringCompareOptions opt;
    opt = NSCaseInsensitiveSearch;
    if ([@"cd" compare:@"CD" options:opt] == NSOrderedSame) {
        NSLog(@"yes");
        // yes
    }

    NSRange r = NSMakeRange(2,2);
    s = @"abcdefghi";
    NSLog(@"%@", [s substringWithRange:r]);
    r = [s rangeOfString:@"gh"];
    NSLog(@"%lu %lu", r.location, r.length);

.. sourcecode:: bash

    2014-09-06 09:01:36.866 prog[10749:507] yes
    2014-09-06 09:01:36.866 prog[10749:507] cd
    2014-09-06 09:01:36.867 prog[10749:507] 6 2
