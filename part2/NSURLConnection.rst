.. _NSURLConnection:

######################
Asynchronous web fetch
######################

.. sourcecode:: objective-c

    // clang main.m -o prog -framework Foundation

    #import <Foundation/Foundation.h>

    @interface TEObject : NSObject <NSURLConnectionDelegate,
        NSURLConnectionDataDelegate>
    { NSMutableData *_incomingData; } 
    - (void) connection:(NSURLConnection *)c didReceiveData:(NSData *)data;
    - (void) connectionDidFinishLoading:(NSURLConnection *)c;
    - (void) connection:(NSURLConnection *)c didFailWithError:(NSError *)e;
    @end

    @implementation TEObject
    - (void) connection:(NSURLConnection *)c didReceiveData:(NSData *)data {
        NSLog(@"received data: %lu bytes", [data length]);
        if (!_incomingData) {
            _incomingData = [[NSMutableData alloc] init];
        }
        [_incomingData appendData:data];
    }
    - (void) connectionDidFinishLoading:(NSURLConnection *)c {
        NSLog(@"finished");
        NSString *s = [[NSString alloc] 
            initWithData:_incomingData
            encoding:NSUTF8StringEncoding ];
            _incomingData = nil;
            NSLog(@"string has %lu characters", [s length]);
            NSLog(@"%@", [s substringWithRange:NSMakeRange(0,100)]);
            exit(0);
    }
    - (void) connection:(NSURLConnection *)c didFailWithError:(NSError *)e {
        NSLog(@"failure: %@", [e localizedDescription]);
        _incomingData = nil;
    }
    @end

    int main (int argc, const char *argv[]) {
        @autoreleasepool {
            TEObject *logger = [[TEObject alloc] init];
            // the whole book
            NSString *s = @"http://www.gutenberg.org/cache/epub/205/pg205.txt";
            NSURL *url = [NSURL URLWithString:s];
            NSURLRequest *uq = [NSURLRequest requestWithURL:url];
            NSURLConnection *c = [[NSURLConnection alloc]
                initWithRequest:uq
                       delegate:logger
               startImmediately:YES ];
           [[NSRunLoop currentRunLoop] run];
         }   
        return 0;
    }

