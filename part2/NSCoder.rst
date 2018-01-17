.. _NSCoder:

#######
NSCoder
#######

Here is a working example showing basic implementation of a class that follows the NSCoder protocol.  It also implements a convenience initializer ``initWithString``.

The NSCoder protocol requires

    - ``- (id)initWithCoder:(NSCoder *)coder``
    - ``- (void)encodeWithCoder:(NSCoder *)coder``
    
Neither of these gets declared in the header.  Since the superclass ``NSObject`` doesn't have ``initWithCoder`` we call the basic ``init`` for it.

Here is the code (in a single file):

``NSCoder.m``

.. sourcecode:: objective-c

    #import <Foundation/Foundation.h>

    @interface SimpleClass: NSObject
    { NSString* name; }

    @property (retain) NSString *name;
    - (void)report;
    @end

    @implementation SimpleClass
    @synthesize name;

    - (id)init {
        if (self = [super init]) {
            [self setName:@"instance of SimpleClass"];
            return self;
        }
        return nil;
    }

    - (id)initWithString:(NSString *)n {
        if (self = [super init]) {
            [self setName:n];
            return self;
        }
        return nil;
    }

    - (id)initWithCoder:(NSCoder *)coder {
        if(self = [super init]) {
            NSString *s = [coder decodeObjectForKey:@"name"];
            NSLog(@"here, %@", s);
            [self setName:s];
            return self;
           }
        return nil;
    }

    - (void)encodeWithCoder:(NSCoder *)coder {
        [coder encodeObject:name forKey:@"name"];
    }

    - (NSString*) description {
        return [self name];
    }

    - (void)report { NSLog(@"%@", self.name); }
    @end

    int main(int argc, char * argv[]) {
        @autoreleasepool {
            SimpleClass *sc = [[SimpleClass alloc] \
                initWithString:@"Tom"];
            NSLog(@"%@", sc);
            NSString *s = @"/Desktop/demo";
            NSString *fn = [NSHomeDirectory() \
                stringByAppendingString:s];
            if ([NSKeyedArchiver archiveRootObject:sc toFile:fn]) {
                NSLog(@"yes");
            }
            SimpleClass *sc2;
            sc2 = [NSKeyedUnarchiver unarchiveObjectWithFile:fn];
            NSLog(@"%@", sc2);
        }
        return 0;
    }
    
And here is how it works:

.. sourcecode:: bash

    > ./prog
    2014-09-05 18:42:15.688 prog[7951:507] Tom
    2014-09-05 18:42:15.691 prog[7951:507] yes
    2014-09-05 18:42:15.692 prog[7951:507] here, Tom
    2014-09-05 18:42:15.693 prog[7951:507] Tom
    >
    
For what it's worth the file where everything is encoded looks like this:

.. sourcecode:: bash

    > hexdump -C demo
    00000000  62 70 6c 69 73 74 30 30  d4 01 02 03 04 05 06 15  |bplist00........|
    00000010  16 58 24 76 65 72 73 69  6f 6e 58 24 6f 62 6a 65  |.X$versionX$obje|
    00000020  63 74 73 59 24 61 72 63  68 69 76 65 72 54 24 74  |ctsY$archiverT$t|
    00000030  6f 70 12 00 01 86 a0 a4  07 08 0d 0e 55 24 6e 75  |op..........U$nu|
    00000040  6c 6c d2 09 0a 0b 0c 54  6e 61 6d 65 56 24 63 6c  |ll.....TnameV$cl|
    00000050  61 73 73 80 02 80 03 53  54 6f 6d d2 0f 10 11 12  |ass....STom.....|
    00000060  5a 24 63 6c 61 73 73 6e  61 6d 65 58 24 63 6c 61  |Z$classnameX$cla|
    00000070  73 73 65 73 5b 53 69 6d  70 6c 65 43 6c 61 73 73  |sses[SimpleClass|
    00000080  a2 13 14 5b 53 69 6d 70  6c 65 43 6c 61 73 73 58  |...[SimpleClassX|
    00000090  4e 53 4f 62 6a 65 63 74  5f 10 0f 4e 53 4b 65 79  |NSObject_..NSKey|
    000000a0  65 64 41 72 63 68 69 76  65 72 d1 17 18 54 72 6f  |edArchiver...Tro|
    000000b0  6f 74 80 01 08 11 1a 23  2d 32 37 3c 42 47 4c 53  |ot.....#-27<BGLS|
    000000c0  55 57 5b 60 6b 74 80 83  8f 98 aa ad b2 00 00 00  |UW[`kt..........|
    000000d0  00 00 00 01 01 00 00 00  00 00 00 00 19 00 00 00  |................|
    000000e0  00 00 00 00 00 00 00 00  00 00 00 00 b4           |.............|
    000000ed
    >
