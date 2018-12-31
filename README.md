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
aspects of your program such as a synopis, command line arguments, environment variables and
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
access. Incidently, the Go programming language also has its own channels that are inspired by
the Plan 9 channels. In both cases, this can be an effective tool to avoid complex and error
prone locking logic in your program.

## Prefer plain text data streams

## Short file paths

## Use files to transfer state changes between processes

## Independent executables

## Code that compiles quickly

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

## Reasonable Defaults

## Optimize for readability

## Hide nothing

## Eschew hot-keys

## Give users control over their screen
## Give content the focus
