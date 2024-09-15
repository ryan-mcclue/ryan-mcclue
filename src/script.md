<!-- SPDX-License-Identifier: zlib-acknowledgement -->
It utilises my personal base C library that includes things like generational handles, a performance profiler, vector math, memory arenas and length based strings.

When I build the program, vscode first launches a compile task that runs my custom bash build script.
This collates common build flags across debug and release builds.
It will then detect the most recently created elf file and launch it in the debugger.

Hi my name is Ryan McClue, and here is a demonstration of my music visualiser program.

On startup, the text will flash red according to the most recent mouse movement.
I find this simple interaction greatly improves the user experience.

If I drag a host of music files into the program, the most recent one begins to play.
The display is an FFT visualisation of the music file.

As the distance between the maximum and minimum frequencies in a waveform are large, 
I converted to a logarithmic scale to make the display more uniform. 

I can pause the music.
Resume it, rewind it and fast forward it.

If I drag more files into the program, a scroll bar will appear 
and I can select other music files to play.

ACTION: 
As the program is hot-reloadable, 
I can change the color of the active music file to orange for example.

This is particularly useful as it gives you the freedom to make code changes without worrying
about having to restart the application and get back to that current state.

This music correlation score at the bottom is a crude indicator of how closely 
the current song's name and length align to a collection of 1million other songs.
The data is stored in a simple binary file which is much faster than text based files like json.
At startup, two threads are launched that asynchronously read and process the file in 4K 
sized ping-pong buffers so all memory operations fit in the L3 cache.

Thank you for watching this demonstration and have a great day.
