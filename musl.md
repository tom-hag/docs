[musl](https://musl.libc.org/) is a POSIX-compliant implementation of the standard c library for linux that is often used **when static linking is preferred**. Given this, it is useful in deployments where you may want predefined depenedencies and statically linked libraries.
**It also has a small footprint**.

It is therefore useful in  embedded and containerised environments such as docker, and is used in alpine images when requiring standard c libraries (e.g. when building a rust application). 

Finallly, it is consdiered more secure than glibc


