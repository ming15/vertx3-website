# Core

At the heart of Vert.x is a set of Java APIs that we call Vert.x Core

[Repository](https://github.com/eclipse/vert.x).

Vert.x core provides functionality for things like:

* Writing TCP clients and servers
* Writing HTTP clients and servers including support for WebSockets
* The Event bus
* Shared data - local maps and clustered distributed maps
* Periodic and delayed actions
* Deploying and undeploying Verticles
* Datagram Sockets
* DNS client
* File system access
* High availability
* Clustering

The functionality in core is fairly low level - you won’t find stuff like database access, authorisation or high level web functionality here - that kind of stuff you’ll find in Vert.x ext (extensions).

Vert.x core is small and lightweight. You just use the parts you want. It’s also entirely embeddable in your existing applications - we don’t force you to structure your applications in a special way just so you can use Vert.x.

You can use core from any of the other languages that Vert.x supports. But here’a a cool bit - we don’t force you to use the Java API directly from, say, JavaScript or Ruby - after all, different languages have different conventions and idioms, and it would be odd to force Java idioms on Ruby developers (for example). Instead, we automatically generate an idiomatic equivalent of the core Java APIs for each language.

From now on we’ll just use the word core to refer to Vert.x core.

Let’s discuss the different concepts and features in core.

