.. _NSColor:

#######
NSColor
#######

.. sourcecode:: objective-c

    #import <Foundation/Foundation.h>
    #import <AppKit/AppKit.h>

    int main(int argc, char * argv[]) {
        @autoreleasepool {
            NSArray *a = [NSColorList availableColorLists];
            for (NSColorList *cl in a) {
                NSLog(@"%@", [cl description]);
            }
        }
        return 0;
    }

.. sourcecode:: bash

    > clang main.m -o prog -framework Foundation -framework AppKit
    > ./prog
    2014-09-06 07:38:40.316 prog[9944:507] NSColorList 0x7fa9a8c10750 \
    name:Apple device:(null) file:/System/Library/Colors/Apple.clr loaded:0
    2014-09-06 07:38:40.318 prog[9944:507] NSColorList 0x7fa9a8c0ed60 \
    name:System device:(null) file:/System/Library/Colors/System.clr loaded:0
    2014-09-06 07:38:40.318 prog[9944:507] NSColorList 0x7fa9a8c11200 \
    name:Crayons device:(null) file:/System/Library/Colors/Crayons.clr loaded:0
    2014-09-06 07:38:40.319 prog[9944:507] NSColorList 0x7fa9a8d03aa0 \
    name:Web Safe Colors device:(null) file:(null) loaded:1
    >

We see the standard color lists, and their files, and ``loaded:0`` or ``loaded:1``.
which means ____

For more about loading color lists, see:

https://discussions.apple.com/thread/2397302?start=0&tstart=0

Change the code to:

.. sourcecode:: objective-c

    NSColorList *cl = [NSColorList colorListNamed:@"Crayons"];
    NSLog(@"%@", [[cl allKeys] componentsJoinedByString:@","]);
    NSColor *c = [cl colorWithKey:@"Teal"];
    NSLog(@"%@", c);

.. sourcecode:: bash

    > clang main.m -o prog -framework Foundation -framework AppKit
    > ./prog
    2014-09-06 07:50:02.040 prog[10051:507] \ 
    Cantaloupe,Honeydew,Spindrift,Sky,Lavender,Carnation,Licorice,\
    Snow,Salmon,Banana,Flora,Ice,Orchid,Bubblegum,Lead,Mercury,\
    Tangerine,Lime,SeaFoam,Aqua,Grape,Strawberry,Tungsten,Silver\
    ,Maraschino,Lemon,Spring,Turquoise,Blueberry,Magenta,Iron,\
    Magnesium,Mocha,Fern,Moss,Ocean,Eggplant,Maroon,Steel,Aluminum\
    ,Cayenne,Asparagus,Clover,Teal,Midnight,Plum,Tin,Nickel
    2014-09-06 07:50:02.042 prog[10051:507] \
    NSCalibratedRGBColorSpace 0 0.501961 0.501961 1
    >
    
Exploring colors:

.. sourcecode:: objective-c

    #import <Foundation/Foundation.h>
    #import <AppKit/AppKit.h>

    int main(int argc, char * argv[]) {
        @autoreleasepool {
            NSColorList *cl = [NSColorList colorListNamed:@"Crayons"];
            NSColor *c = [cl colorWithKey:@"Lavender"];
            NSLog(@"%@", c);
            CGFloat r,g,b,a;
            [c getRed:&r green:&g blue:&b alpha:&a];
            NSLog(@"%5.3f %5.3f %5.3f %5.3f", r, g, b, a);
            NSLog(@"%5.3f", [c redComponent]);
            c = [NSColor colorWithCalibratedRed:0 green:1.0 blue:1.0 alpha:0.5];
            NSLog(@"%@", c);
        }
        return 0;
    }

.. sourcecode:: bash

    > clang main.m -o prog -framework Foundation -framework AppKit
    > ./prog
    2014-09-06 08:00:04.121 prog[10173:507] NSCalibratedRGBColorSpace 0.8 0.4 1 1
    2014-09-06 08:00:04.122 prog[10173:507] 0.800 0.400 1.000 1.000
    2014-09-06 08:00:04.123 prog[10173:507] 0.800
    2014-09-06 08:00:04.123 prog[10173:507] NSCalibratedRGBColorSpace 0 1 1 0.5
    >

