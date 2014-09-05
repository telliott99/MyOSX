.. _dlopen:

############
Using dlopen
############

The next example (as before, it is from)

http://www.amazon.com/Advanced-Mac-OS-Programming-Guides/dp/0321706250

uses ``dlopen``, which is declared in the header file ``dlfcn.h``.  

We use the same code for ``simplemessage.m`` from before but modified to C-syntax.  The original looks like this:

.. sourcecode:: objective-c

    #import <Foundation/Foundation.h>
    #import "BundlePrinter.h"
    @interface SimpleMessage:NSObject <BundlePrinterProtocol> {}
    @end
    @implementation SimpleMessage
    + (BOOL)activate {
        NSLog(@"SimpleMessage plug-in activated");
        return (YES);
    }
    + (void)deactivate {
        NSLog(@"SimpleMessage plug-in deactivated");
    }
    - (NSString *)message {
        return (@"This is a simple message");
    } @end

the modified version is:

.. sourcecode:: c

    #import <string.h>
    #import <stdio.h>

    int activate(void) {
        printf("SimpleMessage plug-in activated");
        return (1);
    }
    void deactivate(void) {
        printf("SimpleMessage plug-in deactivated");
    }
    char *message(void) {
        return (strdup("This is a simple message"));
    }

We put the code in ``simplemessage.m``

And then do 

.. sourcecode:: bash

    clang -Wall -o simplemessage.msg -bundle simplemessage.m

The code that will use simplemessage.msg is next.  It's a bit long because it uses various libraries.  Put it in the file ``bundleprinter-dl.m``

.. sourcecode:: c

    #import <dirent.h>
    #import <stdlib.h>
    #import <stdio.h>
    #import <errno.h>
    #import <string.h>
    #import <dlfcn.h>

    typedef int (*ActivateFP) (void);
    typedef void (*DeactivateFP) (void);
    typedef char * (*MessageFP) (void);
    char *processPlugin(const char *path) {
        char *message = NULL;
        void *module;
        module = dlopen(path, RTLD_LAZY);
        if (module == NULL) {
            fprintf(stderr, "couldn't load plugin in at path %s.  error is %s\n",
                path, dlerror());
            goto bailout;
        }
        ActivateFP activator;
        DeactivateFP deactivator;
        MessageFP messagator;
        activator = dlsym(module, "activate");
        deactivator = dlsym(module, "deactivate");
        messagator = dlsym(module, "message");
        if (activator == NULL || deactivator == NULL
           || messagator == NULL) {
             fprintf(stderr,
                 "could not find message symbol (%p %p %p)\n",
                 activator, deactivator, messagator);
             goto bailout;
        }
    
        int result;
            result = (activator)();
            if (!result) {  goto bailout; }
            message = (messagator)();
            (deactivator)();
            bailout:
            if (module != NULL) {
                result = dlclose(module);
                if (result != 0) {
                    fprintf(stderr,
                        "could not dlclose %s.  Error is %s\n",
                        path, dlerror());
        } }
            return (message);
        }
        int main (int argc, char *argv[]) {
            DIR *directory;
            struct dirent *entry;
            directory = opendir(".");
            if (directory == NULL) {
                fprintf (stderr,
                    "could not open directory for plugins\n");
                fprintf(stderr, "error: %d (%s)\n", errno, strerror(errno));
                exit(EXIT_FAILURE);
            }
            while ((entry = readdir(directory)) != NULL) {
                if (strstr(entry->d_name, ".msg") != NULL) {
                    char *message;
                    message = processPlugin(entry->d_name);
                    printf("\nmessage is:  '%s'\n", message);
                    if (message != NULL) { free(message); }
        } }
            closedir(directory);
            return EXIT_SUCCESS;
        }

.. sourcecode:: bash

    > clang -g -o bundleprinter-dl bundleprinter-dl.m
    > ./bundleprinter-dl
    SimpleMessage plug-in activatedSimpleMessage plug-in deactivated
    message is:  'This is a simple message'
    >

