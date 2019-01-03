# The Plan 9 Way

"The UNIX Way" is a fairly well known and understood set of principles for software design.
All things being equal large systems should be decomposed into smaller reusable tools that
attempt to focus on one area of expertise. These tools should provide simple interfaces and
strive for simple implementation. In order to accomplish complex tasks the tools can be
composed together using communication mediums such as pipes and files that are easy for
users and programs to hook the tools together. There are limits to this approach due
to pressures from real design problems and also from limits inherent in UNIX itself.
The result is that much software deviated from these principles or never applied
them at all, including the newest generation of UNIX systems.

Plan 9 is a newer system developed by some of the original UNIX team from AT&T Bell Labs.
It was rewritten from the ground up fixing a number of fundamental problems with the
aging UNIX system. Some of the original limitations of UNIX are gone, which removed
certain roadblocks for Plan 9 to achieve better software design.

This article attempts to capture new capabilities and design patterns after several months
of investigating the Plan 9 system both as a user and a programmer. It is felt that
some of the original limitations of UNIX held back the development of popular software
systems preventing them from ever attempting the UNIX way. Now that Plan 9 has had time
to mature it's time to showcase what has been done there.

## Re-use the system where possible

The Plan 9 system has a variety of tools for searching, editing, file management and a command
line interface among many other things. It is tempting to make your own versions. Sometimes it's 
simple Not Invented Here syndrome. Other times there are limitations or omissions that don't lend
themselves to your particular use case. Developers re-implement at times to cut their teeth on
something different. These pitfalls should be avoided and in doing so there are a number of 
advantages.

Re-using system functionality helps to avoid bloating your own program or introducing extra
dependencies. As your code base becomes bigger there are a number of compounding effects. It
takes longer to compile, which in turn makes it slightly harder to find obvious problems while
developing it. This may sound silly at first but it can subtly trip up your thought patterns.
More code is more places where things can go wrong. Relying on a dependency just shifts these
problems to a third party and introduces more complexity building and deploying your program.

Users can more easily understand how to use the system as a whole if there is a great deal of
consistency in how it works. Numerous variations can produce confusion and frustration as it
adds more cognitive load. Popularity of certain walled gardens produced by major technology
companies would seem to confirm this since users know what to expect in the dizzying variety
found elsewhere. If you give users a consistent base system then each skill that they learn can
apply in other places in the system including your program, multiplying their capabilities. This
gives the user a great incentive to learn. Gone are the days when users were assumed to be
unwilling to learn and interfaces must be somehow universally intuitive. Even mouse and touch
gestures must be demonstrated to people before they pick it up. Once they do many doors are
opened to them, but not all of them.

So, what kinds of facilities does Plan 9 offer for re-use? As a successor to UNIX the base
system is focused on the concept of a file. It takes the original idea that "everything is a
file" further than UNIX ever could. As expected, a file can be read or written as a sequence of
bytes. There is metadata, such as the last modification date, owner and group. Plan 9 adds
attributes to each file, such as version and append-only. While these attributes have somewhat
obvious use for storage-based filesystems they can also be used to model interfaces to
application state. Indeed, it is quite straight forward for a program to have its own filesystem
in Plan 9. Those filesystems can be mounted and exposed to other processes as files so that
any tool that works with files to interface with the program, including the core tools like
ls, grep, cat or even the rc shell. It is worth noting that any user of the system can mount a
filesystem, not just an administrative account, and each process can have its own mounts, which
really opens things up.

Plan 9 has its own manual system inherited from UNIX called "man." You can document different
aspects of your program such as a synopsis, command line arguments, environment variables and
even details of the filesystem it exposes. The manuals are text-only and usually quite brief,
which helps to avoid documentation bloat. If every program contributes useful manuals to the
built-in system then users can more easily find it and read it in a consistent format. There
are times where additional materials helps to explain more background or even provide tutorials.
Documents meant for lengthy reading like this are often written as PDF or PostScript and kept
in the /sys/doc directory. There is a PDF/PS reader available so that you can read them at your
leisure.

Each process is given a window to work with. It is considered bad form to open a new window
unless the user drew the window to the size and position that they want for their task. Any kind
of pop up dialog boxes and separate panels should be kept within the bound of the main window.
This does much to avoid so much window clutter and helps the user to retain control of their
screen real estate and layout. The protocol for interacting with the graphics is exposed as a
set of special files allowing a program written in any language to perform basic graphics
operations. If the program is written in C there are extra libraries available that can help
use frames as an example. There should be no need to build your own windows, windowing
system or widget toolkits because the facilities are all there.

