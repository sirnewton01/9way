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

The Plan 9 system has a variety of tools for searching, editing, file management and a command line interface
among many other things. It is tempting to make your own versions. Sometimes it's simple Not Invented Here
syndrome. Other times there are limitations or omissions that don't lend themselves to your particular
use case. Developers re-implement at times to cut their teeth on something different. These pitfalls should
be avoided and in doing so there are a number of advantages.

Re-using system functionality helps to avoid bloating your own program or introducing extra dependencies.
As your code base becomes bigger there are a number of compounding effects. It takes longer to compile, which
in turn makes it slightly harder to find obvious problems while developing it. This may sound silly at first
but it can subtly trip up your thought patterns. More code is more places where things can go wrong. Relying
on a dependency just shifts these problems to a third party and introduces more complexity building and
deploying your program.

Users can more easily understand how to use the system as a whole if there is a great deal of consistency in
how it works. Numerous variations can produce confusion and frustration as it adds more cognitive load.
Popularity of certain walled gardens produced by major technology companies would seem to confirm this since
users know what to expect in the dizzying variety found elsewhere. If you give users a consistent base system
then each skill that they learn can apply in other places in the system including your program, multiplying
their capabilities. This gives the user a great incentive to learn. Gone are the days when users were assumed
to be unwilling to learn and interfaces must be somehow universally intuitive. Even mouse and touch gestures
must be demonstrated to people before they pick it up. Once they do many doors are opened to them, but not all
of them.

So, what kinds of facilities does Plan 9 offer for re-use?

### Tools
### Filesystem
### Namespaces
### Permissions
### Command line
### Manuals
### Windows

## Share memory by communicating

## Prefer plain text data streams

## Short file paths

## Use files to transfer state changes between processes

## Independent executables

## Code that compiles quickly

## Textual UI's

## Reasonable Defaults

## Optimize for readability

## Hide nothing

## Eschew hot-keys

