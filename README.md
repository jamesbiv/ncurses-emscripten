<h1>ncurses for Emscripten (WASM)</h1>

<h2>Introduction</h2>

<p>Here you’ll find the installation instructions and patch required to compile ncurses 6.1+ for Emscripten.</p>

<p>Please take note of the following considerations:</p>

<ul>
<li>Before using this patched version of ncurses it's important that code linking or compiling from this repository use <em>-s ASSERTIONS=1</em> as further debugging of this library is required.</li>
<li>The patch contains a redeclaration of <em>getenv()</em> from <em>stdlib.h</em> which solves the problem described in <a href="https://github.com/emscripten-core/emscripten/issues/6778" target="_blank">issue #6778</a>.</li>
<li>There is no Makefile patching done to the make process itself and since the build requires several native applications to be compiled the make process is not as straightforward as other libraries or applications. Creating a companion make process for both the native and Emscripten elements would be ideal but now for we are just changing the make process manually. Instructions to modifying the relevant Makefiles are below.</p>
</ul>

<p>Lastly, a large part of this work can be credited to <a href="https://github.com/ilyaigpetrov" target="_blank">ilyaigpetrov</a> found in his repository (<a href="https://github.com/ilyaigpetrov/ncurses-for-emscripten" target="_blank">ilyaigpetrov/ncurses-for-emscripten</a>).</p>

<p>Also, it’s worth noting that there are several other projects of a curses nature out there and if you’re looking at the most optimal solution for curses for Emscripten each of them have their merits.</p>

<ul>
<li><a href="https://github.com/sabotage-linux/netbsd-curses" target="_blank">https://github.com/sabotage-linux/netbsd-curses</a></li>
<li><a href="https://github.com/wmcbrine/PDCurses" target="_blank">https://github.com/wmcbrine/PDCurses</a></li>
<li><a href="https://github.com/rhaberkorn/emcurses" target="_blank">https://github.com/rhaberkorn/emcurses</a></li>
</ul>

<h2>Installation</h2>

<p>It's recommended to create a separate directory to build this project in. For example <em>ncurses-emscripten/</em>.

<h3>Downloading and patching ncurses</h3>

<pre>~$ wget https://ftp.gnu.org/pub/gnu/ncurses/ncurses-6.1.tar.gz</pre>

<pre>~$ wget https://raw.githubusercontent.com/jamesbiv/ncurses-emscripten/master/ncurses-6.1_emscripten.patch</pre>

<pre>~$ tar -xzvf ncurses-6.1.tar.gz</pre>

<p><b>Note:</b> Because this is a source tree level patch make sure the patch is in the directory before the installed directory. For example, if the directory for ncurses is <em>/home/user/ncurses-6.1</em> then make sure the patch is located at <em>/home/user/ncurses-6.1_emscripten.patch</em>.</p>

<pre>~$ patch -p0 < ncurses-6.1_emscripten.patch</pre>

<pre>~$ cd ncurses-6.1</pre>

<h3>Configure and make the native components</h3>

<p><b>Note:</b> Using <em>--prefix</em> defines the installation path for ncurses. Feel free to change <em>--prefix</em> to anything you wish, just make sure you specify the same directory below when calling <em>emconfigure</em>. Further, it’s recommended to use <em>--prefix</em> because you really don't want this to be a global library either way.</p>

<pre>~$ ./configure --prefix=`pwd`/build</pre>

<pre>~$ make && make install</pre>

<h3>Configure and make the Emscripten components</h3>

<p><b>Note:</b> Using <em>--prefix</em> defines the installation path for ncurses. Make sure you use the same path you've described above.<p>

<pre>~$ emconfigure ./configure --prefix=`pwd`/build</pre>

<h3>Modifying the Makefiles for Emscripten</h3>

<pre>~$ nano -w ./Makefile</pre>

<p>Comment out the following lines of code using <b>#</b> to reflect the following:</p> 

<pre>
Line 113
#cd man && ${MAKE} ${TOP_MFLAGS} $@
Line 116
#cd progs && ${MAKE} ${TOP_MFLAGS} $@
Line 120
#cd test && ${MAKE} ${TOP_MFLAGS} $@
Line 121
#cd misc && ${MAKE} ${TOP_MFLAGS} $@
Line 122
#cd c++ && ${MAKE} ${TOP_MFLAGS} $@
</pre>

<p>Save and exit nano</p> 

<pre>~$ nano -w ./ncurses/Makefile</pre>

<p>Comment out the following lines of code using <b>#</b> to reflect the following:</p> 

<p><strong>From line 233</strong></p>

<pre>
#make_keys$(BUILD_EXEEXT) : \
#               $(tinfo)/make_keys.c \
#               names.c
#       $(BUILD_CC) -o $@ $(BUILD_CPPFLAGS) $(BUILD_CCFLAGS) $(tinfo)/make_keys.c $(BUILD_LDFLAGS) $(BUILD_LIBS)

#make_hash$(BUILD_EXEEXT) : \
#               $(tinfo)/make_hash.c \
#               ../include/hashsize.h
#       $(BUILD_CC) -o $@ $(BUILD_CPPFLAGS) $(BUILD_CCFLAGS) $(tinfo)/make_hash.c $(BUILD_LDFLAGS) $(BUILD_LIBS)

#report_offsets$(BUILD_EXEEXT) : \
#               $(srcdir)/report_offsets.c
#       $(BUILD_CC) -o $@ $(BUILD_CPPFLAGS) $(BUILD_CCFLAGS) $(srcdir)/report_offsets.c $(BUILD_LDFLAGS) $(BUILD_LIBS)
</pre>

<p>Comment out the following lines of code <em>in the same Makefile</em> using <b>#</b> to reflect the following:</p> 

<p><strong>From line 283</strong></p>

<pre>
#       -rm -f make_keys$(BUILD_EXEEXT)
#       -rm -f make_hash$(BUILD_EXEEXT)
#       -rm -f report_offsets$(BUILD_EXEEXT)
</pre>

<p>Save and exit nano</p> 

<h3>Compiling and installing for Emscripten</h3>

<p><b>Note:</b> Ideally <em>make clean</em> should be committed before the <em>emconfigure</em> but because we are preserving parts of the native build we commit the <em>make clean</em> after the Makefile modifications.</p>

<pre>~$ make clean</pre>

<pre>~$ emmake make && emmake make install</pre>

<pre>~$ cd build <em>(or whatever directory you've asked it to install too)</em></pre> 



