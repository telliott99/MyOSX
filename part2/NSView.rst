.. _NSView:

###############
NSView subclass
###############

Make a new Xcode project, a Cocoa App named Switch.

Add some image files to the project:  bush.png, chimp.png

http://telliott99.blogspot.com/2009/10/binding-image-to-checkbox.html

Drag a custom view to window in the nib and change its class to MyView.  Under File > New: add files for a Cocoa class MyView, resulting in ``MyView.m`` and ``MyView.h``.

In ``MyView.m``:

.. sourcecode:: objective-c

    #import "MyView.h"

    @implementation MyView

    -(void)awakeFromNib{
        NSString *fn = [[NSBundle mainBundle]
              pathForResource:@"bush" ofType:@"png"];
        bush = [[NSImage alloc] initWithContentsOfFile:fn];
        chimp = [NSImage imageNamed:@"chimp"];
        iRect = NSMakeRect(0,0,bush.size.width,bush.size.height);
        flag = YES;
        NSLog(@"%@", bush);
    }

    - (void)drawRect:(NSRect)dirtyRect {
        [super drawRect:dirtyRect];
        NSImage *img;
        NSRect vRect = [self bounds];
        if (flag) { img = bush; }
        else { img = chimp; }

        float f = 1.0;
        float fw, fh;
        if (! (NSContainsRect(vRect,iRect))) {
            fw = img.size.width/vRect.size.width;
            fh = img.size.height/vRect.size.height;
            if (fw >= fh) { f = fw; }
            else { f = fh; }
            NSLog(@"%4.2f", f);
        }
        NSRect sRect;
        sRect.size.width = vRect.size.width * f;
        sRect.size.height = vRect.size.height * f;
        sRect.origin = NSZeroPoint;
        [img drawInRect:vRect
               fromRect:sRect
              operation:NSCompositeCopy
               fraction:1.0];
    }

    - (IBAction)mouseDown:(NSEvent *)e {
        NSLog(@"mouseDown");
        if (flag == YES) { flag = NO; }
        else { flag = YES; }
        [self setNeedsDisplay:YES];
    }

    @end

In ``awakeFromNib`` we load the images from the bundle.  Both methods shown give the same result.  In drawing the images, we re-scale the image to fit the size of the custom view.  And finally, clicking on the custom view (not just the window, but inside the view), switches the image.

.. image:: /figures/Switch.png
    :scale: 100 %

