# Builds, Libraries, and General Thoughts on C++

### Installing OpenCV on OSX 10.8

Following several examples online, I attempted to build openCV from source, using cmake and make.  This failed for me.  In the end, I was able to successfully install the library using homebrew... this too wasn't as straightforward as I'd hoped, since I needed to add the homebrew/science repos before opencv showed up as a package:

```bash
$ brew tap homebrew/science
$ brew install opencv
```

That did the trick, installing the necessary dependencies, building opencv, and sticking the dynamic libraries in /usr/local/bin.

Next I needed to figure out how to include the libraries when building a c++ application.  I copied a small example from the OpenCV site, and called it DisplayImage.cpp:

```c++
#include <stdio.h>
#include <opencv2/opencv.hpp>

using namespace cv;

int main( int argc, char **argv ) {
    Mat image;
    image = imread( argv[1], 0 );

    if ( argc != 2 || !image.data ) {
        printf( "No image data \n" );
        return -1;
    }

    namedWindow( "Display Image", CV_WINDOW_AUTOSIZE );
    imshow( "Display Image", image );

    waitKey(0);

    return 0;
}
```

And attempted to build using g++:

```bash
$ g++ -o DisplayImage DisplayImage.cpp
```

Of course, this failed because the compiler couldn't find the OpenCV libs.  So I read a little about linking, and after some trial and error found that I could link to the necessary libs using g++'s -l flag:

```bash
$ g++ -o DisplayImage DisplayImage.cpp -lopencv_core -lopencv_highgui
```

And VIOLA!  I ended up with a little application that opened an image.  Yay!

### The Big Win

Now here's the exciting part:  It's entirely EASY to build OF projects without Xcode!  Who knew?  Turns out the when you create a new project with the Project Generator, it includes a makefile that is already set up to work with make.  In order to get things working nicely with SublimeText, I created a new build system called OF, with the following configuration:

```
{
    "cmd": ["cd '$project_path' && make Debug && make run Debug"],
    "shell": true,

    "variants": [
        { 
            "cmd": ["cd '$project_path' && make && make run"],
            "name": "OF Release and Run",
            "shell": true
        },
        { 
            "cmd": ["cd '$project_path' && make clean"],
            "name": "OF Clean",
            "shell": true
        }
    ]
}
```

With this in place I can hit cmd-b while working in the project and sublime will build the Debug version of the application, then run it!  Wooo hooo!  The other varients make it possible to run the Release build or clean (remove built apps) using the package commands.  (Hit shift-cmd-P, start typing the name of the command.)

One minor caveat: the project does need to be an actual SublimeText project in order for the builds to work, since they use the $project_path variable.

*NOTE:* In order to get the clean running, I did have to modify the compile.project.mk make file.  I added the following after line 310:

```
# in openFrameworks_0.8.0/libs/openFrameworksCompiled/project/makefileCommon, after line 310
rm -rf $(TARGET).app
```

The app that gets built is a directory with the .app extension, so this was necessary to properly remove it.

### Overview of the Process So Far

So here's what I've got now.

    1) Create a new OF project with the projectGenerator
    2) In the terminal, navigate to the generated project dir, and type ```subl .``` to open it in sublime.
    3) Once the project opens, choose *Project->Save Project As...* from the menu and give it the same name as you did in the project generator
    4) If your application ends up with a weird name, it might be because the path to the src has spaces in it.  You can set a name in config.make using APPNAME = SomeName
    5) Start writing code!  When you're ready to build, hit cmd-b.