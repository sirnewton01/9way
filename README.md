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

## Re-use system capabilities instead of inventing your own
### Tools
### Filesystem
### Namespaces
### Permissions
### Command line

## Share memory by communicating

## Prefer plain text data streams

## Short file paths

## Use files to transfer state changes between processes

## Independent executables

## Code that compiles quickly

## Textual UI's

## Reasonable Defaults

## Optimize for readability

