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
    Tasker : NSObject {}
    -(void) doSetup;
    -(void) checkTaskStatus:(NSNotification *)notification;
    @end

    @implementation
    Tasker : NSObject

    -(void) doSetup{
        NSNotificationCenter *nc;
        nc = [NSNotificationCenter defaultCenter];
        [nc addObserver:self
               selector:@selector(checkTaskStatus:)
                   name:NSTaskDidTerminateNotification
                 object:nil];
    }

    -(void) checkTaskStatus:(NSNotification *)notification {
        if ([[notification object] terminationStatus]) {
            NSLog(@"success");
        };
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
        [t launch];
        [t waitUntilExit];
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
    2014-09-10 16:32:33.321 task[596:507] VALUE
    2014-09-10 16:32:33.323 task[596:507] <NSConcreteTask: 0x7fc1f2c0c0e0>
    Hello world to me!
    MYKEY VALUE
    >