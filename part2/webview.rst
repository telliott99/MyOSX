.. _webview:

#######
WebView
#######

This used to work.  But now it doesn't.  I'll get back to it.

Here is the code for a class that is a controller for a WebView.  All you need is a WebView in the nib and set up this class as its delegate (DownloadDelegate and FrameLoadDelegate) and also set the WebView as an outlet.

Don't forget to link to the WebKit framework in Xcode.

``MyWebViewController.h``

.. sourcecode:: objective-c

    #import <Cocoa/Cocoa.h>
    #import <WebKit/WebKit.h>

    @interface MyWebViewController : NSObject {
    
    IBOutlet WebView *webView;
    
    }

    @end

``MyWebViewController.m``

.. sourcecode:: objective-c

    import "MyWebViewController.h"

    @implementation MyWebViewController

    - (void)applicationDidFinishLaunching:(NSNotification *)aNotification {
        NSLog(@"WVC Application did finish launching");
    }

    - (void)awakeFromNib {
        [webView setFrameLoadDelegate:self];
        [webView setResourceLoadDelegate:self];
        [webView setUIDelegate:self];
        [self loadWebForm];
    }

    - (void)loadWebForm {
        NSString *path;
        NSString *fn = @"/Desktop/form.html";
        path = [NSHomeDirectory() stringByAppendingString:fn];
        NSURL* url = [NSURL URLWithString:path];
        NSURLRequest *req = [NSURLRequest requestWithURL:url];
        WebFrame *wf = [webView mainFrame];
        [wf loadRequest:req];
        [self plotData];
    }

    - (void) webView:(WebView*)wv didStartProvisionalLoadForFrame:(WebFrame*)frame {
        NSLog(@"provisionalLoad..");
        NSLog(@"URL: %@", [webView mainFrameURL]);
        NSLog(@"Data: %@", [[[webView mainFrame] provisionalDataSource] data]);
    }

    - (id) webView:(WebView*)wv identifierForInitialRequest:(NSURLRequest*)req fromDataSource:(id)listener {
        NSLog(@"identifier..");
        NSLog(@"Request: %@%@", req, listener);
        return self;
    }

    /*
     This doesn't work, even though WebView inherits from NSView
     I get data, and it makes a pdf, but the pdf has no content
     */

    - (void) plotData {
        NSData *data = [webView dataWithPDFInsideRect:[webView bounds]];
        NSLog(@"%@", data);
        NSString *fn = [NSHomeDirectory() stringByAppendingString:@"/Desktop/x.pdf"];
        [data writeToFile:fn atomically:YES];

    }

    @end
    
The code for ``form.html`` is

.. sourcecode:: html

    <Content-type: text/html>

    <form name="input" action="cgi-bin/script.py" method="get">
    
    First name: <input type="text" name="firstname" /><br />
    Last name: <input type="text" name="lastname" /><br /><br />
    <input type="radio" name="sex" value="male" /> Male<br />
    <input type="radio" name="sex" value="female" /> Female<br /><br />
    <input type="checkbox" name="vehicle" value="Bike" /> I have a bike<br />
    <input type="checkbox" name="vehicle" value="Car" /> I have a car<br /><br />

    <textarea rows="5" cols="40">
    The cat was playing in the garden.
    </textarea>
    <input type="submit" value="Submit" />
    </form>
    </html>
