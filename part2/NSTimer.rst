.. _NSTimer:

#######
NSTimer
#######

This is from Hillegass (Objective C).

.. sourcecode:: objective-c

    // clang timer.m -o timer -framework Foundation

    #import <Foundation/Foundation.h>

    @interface TELogger : NSObject
    @property (nonatomic, retain) NSDate *lastTime;
    - (NSString *)lastTimeString;
    - (void)updateLastTime:(NSTimer *)t;
    @end

    @implementation TELogger
    - (NSString *)lastTimeString {
        static NSDateFormatter *df = nil;
        if (!df) {
            df = [[NSDateFormatter alloc] init];
            [df setTimeStyle:NSDateFormatterMediumStyle];
            [df setDateStyle:NSDateFormatterMediumStyle];
            NSLog(@"created date formatter: %@", df);
        }
        return [df stringFromDate:self.lastTime];
    }
    - (void)updateLastTime:(NSTimer *)t {
        NSDate *now = [NSDate date];
        [self setLastTime:now];
        NSLog(@"set time: %@", self.lastTimeString);
    }
    @end

    int main (int argc, const char *argv[]) {
        @autoreleasepool {
            TELogger *logger = [[TELogger alloc] init];
            // _unused NSTimer *timer;
            NSTimer *timer;
            timer = [NSTimer 
            scheduledTimerWithTimeInterval:2.0
                  target:logger
                selector:@selector(updateLastTime:)
                userInfo:nil
                 repeats:YES];
            [[NSRunLoop currentRunLoop] run];
        }
        return 0;
    }

Since the listing is so short, I just put the class interface and implementation together with ``main`` in one file.  The ``TELogger`` class keeps track of the time, and logs it.  We set a timer to fire every 2 seconds and update the time.

.. sourcecode:: bash

    > ./timer
    2014-09-13 05:23:29.632 timer[947:507] created date formatter: <NSDateFormatter: 0x7fb0f8600b20>
    2014-09-13 05:23:29.643 timer[947:507] set time: Sep 13, 2014, 5:23:29 AM
    2014-09-13 05:23:31.631 timer[947:507] set time: Sep 13, 2014, 5:23:31 AM
    2014-09-13 05:23:33.632 timer[947:507] set time: Sep 13, 2014, 5:23:33 AM
    ^Z
    [2]+  Stopped                 ./timer
    >

There are a few more details.  The ``retain`` is not in their code but I got an error without it.

.. sourcecode:: bash

    > clang timer.m -o timer -framework Foundation
    timer.m:4:1: warning: no 'assign', 'retain', or 'copy'
          attribute is specified - 'assign' is assumed
          [-Wobjc-property-no-attribute]
    @property (nonatomic) NSDate *lastTime;
    ^
    timer.m:4:1: warning: default property attribute 'assign'
          not appropriate for non-GC object
          [-Wobjc-property-no-attribute]
    2 warnings generated.
    >

It seems a little weird to get a new ``NSDateFormatter`` every time ``updateLastTime`` is called, but it looks like that's what we do.

In this line

.. sourcecode:: objective-c

    selector:@selector(updateLastTime:)

it's important to include the colon as part of the selector.  The app crashes without it.

And finally, they suggest the use of ``__unused`` to silence a warning about an unused variable

.. sourcecode:: objective-c

    __unused NSTimer *timer;

I don't get the warning. And I had to look twice to realize that there is a double underscore in front of the modifier.
