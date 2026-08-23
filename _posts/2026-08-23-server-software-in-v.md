## Building a Minecraft Bedrock Server in V: What We Learned So Far

Minecraft server software is one of those things that can look surprisingly simple from the outside.

Accept connections, understand the protocol, keep some players in a world, send chunks, handle packets. Done, right? Not quite.

Over the last several months, we've been working on github.com/bedrock-v, an ecosystem of Minecraft: Bedrock Edition projects written in V. A large part of that work has gone into Vedrock, our server framework implementation.

It started partly because we wanted to build something and partly because we wanted to see how far V could be pushed for this kind of software.

### Why V?

There are already mature ways to write Minecraft Bedrock servers.

You can use established software or if you're interested in writing your own implementation, languages such as Go already have projects like Dragonfly (github.com/df-mc) demonstrating that this can work extremely well.

So why V?

There isn't a particularly complicated answer: we, and I personally, like V and we wanted to build with it. (especially with xRookieFight)

And the Minecraft ecosystem around V is tiny. So, why not? :D 

That last part actually made the project more interesting.

There wasn't a huge V library waiting for us that abstracted Minecraft away. A lot of things had to be understood and implemented ourselves.

And that quickly turns "let's make a Minecraft server" into questions about networking, memory, synchronization, serialization, compression and architecture.

### Making It Work Is the Easy Part

One of the first lessons was that there's a massive difference between:

> the player/client joined the server

and:

> this architecture can actually support a server.

Early implementations can be deceptive. You implement enough of the protocol, Minecraft connects, the player spawns, chunks appear and suddenly it feels like most of the difficult work has been done.

Then you add more players, or increase the view distance, or actually measure memory, or profile what happens while chunks are being generated. And everything changes.

Vedrock has gone through multiple architectural changes because solutions that worked perfectly well during early development became obvious bottlenecks once we started treating the server as an actual concurrent system.

When we actually measured our pipeline using our normal generator, the results were roughly:

Generation: ~5.28 ms Serialization: ~3.10 ms Compression: ~1.78 ms
Serialization alone was around 59% of the generation time.

Serialization and compression together represented roughly 92% of the raw generation cost.

That changed how we looked at optimization.
Making the terrain generator 10% faster sounds great, but it doesn't accomplish much if every generated chunk subsequently goes through another expensive pipeline that you're ignoring.

And the question became: "How do we avoid doing expensive work unnecessarily?"
And latest results were like:

![benchmark report](https://raw.githubusercontent.com/nepinhum/blog/main/assets/vedrock/image_1.png)
![benchmark report](https://raw.githubusercontent.com/nepinhum/blog/main/assets/vedrock/image_2.png)
![benchmark report](https://raw.githubusercontent.com/nepinhum/blog/main/assets/vedrock/image.png)
![benchmark report](https://raw.githubusercontent.com/nepinhum/blog/main/assets/vedrock/image_3.png)

Content by github.com/xRookieFight.

### Learning From Existing Software

We're not building this in isolation. One particularly useful reference has been Dragonfly, a mature Minecraft Bedrock server implementation written in Go.
Also, Minestom in Java is another great reference.

Looking at an established project is useful because many seemingly strange architectural decisions begin making sense once your own implementation encounters the problem they were designed to solve.

Shared chunk requests are a good example.

Worker management is another.

There's an important distinction here between copying code and learning why an architecture exists.

Sometimes we've independently reached a problem and later realized that another software already had an elegant answer to it.

### V Has Been Part of the Experiment

bedrock-v has also become an experiment in using V for software that isn't just a small command line utility or application. (Though, Alexander Medvednikov already had proven that V could be used for anything you wish)

That means we've encountered language behavior that we probably wouldn't have cared about otherwise: closure captures, heap backed structures, garbage collector lifetimes, concurrency behavior and interoperability with lower level libraries.

Sometimes we've had to test behavior ourselves rather than rely entirely on documentation. That's simultaneously frustrating and one of the reasons I enjoy the project.

And as V itself develops, that makes the project even more interesting.

### Are We the Fastest?

It's tempting to answer questions like this with benchmarks. I don't think that's particularly useful yet.

We've seen extremely promising results. Vedrock can be remarkably lightweight and V gives us a good foundation for what we're trying to do. But "fastest Minecraft server" is an enormous claim.

Different generators, player behavior, hardware, view distances, network conditions and implementations can completely change a benchmark. So, I'd rather publish numbers together with the workload that produced them and let those results speak for themselves.

What matters to us more is that each version becomes measurably better than the architecture we had before it.

### Where We Are Now

Vedrock today is very different from the program we started with and still has a long way to go before it can compete with established server software. It will probably look very different again several months from now.

There are still plenty of things we want to improve: scheduling, overload behavior, chunk processing, serialization reuse, networking, vanilla features, broken mechanics and the APIs that eventually make a server implementation actually pleasant to build on. That's fine.

We'll see where it goes.

https://github.com/bedrock-v/bedrock-v
https://github.com/vlang/v

Special thanks to 
- https://github.com/AslakOffi
- https://github.com/xRookieFight
- https://github.com/mitsuakki
- and all the other contributors
