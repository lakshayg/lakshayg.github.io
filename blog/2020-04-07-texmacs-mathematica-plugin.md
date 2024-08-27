# Mathematica plugin for TeXmacs

I have used TeXmacs on and off over my college years. Just today, I came across it again when I wanted to write a quick mathematical proof to share with one of my colleagues and did not feel like waiting for LaTeX to install on my machine.

Going through the TeXmacs installation directory, I come across something I wasn't expecting

```sh
$ ls /Applications/TeXmacs-1.99.12.app/Contents/Resources/share/TeXmacs/plugins
...
mathematica
...
```

Mathematica! that's cool, I think to myself. Over the last 2 months or so, I have also been trying to learn some mathematica programming and this seemed like something that could make it more useful for me. I looked around in the TeXmacs GUI but could not find any mentions of Mathematica. Having no clue, I went to the mathematica plugin directory to see if I can find something there. One particular file caught my attention

```
bin/tm_mathematica
```

Without much thought, I execute it

```
$ ./bin/tm_mathematica
Mathematica seems not to be installed
```

That's weird, I'm pretty sure I have mathematica installed. Let's look at `bin/tm_mathematica`.

```
    MATH0=`which math`
    if [ $? -ne 0 ]
    then error "Mathematica seems not to be installed"
    fi
```

Aha! It detects if mathematica is installed by check if the `math` executable exists. I'm not sure why I did not have this executable. I had a vague memory of calling the `math` executable on my university's machine but that was a few years ago. I looked through the mathematica installation directory to see if I could find any clues.

```
$ ls /Applications/Mathematica.app/Contents/MacOS
MathKernel
Mathematica
MathematicaServer
WolframKernel
wolfram
wolframscript
```

`MathKernel` looked promising so I fired it up and it seemed to be just like `math`. At this point, I go ahead and create a script `/usr/local/bin/math`

```
#!/bin/bash
/Applications/Mathematica.app/Contents/MacOS/MathKernel "$@"
```

Now that I had math, the `bin/tm_mathematica` script could proceed. I ran the script again. This time, I see that it tries to compile a C file but fails due to missing headers (mathlink.h) and libraries (libML64i4 and a few others). I look at the script again and see that the rest of the script does a bunch of stuff but most importantly:

1) Create the environment variable `MATHLINK_PATH`
2) Call `make -f Makefile.lazy`

Looking at `Makefile.lazy` which is present in the mathematica plugin directory, I see that it calls

```
gcc -m64 -o $(TEXMACS_HOME_PATH)/bin/tm_mathematica.bin src.lazy/tm_mathematica.c -I $(MATHLINK_PATH) -L $(MATHLINK_PATH) -lML64i4 -lm -lpthread -lrt -lstdc++ -ldl -luuid
```

To figure out what `MATHLINK_PATH` should be, I run:

```
$ locate mathlink.h
/Applications/Mathematica.app/Contents/Documentation/English/System/ReferencePages/Files/mathlink.h.nb
/Applications/Mathematica.app/Contents/Frameworks/mathlink.framework/Versions/4/Headers/mathlink.h
/Applications/Mathematica.app/Contents/Frameworks/mathlink.framework/Versions/4.36/Headers/mathlink.h
/Applications/Mathematica.app/Contents/SystemFiles/Links/MathLink/DeveloperKit/MacOSX-x86-64/CompilerAdditions/mathlink.framework/Versions/4/Headers/mathlink.h
/Applications/Mathematica.app/Contents/SystemFiles/Links/MathLink/DeveloperKit/MacOSX-x86-64/CompilerAdditions/mathlink.framework/Versions/4.36/Headers/mathlink.h
/Applications/Mathematica.app/Contents/SystemFiles/Links/MathLink/DeveloperKit/MacOSX-x86-64/CompilerAdditions/mathlink.h
```

Since `MATHLINK_PATH` has been used both as an include directory and a link directory, it must contain both, a header and a library. `/Applications/Mathematica.app/Contents/SystemFiles/Links/MathLink/DeveloperKit/MacOSX-x86-64/CompilerAdditions` was the only path which met those conditions so set that as the `MATHLINK_PATH` in `bin/tm_mathematica`.

Next, I notice that `Makefile.lazy` referred to `TEXMACS_HOME_PATH`. This was not being set in `tm_mathematica` so I go ahead and also set that in the script.

```
export MATHLINK_PATH="/Applications/Mathematica.app/Contents/SystemFiles/Links/MathLink/DeveloperKit/MacOSX-x86-64/CompilerAdditions"
export TEXMACS_HOME_PATH="$HOME/.TeXmacs"
```

I run `bin/tm_mathematica` again and this time things actually seem to move, however, I again end up with a bunch of linker errors. After a little bit of googling I come across this page https://community.wolfram.com/groups/-/m/t/610879 which talked about linking errors.

Here are the contents of that page:

Question
> On OS X 10.11 with Mathematica 10.0.0.0 while linking I get a slew of errors starting with the following. I am using clang-700.1.76. I suspect I am missing some library in linking, but cannot seem to find where this is coming from. Any help would be much appreciated.
>
> Undefined symbols for architecture x86_64: "_CFDictionaryCreate", referenced from: MLAlertdarwin in libMLi4.a(mlosx.c.o) MLRequestdarwin in libMLi4.a(mlosx.c.o) MLConfirmdarwin in libMLi4.a(mlosx.c.o)

Answer
> In case anyone else runs into this I was able to solve it with the following:
> 
> compile with cc 
> link with g++
> (both are clang++ modulo a configuration prefix and gxx-include dir)
>
> In linking used the flags:
> -stdlib=libstdc++ -framework Foundation -lc++

Reading this, I modify Makefile.lazy to use this:

```
$(TEXMACS_HOME_PATH)/bin/tm_mathematica.bin: src.lazy/tm_mathematica.c
	cc -c src.lazy/tm_mathematica.c -I $(MATHLINK_PATH) -o /tmp/tm_mathematica.o
	g++ /tmp/tm_mathematica.o -L $(MATHLINK_PATH) -lMLi4 -lm -lpthread -luuid -ldl -framework Foundation -lc++ -o $(TEXMACS_HOME_PATH)/bin/tm_mathematica.bin
```

I also had to install `ossp-uuid` to be able to use `-luuid` in the Makefile.

```
brew install ossp-uuid
```


On running `bin/tm_mathematica` again, everything goes smoothly.

Next I start TeXmacs to see if I actually had access to mathematica now.

Insert -> Session -> Mathematica

There it was!

On starting the Mathematica session I get the text "Mathematica" and the usual mathematica prompt "In[1]" in red which I did not like because all that red seemed like there was some error however everything seemed to work fine.

Out of curiosity, I check `src.lazy/tm_mathematica.c` and there I see

```
  fputs("\2latex:\\red Mathematica",stdout);

  while (1) {
    /* Prompt */
    printf("\2prompt#\\red In[%1d]:= {}\5\5",InNum++);
    fflush(stdout);
    if (getline(&input,&size,stdin)>=0) command(input);
  }
```

I change the "red" to "blue", re-run `bin/tm_mathematica`, restart TeXmacs, start mathematica session and sure enough, I see a blue prompt this time. This is all for the time being but there are a few limitations I would like to find a solution to:

1) Plotting does not work
2) Mathlink seems to have been deprecated in favor of WSTP
3) The tm_mathematica script does not work out of the box
