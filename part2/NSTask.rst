.. _NSTask:

######
NSTask
######

http://telliott99.blogspot.com/2008/08/nstask.html

http://telliott99.blogspot.com/2010/12/nstask.html

http://telliott99.blogspot.com/2011/01/nstask-again.html

``talk.m``:

.. sourcecode:: objective-c

    #import <Foundation/Foundation.h>

    @interface
    Tasker : NSObject { }
    
    // this property is just for use in checkTaskStatus:
    @property (retain) NSTask* task;
    
    -(void) doSetup;
    -(void) checkTaskStatus:(NSNotification *)notification;
    @end

    @implementation
    Tasker : NSObject

    @synthesize task;

    -(void) doSetup{
        NSNotificationCenter *nc;
        nc = [NSNotificationCenter defaultCenter];
        [nc addObserver:self
               selector:@selector(checkTaskStatus:)
                   name:NSTaskDidTerminateNotification
                 object:nil];
    }

    -(void) checkTaskStatus:(NSNotification *)notification {
        NSLog(@"checkTaskStatus");
        if ([[notification object] terminationStatus]) {
            NSLog(@"success");
        }
        // defined by each task
        int ATASK_SUCCESS_VALUE = 0;
        if (![task isRunning]) {
            int status = [task terminationStatus];
            if (status == ATASK_SUCCESS_VALUE)
                NSLog(@"Task succeeded.");
            else
                NSLog(@"Task failed.");
        }
    }

    @end


    int main (int argc, const char* argv[]) {
        NSString *path = @"/usr/bin/python";
        NSString *home = NSHomeDirectory();
        NSString *script = @"/Desktop/script.py";
        NSString *arg1 = [NSString stringWithFormat:@"%@%@", home, script];
        NSArray *args = [NSArray arrayWithObjects:arg1,@"to me",nil];

        // one way:
        /*
        NSTask *t = [NSTask launchedTaskWithLaunchPath:path arguments:args];
        NSLog(@"%@",[t description]);
        [t waitUntilExit];
        */

        // another, using a class:
        NSTask *t = [[NSTask alloc] init];
        [t setLaunchPath:path];
        [t setArguments:args];
        // this way, we can set environmental variables
        NSMutableDictionary *md = [[NSMutableDictionary alloc] init];
        [md setObject:@"MY_VALUE" forKey:@"MYKEY"];
        [t setEnvironment:md];
        NSLog(@"%@", [[t environment] objectForKey:@"MYKEY"]);

        NSLog(@"%@",[t description]);
        Tasker *tr = [[Tasker alloc] init];
        [tr doSetup];
        [tr setTask:t];

        [t launch];
        //[t waitUntilExit];

        [[NSRunLoop currentRunLoop] 
            runUntilDate:[NSDate dateWithTimeIntervalSinceNow:2.0]]; 
        return 0;
    }
    
And then

``script.py``:

.. sourcecode:: objective-c
    
    import os, sys

    try:
        print 'Hello world ' + sys.argv[1] + '!'
    except IndexError:
        print 'Hello world, everybody!'

    D = os.environ
    k = 'MYKEY'
    if k in D:  print k, D[k]
    else:  
        print k, 'not present'
        print D

Test the script:

.. sourcecode:: bash

    > export MYKEY="my_value"
    > env
    ..
    MYKEY=my_value
    > python script.py 
    Hello world, everybody!
    MYKEY my_value
    >

And run it:

.. sourcecode:: bash

    > clang task.m -o task -framework Foundation
    > ./task
    2014-09-10 19:21:36.518 task[520:507] MY_VALUE
    2014-09-10 19:21:36.520 task[520:507] <NSConcreteTask: 0x7feaa8e01e40>
    Hello world to me!
    MYKEY MY_VALUE
    2014-09-10 19:21:36.550 task[520:507] checkTaskStatus
    2014-09-10 19:21:36.550 task[520:507] Task succeeded.
    >

We see that everything works.  Even ``checkTaskStatus:`` is getting called.