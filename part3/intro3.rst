.. _intro3:

################
Xcode and AppKit
################

I have written a number of Cocoa Applications over the years and worked my way through Hillegass's book *Cocoa Programming* at least twice.  Few of these efforts survive on my current laptop, probably few anywhere, and none on the web.  

But I downloaded the newest version of Xcode (Xcode6-Beta5) this morning in order to work on the Swift programming language.  So I thought I might explore again a little bit with Xcode too, and try to document my path through the maze.

--------
IBAction
--------

Let's make a project in Xcode, call it Button.  It should be a Cocoa Application, accepting the Xcode defaults.  We're going to put a button in the window, and hook it up to our application.

One thing it took me a while to appreciate:  when you open ``MainMenu.xib`` and see this:

.. image:: /figures/button1.png
    :scale: 75 %

you are not looking at the window, but just at the menu bar.  Click the window-like widget (7th icon down on the left side)

.. image:: /figures/button2.png
    :scale: 75 %

and see this:

.. image:: /figures/button3.png
    :scale: 75 %

Now we're ready to go.  Drag out a button from the lower right and put it in the window.  Re-label it.

.. image:: /figures/button4.png
    :scale: 75 %

Set up the code in ``AppDelegate.m`` by adding:

.. sourcecode:: objective-c

    -(IBAction) push: (id)sender {
        NSLog(@"push");
    }

and put the appropriate declaration in ``AppDelegate.h``:

.. sourcecode:: objective-c

    -(IBAction) push: (id)sender;

Finally, go back to ``MainMenu.xib`` and control-drag *from the button to the AppDelegate icon* (it's the fourth icon down, an Object cube with the tool tip:  AppDelegate).  The button needs to send messages *to* the AppDelegate.

You should see

.. image:: /figures/button5.png
    :scale: 75 %

Click on push.

Now run the application, push the button and watch the log:

.. sourcecode:: objective-c

    2014-08-18 08:08:13.445 Button[3552:303] push

It works!

--------
IBOutlet
--------

Continuing in Xcode, we're going to put a text field in the window, and hook it up to our application.

The code for an IBOutlet is hard for me to remember, even though it's simple.  In the header file, put this

.. sourcecode:: objective-c

    @property (weak) IBOutlet NSTextField *textField;
    
Make sure that if there are instance variables declared (between brackets ``{ }``), that the ``@property`` declaration is after the closing bracket.

Control-drag *from the AppDelegate icon to the text field* (the fourth icon down, an Object cube with the tool tip:  AppDelegate).  The AppDelegate needs to send messages *to* the textfield.

.. image:: /figures/outlet.png
    :scale: 100 %

To make things a little more interesting, we will add two variables, so the header looks like this:

.. sourcecode:: objective-c

    #import <Cocoa/Cocoa.h>
    @interface AppDelegate : NSObject <NSApplicationDelegate>
    {
        NSString *s;
        int n;
    }
    @property (weak) IBOutlet NSTextField *textField;
    -(IBAction) push: (id)sender;
    @end

(scrunched up a bit).  With these declarations, we can modify the ``(IBAction) push`` as follows:

.. sourcecode:: objective-c

    -(IBAction) push: (id)sender {
        NSLog(@"push");
        s = [self.textField stringValue];
        NSLog(@"%@", s);
        if ([s isEqualToString:@"Label"]) { n = 0; }
        else { n = [s intValue]; }
        n += 1;
        s = [NSString stringWithFormat:@"%d", n];
        NSLog(@"%@", s);
        [self.textField setStringValue:s];
        NSColor *black = [NSColor blackColor];
        NSColor *red = [NSColor redColor];
        if (n % 2) { [self.textField setTextColor:black]; }
        else { [self.textField setTextColor:red]; }
    }

Each time the button is pushed, we query the text field for its value.  The first time it will be "Label".  In that case we set our ``int`` variable to 0.  Otherwise, it should be convertible to an integer, which we get with ``intValue``.  Then, in all cases, we increment it.  So ``n`` will take on the values ``1, 2, 3..``, sequentially.

We convert back to a string (using ``stringWithFormat``), and set the text field's value to that.  For an extra flourish, we set the text color to black for even numbers and red for odd ones.

-----------
Custom View
-----------

Open Xcode and make a new project, a Cocoa Application named Hello.  Accept all the defaults.

Now, in ``MainMenu.xib`` there is a column (a "dock") of icons to the left of the editor window;  click on the bottom one that looks like it is a miniature window.  If you then click on the interior of the window that comes up, the widget at the upper right should say:  "Class:  NSView".  Change that to a custom class:  MyView.

Select the view again and go to ``File > New > File``, then select Cocoa Class.  It should say "Subclass of NSView";  call the new class MyView.

After adding them, you will have two new files in the project.  

In ``MyView.m`` add this to the ``drawRect`` method:

.. sourcecode:: objective-c

    NSLog(@"drawRect");
    [[NSColor greenColor] set];
    NSRectFill ( [self bounds] );

That should be it.  Build and Run the project with ``CMD-R``.

.. image:: /figures/hello.png
    :scale: 75 %
   
