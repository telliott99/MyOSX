.. _NSDate:

######
NSDate
######

The C struct tm
---------------

Also from Hillegass, exploring use of the struct ``tm``.  We can get the number of seconds since the "epoch" with ``time(NULL)``.  To use ``localtime_r`` we pass the addresses of a long holding the current time (in seconds since 1970) and a struct of type ``tm`` named ``now``.  The struct's fields get filled in with the appropriate values.

``main.c``:

.. sourcecode:: objective-c

    // clang main.c -o prog
    #include <stdio.h>
    #include <time.h>

    int main (int argc, const char *argv[]) {
        long s = time(NULL);
        printf("%ld seconds since 1970\n", s);
        struct tm now;
        localtime_r(&s, &now);
        printf("The time is %d:%d:%d\n", 
            now.tm_hour, now.tm_min, now.tm_sec);
        return 0;
    }

.. sourcecode:: bash

    > clang main.c -o prog
    > ./prog
    1410606528 seconds since 1970
    The time is 7:8:48
    >

Looks fine except for a small formatting issue with the minutes.  How about using ``NSDate``?

``main.m``:

.. sourcecode:: objective-c

    // clang main.m -o prog -framework Foundation

    #import <Foundation/Foundation.h>

    int main (int argc, const char *argv[]) {
        @autoreleasepool {
            NSDate *right_now = [NSDate date];
            NSLog(@"address of var:  %p", right_now);
            NSLog(@"time is:  %@", right_now);
            NSLog(@"since 1970:  %f", 
                [right_now timeIntervalSince1970]);
        }
        return 0;
    }
    
.. sourcecode:: bash
 
    > clang main.m -o prog -framework Foundation
    > ./prog
    2014-09-13 07:10:54.432 prog[1993:507] address of var:  0x7fb492408f10
    2014-09-13 07:10:54.443 prog[1993:507] time is:  2014-09-13 11:10:54 +0000
    2014-09-13 07:10:54.443 prog[1993:507] since 1970:  1410606654.432290
    >

Note the issue with the time zone.  (We are at +4000, Eastern Daylight Time, so the time is off by 4 hours).

Reading this

http://stackoverflow.com/questions/8866515/set-nsdate-timezone

I add this:

.. sourcecode:: objective-c

    NSDateFormatter *dateFormatter = [[NSDateFormatter alloc] init];
    NSTimeZone *tz = [NSTimeZone timeZoneWithName:@"Eastern"];
    NSLog(@"%@", [right_now descriptionWithCalendarFormat:nil 
        timeZone:tz 
          locale:nil]);

And now we get

.. sourcecode:: bash

    > clang main.m -o prog -framework Foundation
    > ./prog
    2014-09-13 07:15:29.659 prog[2035:507] address of var:  0x7fbd61408f50
    2014-09-13 07:15:29.667 prog[2035:507] time is:  2014-09-13 11:15:29 +0000
    2014-09-13 07:15:29.667 prog[2035:507] since 1970:  1410606929.659410
    2014-09-13 07:15:29.668 prog[2035:507] 2014-09-13 07:15:29 -0400
    >

NSCalendarDate:

Add this

.. sourcecode:: objective-c

    NSCalendar *cal = [NSCalendar currentCalendar];
    unsigned long day;
    day = [cal ordinalityOfUnit:NSDayCalendarUnit
                         inUnit:NSMonthCalendarUnit
                        forDate:right_now ];
    NSLog(@"day of the month: %lu", day);

and get this:

.. sourcecode:: bash

    2014-09-13 07:22:06.085 prog[2101:507] day of the month: 13

Finally, I do the challenge (determine how many seconds I've been alive, assuming I was born yesterday).

.. sourcecode:: objective-c

    NSDateComponents *dc = [[NSDateComponents alloc] init];
    [dc setYear:2014];
    [dc setMonth:9];
    [dc setDay:12];
    [dc setHour:7];
    [dc setMinute:32];
    [dc setSecond:0];
    NSCalendar *g = [[NSCalendar alloc] 
        initWithCalendarIdentifier:NSGregorianCalendar];
    NSDate *dob = [g dateFromComponents:dc];
    NSLog(@"%f", [right_now timeIntervalSinceDate:dob]);

.. sourcecode:: bash

    2014-09-13 07:32:25.489 prog[2170:507] 86425.479177
    > python 
    Python 2.7.8 (default, Jul  2 2014, 10:14:46) 
    [GCC 4.2.1 Compatible Apple LLVM 5.1 (clang-503.0.40)] on darwin
    Type "help", "copyright", "credits" or "license" for more information.
    >>> 3600*24
    86400
    >>>

Looks about right.  The current time is ``07:32:25.489`` and we set the ``dob`` to be 
0 seconds past the hour, hence we obtain ``86425`` rather than ``86400``.

Notice that the month, etc. are 1-based indexing.
