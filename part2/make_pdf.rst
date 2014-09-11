.. _make_pdf:

############
Print to PDF
############

.. sourcecode:: objective-c

    // demo of printing to a pdf without a standard app
    // clang view.m -o view -framework Cocoa

    #import <Cocoa/Cocoa.h>
    
    @interface
    MyView: NSView { }
    @property (retain) NSImage *img;
    @end

    @implementation
    MyView: NSView 

    @synthesize img;

    - initWithFrame:(NSRect) frameRect {
        if (self = [super initWithFrame:frameRect])
            img = [NSImage imageNamed:@"moon"];
            return self;
        return nil;
    }

    -(void) drawRect:(NSRect) dirtyRect {
        [[NSColor cyanColor] set];
        NSRectFill([self bounds]);
        NSRect r = NSMakeRect(50,50,img.size.width,img.size.height);
        [img drawInRect:r];
    }
    
    @end

    int main (int argc, const char* argv[]) {
        @autoreleasepool {
            NSRect f = NSMakeRect(0,0,500,300);
            MyView *mv = [[MyView alloc] initWithFrame:f];
            //[mv setNeedsDisplay:YES];
            NSData *data = [mv dataWithPDFInsideRect:[mv bounds]];
            NSString *fn = @"x.pdf";
            [data writeToFile:fn atomically:YES];
        }
        return 0;
    }

.. sourcecode:: bash

    > clang view.m -o view -framework Cocoa
    > ./view
    >

.. image:: /figures/moon_pdf.png
    :scale: 50 %

