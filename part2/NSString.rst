.. _NSString:

########
NSString
########

In this chapter, we'll be doing some basic exercises with NSString.

However, by far the most common thing I want to look up for an NSString is how to split it on newlines, so let's do that first.

``main.m``

.. sourcecode:: objective-c

    #import <Foundation/Foundation.h>

    int main(int argc, char * argv[]) {
        @autoreleasepool {
            NSString *s = @"abc\ndef\nghi";
            NSArray *a = [s componentsSeparatedByString:@"\n"];
            for (s in a) { NSLog(@"%@", s); }
        }
        return 0;
    }

.. sourcecode:: bash

    > clang main.m -o prog -framework Foundation -framework AppKit
    > ./prog
    2014-09-06 09:42:20.143 prog[11200:507] abc
    2014-09-06 09:42:20.145 prog[11200:507] def
    2014-09-06 09:42:20.145 prog[11200:507] ghi
    >

With that out of the way, we can look at construction of strings.

``string.m``

.. sourcecode:: objective-c

    #import <Foundation/Foundation.h>

    int main(int argc, char * argv[]) {
        @autoreleasepool {
            char *cstr = "abc\0";
            NSStringEncoding ascii = NSASCIIStringEncoding;
            NSString* cs = [NSString stringWithCString:cstr
                                              encoding:ascii];
            NSString* fs = [NSString stringWithFormat:@"%4.2f", 3.1415];
            NSString* heart = [NSString stringWithUTF8String:"\u2665"];
            NSString* us;
            us = [NSString stringWithUTF8String:"\xf0\x9f\x98\x83"];
            NSLog(@"%@ %@ %@ %@", cs, fs, heart, us);
        }
        return 0;
    }

.. sourcecode:: bash

    > clang string.m -o prog -framework Foundation -framework AppKit
    > ./prog
    2014-09-06 09:51:56.408 prog[11305:507] abc 3.14 â™¥ í ½í¸ƒ
    >

So what did we do?  We made a null-terminated C string using ``char *cstr`` and then converted it to an NSString.  We used ``stringWithFormat`` with a floating point number.  And then we used the decimal value for a Unicode code point or the actual UTF-8 encoding to make a string for two unusual characters.

Notice that ``stringWithUTF8String`` does not take an "object", there is no ``@`` before the string.

We see that the Unicode character ``U+2665`` is a "spade".  This is actually its hex value, the decimal value is 9829.  And the value as encoded in UTF-8 would be ``e2 99 a5``.

http://www.fileformat.info/info/unicode/char/2665/index.htm

Finally, we do a smiley face. 

Now let's do some more manipulations:

.. sourcecode:: objective-c

    NSString* heart = [NSString stringWithUTF8String:"\u2665"];
    NSString *s = @"abc";
    NSLog(@"%@", [s stringByAppendingString:heart]);
    NSMutableString *ms;
    ms = [[NSMutableString alloc]initWithCapacity:100];
    [ms appendString:s];
    [ms insertString:heart atIndex:1];
    NSLog(@"%@", ms);

    [ms replaceOccurrencesOfString:@"B" 
                        withString:@"*"
                           options:NSCaseInsensitiveSearch
                             range:NSMakeRange(1,3) ];
    NSLog(@"%@", ms);

.. sourcecode:: bash

    > clang string.m -o prog -framework Foundation -framework AppKit
    > ./prog
    2014-09-06 09:53:06.809 prog[11323:507] abcâ™¥
    2014-09-06 09:53:06.811 prog[11323:507] aâ™¥bc
    2014-09-06 09:53:06.812 prog[11323:507] aâ™¥*c
    >

And then, we'll turn some data into an NSString.  Also we see that we can just paste any UTF-8 data into a string literal and it'll work:

.. sourcecode:: objective-c

    uint utf8 = NSUTF8StringEncoding;
    char b[] = { 0x61, 0x62, 0x63, 0x64, 0x65 };
    NSData *d = [NSData dataWithBytes:b length:5];
    NSString *s;
    s = [[NSString alloc] initWithData:d encoding:utf8];
    NSLog(@"%@", s);
    NSString* heart = @"â™¥";
    NSLog(@"%@", heart);
    
.. sourcecode:: bash

    > clang string.m -o prog -framework Foundation -framework AppKit
    > ./prog
    2014-09-06 09:56:57.716 prog[11344:507] abcde
    2014-09-06 09:56:57.718 prog[11344:507] â™¥
    >

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
    - ``UTF8String``

For the most part, usage is pretty straightforward.

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
    
Let's try a few:

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

One that I couldn't get to work:

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

It's recommended that you use C strings as little as possible, but you can do it

.. sourcecode:: objective-c

    #import <Foundation/Foundation.h>

    int main(int argc, char * argv[]) {
        @autoreleasepool {
            char *cstr = "abc\0";
            NSStringEncoding ascii = NSASCIIStringEncoding;
            NSString* s = [NSString stringWithCString:cstr
                                             encoding:ascii];
            const char *cstr2 = [s UTF8String];
            printf("%s\n", cstr2);
        }
        return 0;
    }
    
.. sourcecode:: bash

    > clang main.m -o prog -framework Foundation
    > ./prog
    abc
    >