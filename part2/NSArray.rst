.. _NSArray:

#######
NSArray
#######

Fast Enumeration
----------------

I wrote a category on NSArray to make it interesting.  The new method tests two arrays for equality.  Also interesting is the way we report the value of a boolean at the end.

The code illustrates various (old and new) ways to make arrays, including the array "literal" construct with ``@[]`` and the old-fashioned ``arrayWithObjects``, which requires a ``nil`` terminator.

.. sourcecode:: objective-c

    // clang main.m -o prog -framework Foundation

    #import <Foundation/Foundation.h>

    @interface NSArray (Equality)
    - (BOOL)isEqualToArray:(NSArray *)other;
    @end
    @implementation NSArray (Equality)
    - (BOOL)isEqualToArray:(NSArray *)other {
        for (int i = 0; i < self.count; i++) {
            id o1 = [self objectAtIndex:i];
            id o2 = [other objectAtIndex:i];
            if (!([o1 isEqualTo:o2])) {
                return NO;
            }
        }
        return YES;
    }
    @end

    int main (int argc, const char *argv[]) {
        @autoreleasepool {
            NSString *s1 = @"one \u2661";
            NSString *s2 = @"two \u2661s";
            NSArray *a = @[s1,s2];
            for (NSString *s in a) {
                NSLog(@"%@",s);
            }
            NSMutableArray *ma = [NSMutableArray array];
            [ma addObject:s1];
            [ma addObject:s2];

            NSArray *a2 = [NSArray arrayWithArray:ma];
            NSArray *a3 = [NSArray arrayWithObjects:s1,s2,nil];
            NSLog([a isEqualTo:a2] ? @"Yes" : @"No");
            NSLog([a isEqualTo:a3] ? @"Yes" : @"No");
        }   
        return 0;
    }
    

.. sourcecode:: bash

    > clang main.m -o prog -framework Foundation
    > ./prog
    2014-09-13 08:15:01.334 prog[2363:507] one ♡
    2014-09-13 08:15:01.336 prog[2363:507] two ♡s
    2014-09-13 08:15:01.337 prog[2363:507] Yes
    2014-09-13 08:15:01.337 prog[2363:507] Yes
    >

Sorting
-------

Arrays, objects, and sorting ..

.. sourcecode:: objective-c

    // clang main.m -o prog -framework Foundation

    #import <Foundation/Foundation.h>

    @interface TEObject : NSObject
    @property (nonatomic, retain) NSString *s;
    @property (nonatomic) unsigned int i;
    @property (nonatomic) unsigned int j;
    @end

    @implementation TEObject
    - init {
        if (self = [super init]) {
            return self;
        }
        return nil;
    }
    - initWithName:(NSString *)str v1:(int)m v2:(int)n {
        [self init];
        self.s = str;
        self.i = m;
        self.j = n;
        return self;
    }
    - (NSString *)description {
        return [NSString stringWithFormat:@"TEObject: %d %d", self.i, self.j ];
    }
    @end

    int main (int argc, const char *argv[]) {
        @autoreleasepool {
            TEObject *o = [[TEObject alloc] initWithName:@"o" v1:1 v2:0];
            TEObject *p = [[TEObject alloc] initWithName:@"p" v1:3 v2:2];
            TEObject *q = [[TEObject alloc] initWithName:@"q" v1:2 v2:3];
            TEObject *r = [[TEObject alloc] initWithName:@"q" v1:2 v2:2];
            NSArray *a = @[p,q,r,o];
            for (TEObject *obj in a) { NSLog(@"%@", obj); }
            printf("\n");

            NSMutableArray *ma = [NSMutableArray arrayWithArray:a];
            NSSortDescriptor *sd1, *sd2;
            sd1 = [NSSortDescriptor sortDescriptorWithKey:@"i"
                                                ascending:YES];
            [ma sortUsingDescriptors:@[sd1]];
            for (TEObject *obj in ma) { NSLog(@"%@", obj); }
            sd2 = [NSSortDescriptor sortDescriptorWithKey:@"j"
                                                ascending:YES];
            printf("\n");
            ma = [NSMutableArray arrayWithArray:a];
            [ma sortUsingDescriptors:@[sd2,sd1]];
            for (TEObject *obj in ma) { NSLog(@"%@", obj); }
        }   
        return 0;
    }

.. sourcecode:: objective-c

    > clang main.m -o prog -framework Foundation
    > ./prog
    2014-09-13 09:16:28.039 prog[2661:507] TEObject: 3 2
    2014-09-13 09:16:28.042 prog[2661:507] TEObject: 2 3
    2014-09-13 09:16:28.043 prog[2661:507] TEObject: 2 2
    2014-09-13 09:16:28.043 prog[2661:507] TEObject: 1 0

    2014-09-13 09:16:28.044 prog[2661:507] TEObject: 1 0
    2014-09-13 09:16:28.044 prog[2661:507] TEObject: 2 3
    2014-09-13 09:16:28.045 prog[2661:507] TEObject: 2 2
    2014-09-13 09:16:28.045 prog[2661:507] TEObject: 3 2

    2014-09-13 09:16:28.046 prog[2661:507] TEObject: 1 0
    2014-09-13 09:16:28.046 prog[2661:507] TEObject: 2 2
    2014-09-13 09:16:28.046 prog[2661:507] TEObject: 3 2
    2014-09-13 09:16:28.047 prog[2661:507] TEObject: 2 3
    >
