.. _linking:

################
Linking in Xcode
################

This is an elementary demonstration of linking a library.  I made a new Xcode project, a command line tool in Objective C.  I named it Root because I'd intended to do a math demo with it.  Here is the code:

.. sourcecode:: c

    #import <readline/readline.h>
    #import <stdio.h>

    int main(int argc, const char * argv[]) {
        @autoreleasepool {
            printf("What's your name?\n");
            const char *name = readline(NULL);
            printf("%s is cool\n", name);
        }
        return 0;
    }

Simple, standard C code.  It won't build, as is, because the linker can't find the ``readline`` library.

With the project selected in the file view, and build phases selected in the middle

.. image:: /figures/linking.png
   :scale: 75 %

you should see "Link binary with libraries" (this screenshot was taken *after* linking, so we see ``libreadline.dylib`` in the list.  To add it, click the + button and then search for it in the dropdown list.

.. image:: /figures/linking2.png
   :scale: 100 %

Build the project.  To run it from the command line, select the target (under Products), and then do File > Show in Finder

.. image:: /figures/linking3.png
   :scale: 100 %

and drag the executable to the Desktop (except, make sure the Xcode project is somewhere else before you do that).

Now, do

.. sourcecode:: bash

    > ./Root
    What's your name?
    

Put in your name:

.. sourcecode:: bash

    > ./Root
    What's your name?
    Tom
    Tom is cool
    >

That's it!  