.. sourcecode:: bash

    > clang main.m -o prog -framework Foundation
    > ./prog
    2014-09-13 10:27:48.924 prog[3264:507] received data: 1395 bytes
    2014-09-13 10:27:48.947 prog[3264:507] received data: 30090 bytes
    2014-09-13 10:27:49.058 prog[3264:507] received data: 29560 bytes
    2014-09-13 10:27:49.058 prog[3264:507] received data: 11061 bytes
    2014-09-13 10:27:49.091 prog[3264:507] received data: 3764 bytes
    2014-09-13 10:27:49.092 prog[3264:507] received data: 3596 bytes
    2014-09-13 10:27:49.094 prog[3264:507] received data: 7592 bytes
    2014-09-13 10:27:49.095 prog[3264:507] received data: 3909 bytes
    2014-09-13 10:27:49.096 prog[3264:507] received data: 3527 bytes
    2014-09-13 10:27:49.096 prog[3264:507] received data: 7608 bytes
    2014-09-13 10:27:49.096 prog[3264:507] received data: 3613 bytes
    2014-09-13 10:27:49.096 prog[3264:507] received data: 4210 bytes
    2014-09-13 10:27:49.097 prog[3264:507] received data: 3532 bytes
    2014-09-13 10:27:49.127 prog[3264:507] received data: 7195 bytes
    2014-09-13 10:27:49.129 prog[3264:507] received data: 3686 bytes
    2014-09-13 10:27:49.130 prog[3264:507] received data: 3707 bytes
    2014-09-13 10:27:49.131 prog[3264:507] received data: 3879 bytes
    2014-09-13 10:27:49.132 prog[3264:507] received data: 3667 bytes
    2014-09-13 10:27:49.133 prog[3264:507] received data: 3637 bytes
    2014-09-13 10:27:49.134 prog[3264:507] received data: 3550 bytes
    2014-09-13 10:27:49.134 prog[3264:507] received data: 3638 bytes
    2014-09-13 10:27:49.136 prog[3264:507] received data: 3697 bytes
    2014-09-13 10:27:49.138 prog[3264:507] received data: 3521 bytes
    2014-09-13 10:27:49.139 prog[3264:507] received data: 3715 bytes
    2014-09-13 10:27:49.140 prog[3264:507] received data: 3633 bytes
    2014-09-13 10:27:49.140 prog[3264:507] received data: 3721 bytes
    2014-09-13 10:27:49.166 prog[3264:507] received data: 3637 bytes
    2014-09-13 10:27:49.166 prog[3264:507] received data: 3468 bytes
    2014-09-13 10:27:49.169 prog[3264:507] received data: 3711 bytes
    2014-09-13 10:27:49.170 prog[3264:507] received data: 3635 bytes
    2014-09-13 10:27:49.173 prog[3264:507] received data: 3793 bytes
    2014-09-13 10:27:49.173 prog[3264:507] received data: 3863 bytes
    2014-09-13 10:27:49.174 prog[3264:507] received data: 3667 bytes
    2014-09-13 10:27:49.174 prog[3264:507] received data: 3902 bytes
    2014-09-13 10:27:49.175 prog[3264:507] received data: 3722 bytes
    2014-09-13 10:27:49.175 prog[3264:507] received data: 3642 bytes
    2014-09-13 10:27:49.178 prog[3264:507] received data: 3549 bytes
    2014-09-13 10:27:49.179 prog[3264:507] received data: 3474 bytes
    2014-09-13 10:27:49.179 prog[3264:507] received data: 3652 bytes
    2014-09-13 10:27:49.180 prog[3264:507] received data: 3501 bytes
    2014-09-13 10:27:49.181 prog[3264:507] received data: 3520 bytes
    2014-09-13 10:27:49.181 prog[3264:507] received data: 3838 bytes
    2014-09-13 10:27:49.199 prog[3264:507] received data: 3403 bytes
    2014-09-13 10:27:49.200 prog[3264:507] received data: 3584 bytes
    2014-09-13 10:27:49.202 prog[3264:507] received data: 3641 bytes
    2014-09-13 10:27:49.203 prog[3264:507] received data: 3810 bytes
    2014-09-13 10:27:49.214 prog[3264:507] received data: 3582 bytes
    2014-09-13 10:27:49.215 prog[3264:507] received data: 3762 bytes
    2014-09-13 10:27:49.219 prog[3264:507] received data: 3731 bytes
    2014-09-13 10:27:49.219 prog[3264:507] received data: 3549 bytes
    2014-09-13 10:27:49.220 prog[3264:507] received data: 3670 bytes
    2014-09-13 10:27:49.220 prog[3264:507] received data: 3555 bytes
    2014-09-13 10:27:49.222 prog[3264:507] received data: 7379 bytes
    2014-09-13 10:27:49.222 prog[3264:507] received data: 3851 bytes
    2014-09-13 10:27:49.223 prog[3264:507] received data: 3784 bytes
    2014-09-13 10:27:49.227 prog[3264:507] received data: 3665 bytes
    2014-09-13 10:27:49.227 prog[3264:507] received data: 3530 bytes
    2014-09-13 10:27:49.236 prog[3264:507] received data: 3609 bytes
    2014-09-13 10:27:49.236 prog[3264:507] received data: 3732 bytes
    2014-09-13 10:27:49.237 prog[3264:507] received data: 3807 bytes
    2014-09-13 10:27:49.238 prog[3264:507] received data: 3644 bytes
    2014-09-13 10:27:49.238 prog[3264:507] received data: 3418 bytes
    2014-09-13 10:27:49.239 prog[3264:507] received data: 3600 bytes
    2014-09-13 10:27:49.239 prog[3264:507] received data: 3742 bytes
    2014-09-13 10:27:49.252 prog[3264:507] received data: 3658 bytes
    2014-09-13 10:27:49.256 prog[3264:507] received data: 3741 bytes
    2014-09-13 10:27:49.256 prog[3264:507] received data: 3762 bytes
    2014-09-13 10:27:49.258 prog[3264:507] received data: 3691 bytes
    2014-09-13 10:27:49.258 prog[3264:507] received data: 3827 bytes
    2014-09-13 10:27:49.273 prog[3264:507] received data: 19153 bytes
    2014-09-13 10:27:49.273 prog[3264:507] received data: 11295 bytes
    2014-09-13 10:27:49.284 prog[3264:507] received data: 3874 bytes
    2014-09-13 10:27:49.285 prog[3264:507] received data: 29120 bytes
    2014-09-13 10:27:49.286 prog[3264:507] received data: 3602 bytes
    2014-09-13 10:27:49.286 prog[3264:507] received data: 3682 bytes
    2014-09-13 10:27:49.287 prog[3264:507] received data: 3507 bytes
    2014-09-13 10:27:49.295 prog[3264:507] received data: 10852 bytes
    2014-09-13 10:27:49.295 prog[3264:507] received data: 3612 bytes
    2014-09-13 10:27:49.302 prog[3264:507] received data: 3516 bytes
    2014-09-13 10:27:49.309 prog[3264:507] received data: 3899 bytes
    2014-09-13 10:27:49.310 prog[3264:507] received data: 3765 bytes
    2014-09-13 10:27:49.310 prog[3264:507] received data: 3625 bytes
    2014-09-13 10:27:49.311 prog[3264:507] received data: 3778 bytes
    2014-09-13 10:27:49.311 prog[3264:507] received data: 7352 bytes
    2014-09-13 10:27:49.312 prog[3264:507] received data: 3915 bytes
    2014-09-13 10:27:49.319 prog[3264:507] received data: 3627 bytes
    2014-09-13 10:27:49.320 prog[3264:507] received data: 3618 bytes
    2014-09-13 10:27:49.321 prog[3264:507] received data: 3616 bytes
    2014-09-13 10:27:49.321 prog[3264:507] received data: 3551 bytes
    2014-09-13 10:27:49.322 prog[3264:507] received data: 3452 bytes
    2014-09-13 10:27:49.323 prog[3264:507] received data: 3609 bytes
    2014-09-13 10:27:49.323 prog[3264:507] received data: 3559 bytes
    2014-09-13 10:27:49.324 prog[3264:507] received data: 3623 bytes
    2014-09-13 10:27:49.325 prog[3264:507] received data: 3698 bytes
    2014-09-13 10:27:49.326 prog[3264:507] received data: 3509 bytes
    2014-09-13 10:27:49.328 prog[3264:507] received data: 10930 bytes
    2014-09-13 10:27:49.328 prog[3264:507] received data: 3815 bytes
    2014-09-13 10:27:49.328 prog[3264:507] received data: 3777 bytes
    2014-09-13 10:27:49.329 prog[3264:507] received data: 3667 bytes
    2014-09-13 10:27:49.331 prog[3264:507] received data: 3596 bytes
    2014-09-13 10:27:49.332 prog[3264:507] received data: 3742 bytes
    2014-09-13 10:27:49.334 prog[3264:507] received data: 3683 bytes
    2014-09-13 10:27:49.334 prog[3264:507] received data: 3974 bytes
    2014-09-13 10:27:49.335 prog[3264:507] received data: 3819 bytes
    2014-09-13 10:27:49.336 prog[3264:507] received data: 3647 bytes
    2014-09-13 10:27:49.339 prog[3264:507] received data: 3754 bytes
    2014-09-13 10:27:49.340 prog[3264:507] received data: 3456 bytes
    2014-09-13 10:27:49.344 prog[3264:507] received data: 4057 bytes
    2014-09-13 10:27:49.345 prog[3264:507] received data: 3812 bytes
    2014-09-13 10:27:49.346 prog[3264:507] received data: 3683 bytes
    2014-09-13 10:27:49.346 prog[3264:507] received data: 3741 bytes
    2014-09-13 10:27:49.352 prog[3264:507] received data: 3603 bytes
    2014-09-13 10:27:49.352 prog[3264:507] received data: 3496 bytes
    2014-09-13 10:27:49.355 prog[3264:507] received data: 3701 bytes
    2014-09-13 10:27:49.355 prog[3264:507] received data: 7251 bytes
    2014-09-13 10:27:49.356 prog[3264:507] received data: 3552 bytes
    2014-09-13 10:27:49.357 prog[3264:507] received data: 3349 bytes
    2014-09-13 10:27:49.358 prog[3264:507] received data: 3628 bytes
    2014-09-13 10:27:49.358 prog[3264:507] received data: 3620 bytes
    2014-09-13 10:27:49.359 prog[3264:507] received data: 3674 bytes
    2014-09-13 10:27:49.359 prog[3264:507] received data: 3505 bytes
    2014-09-13 10:27:49.361 prog[3264:507] received data: 7184 bytes
    2014-09-13 10:27:49.361 prog[3264:507] received data: 3878 bytes
    2014-09-13 10:27:49.362 prog[3264:507] received data: 3680 bytes
    2014-09-13 10:27:49.362 prog[3264:507] received data: 3842 bytes
    2014-09-13 10:27:49.363 prog[3264:507] received data: 3565 bytes
    2014-09-13 10:27:49.363 prog[3264:507] received data: 3912 bytes
    2014-09-13 10:27:49.364 prog[3264:507] received data: 4041 bytes
    2014-09-13 10:27:49.365 prog[3264:507] received data: 7763 bytes
    2014-09-13 10:27:49.366 prog[3264:507] received data: 3796 bytes
    2014-09-13 10:27:49.370 prog[3264:507] received data: 3618 bytes
    2014-09-13 10:27:49.370 prog[3264:507] received data: 3827 bytes
    2014-09-13 10:27:49.371 prog[3264:507] received data: 3984 bytes
    2014-09-13 10:27:49.371 prog[3264:507] received data: 3841 bytes
    2014-09-13 10:27:49.373 prog[3264:507] received data: 13316 bytes
    2014-09-13 10:27:49.374 prog[3264:507] received data: 3847 bytes
    2014-09-13 10:27:49.374 prog[3264:507] received data: 4119 bytes
    2014-09-13 10:27:49.379 prog[3264:507] received data: 173 bytes
    2014-09-13 10:27:49.379 prog[3264:507] finished
    2014-09-13 10:27:49.380 prog[3264:507] string has 665331 characters
    2014-09-13 10:27:49.380 prog[3264:507] The Project Gutenberg EBook of Walden, and On The Duty Of Civil
    Disobedience, by Henry David Thorea
    > 