In some cases programs need to customize the view of the system. For example, the program will
be running code that is not very trustworthy. Also, some programs want to be able to provide
their own filesystem interface to child programs. These scenarios are typically handled with
virtual machines, sandboxes, jails, containers or custom plugin systems. Plan 9 covers much of
the functionality with namespaces. Each process gets its own view of the file trees, can
customize those views for itself and any child processes. Since the vast majority of interactions
with other processes or the operating system itself happen through file-based protocols
customizing the available files using namespaces controls the entire world view of the process
with this one capability.

The Plan 9 creators provided really useful building blocks that work well together. As a
program writer it is really worth considering how to re-use those elements in your program
rather than fighting against them or rewriting your own version. As with anything else in the
domain of software engineering there are tradeoffs, but this is definitely worth a consideration
when engineering your solution.

## Share data by communicating

The Go programming language carried this same idea forward from Plan 9. Avoid creating a global
data block and controlling access to it. Instead, allow different entities in the system to
get copies of that data. Plan 9 has a number of ways to achieve this result. You can structure
your program in such a way that it can pipe the data from input to output statelessly. This is
of course the ideal case, but it's not always possible for long running processes.

Plan 9 programs can expose their internal state as filesystems. Data is copied out using a
file descriptor to another process with simple read operations. Similarly, external replicas
of the data can be written to the file to copy the data to the program. This resembles a
RESTful (Representational State Transfer) design used in many web-based systems, but can more
seamlessly be applied to internal system process communication, and was invented decades before
REST was formalized.

Within a single process and its forked sub-processes there is a channel library available to
C programs that allows you to copy data in a way that it is light weight and safe for concurrent
access. Incidentally, the Go programming language also has its own channels that are inspired by
the Plan 9 channels. In both cases, this can be an effective tool to avoid complex and error
prone locking logic in your program.

## Prefer plain text data streams

As in UNIX, plain text is preferred over binary data. Text can more easily be read by a human.
A variety of tools can work with it including quite a number of system tools, such as grep, awk
and sed. Text can be viewed almost anywhere on nearly every operating system making it more
ubiquitous and future-proof. Text is much smaller than most document formats and so it is
faster to copy or transmit. Plain text is really easy to type and edit.

In the past it was more difficult to handle text encoded for a different language. There were
special encodings for different language. Often times the user would be expected to know the
encoding for a file before opening it. With the advent of Unicode and the UTF-8 encoding these
problems are much less common. In fact, Plan 9 was the first operating system to adopt the UTF-8
encoding and to make it mandatory across the entire system lessening the burden on the user.
Other systems have proven very slow to adopt it.

For the various protocols used in Plan 9, which operate using special files, many of them are
text-based making it more natural for end users to interact with them. There are exceptions of
course where performance is major concern or other technical reasons. The text-based protocols
are much easier to learn, understand, interact and even mock. If you want to move the mouse
to a specific place you can echo a set of coordinates to /dev/mouse and the pointer will move.
There are numerous other examples.

## Shorten file paths

File paths can get long and unwieldy. There are a number of reasons for this. Average users
aren't generally allowed to put files in top-level directories because those are shared with
other users. This tends to force users to put their own files in their home directory where
they have permission to place them. References to these files in various places retain these
longer paths and generally get multiplied by all of the environment variables, configuration
files and documents that refer to them or subdirectories.

Plan 9 gives you the ability to bind files from one place into another on a per-process basis.
For example, a user can bind their favorite programs into the root /bin directory, for only their
processes. No other user needs to be impacted by their customization. This also explains why
Plan 9 doesn't have multiple directories for programs (e.g. /usr/bin, /usr/local/bin) since
multiple locations can be bound into the single /bin. It's also why the PATH environment variable
isn't used very much anymore. The same applies to process filesystems, such as acme, which
mounts its filesystem in /acme making it much easier for scripts to interface with it with the
shorter paths.

Sometimes, insufficient care is taken to name the files and directories themselves. It is nice
to have the freedom to name something "My lovely essay about horses and ranches in southern France.txt" it is often better to give it a name that is unique and terse so that it is easier to
type frequently. Also, the spaces are not prohibited but can make lists of files harder to
read. If you need to find a file based on its contents you can often use grep to scan a list of
files for a snippet or pattern. For common files and paths the manual can provide these details
and explain the mnemonics.

## Self-contained executables

As you have seen in previous sections, the Plan 9 systems is very flexible in the
customization of the file tree (ie. namespace). As a result, there can be considerable variation
in the available files. As a result, program executables tend to minimize their dependencies
on other files because they may not always be available in a context. This is why the system
doesn't support dynamic libraries (e.g. shared objects or DLL's). Each executable is statically
linked to include what it uses and exclude what is unnecessary. This gives the executable a
much higher chance of running in a minimal namespace or sandbox where some dependencies
are not available. This same design decision was carried forward with Go, which has given it an
advantage when it comes to deployment over other languages that have heavy and complex runtime
dependencies. Containers were built as a complicated solution to the problem of deploying
programs with complex dependencies. Plan 9's use of namespaces and statically linked executables
proved to be a much simpler and elegant solution.

## Compile quickly

One aspect of software design that is often overlooked is the time spent waiting for a
program to compile. There is of course a natural tendency for programs to become larger, which
increases the time it takes to compile it. Some compilers and code base can take hours to
compile, making it much more difficult to iterate on changes to the program. It can also lead
to very complicated instructions for incremental compilation, convoluted build scripts or even
tactical compiler solutions, such as pre-compiled headers. In the worse cases, programmers
have given up on compiled languages entirely preferring instead interpreted languages losing out
on the benefits.

Compile times are affected by the size and nature of the code base, but they are also affected
by the compiler design and the language itself. Plan 9 addresses the compiler and language part.
On a typical modern system the entire system can be recompiled in just a few minutes, including
kernel, compilers, user applications and everything, which is quite impressive and makes
installing updates to the system quite trivial. This is made possible by a concerted effort to
keep the code bases for programs as small as possible while taking advantage of the compiler
speed.

## Textual UI's

For the most part, Plan 9 does not use icons in the UI. Instead, text is used since it can be
more easily recognized than an icon. It is easier to search for a snippet of text in a manual
than an image, which would require scanning the entire document. The most obvious exception to
this rule is the mouse cursor, which is sometimes used to indicate program states, such as busy
or even prompts for the user, such as "ok?" The latter is in fact a text cursor, which is about
the only example that comes to mind.

The acme tool actually makes use of the text actions in the UI by allowing the user to modify
them directly in menu bars to suit their needs. In other icon-based tools this usually requires
a specialized menu configuration panel so that the user can pinpoint the icon and drag it in
or out of the menu and order it. It doesn't seem like a serious concern, but adds that extra
bit of work for the user and code complexity.

While Plan 9 has a built-in graphics and windowing system, much of the system uses these
capabilities sparingly. As with the icons described above there are design problems faced when
deviating from text. Text is easily typed, searched, edited, copied and stored, while graphics
are more restrictive. The system as a whole tends to use graphics only when it adds real value
to the user workflow. Clearly, something like map rendering or image viewing is better presented
in a graphical UI than somehow as text. The point is that graphics should be used sparingly as
it can also be distracting. The same arguments can be made of other media, such as audio.

## Use sane defaults

For every user, there will be a time when they will use your program for the first time. Before
they have had a chance to peel back the layers of the onion and appreciate the full capability
they will need to try it. As mentioned earlier, the manual can provide a brief overview of the
primary elements to get them started, but only if there aren't so many mandatory parameters.
If the default values for optional parameters are unsuitable in many cases then it will make
it more difficult for a new user to become comfortable with it. Aggressively minimize the
number of options to avoid many of these pitfalls.

Reasonable defaults should also try to conform to the established conventions of the system to
improve usability. There are certain well known directories where most users will expect to
find certain things, such as the user's home directory. There are key and mouse bindings that
a number of programs share, such as left-click positions or selects things while right-click
opens or searches for things.

## Optimize for readability

There is a strong emphasis on making it easier for users to read things. Code is optimized to
make it easier to read through the use of small functions with clear names. It is organized
into different files and directories according to functionality. The UI is generally sparse with
little window trim or menu bars so that the content is afforded the additional space and easier
to read. The focus is on the content and less on anything that might distract from it.

## Eschew hot-keys

Many traditional computer systems made heavy use of special key strokes to perform various
actions. The idea was that it would help users to perform actions more quickly. In an era where
all interfaces were text-only this may have been the case. The Plan 9 team found that point and
click with the mouse was actually more efficient in many cases and so they minimized the number
of hot key combinations used in the base system (e.g. Alt-F4 and Cmd-Q) preferring instead either
full text commands that are visible on the screen, such as Get or Snarf, or mouse gestures
using one or more of the buttons. Sometimes there is even combinations of both with some very
interesting results. Especially now that most computer users are familiar with mouse driven
graphical interfaces this choice seems to be particularly well suited to the current state.
Many developers are already using mouse-driven UI with command-line hybrids, albeit
with outdated and limited integration points and with the outdated hot-key concepts hanging
around.
