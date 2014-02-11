Note
====

This text was the basis of the presentation and creates a ~30 minute presentation. It was cut down such that the actual presentation was ~20 minutes, allowing time for questions. The actual presentation content can be seen by watching a video of the presentation.

Roughly, each paragraph is a slide - where there is a line break, that means it relates to the next slide. They may not match up exactly due to slides that were added or removed without this text being changed.

The text
--------

Good afternoon, I’m Hayden and this is James, and we’re here to talk about EACOF, a framework for monitoring the energy consumption of software on desktop systems.

Here’s a quick overview of the topics we will be talking about over the next half an hour
First we’ll be looking at the problem that exists and the reason why we started to develop EACOF –
Briefly, this is the problem of software draining power, but currently it is hard to work out why or how the battery is draining.
Following this, we’ll describe how we’ve designed EACOF to overcome the hurdles and allow you to easily solve the energy-measurement problems you face.
This will include taking a look at how EACOF can be integrated into your software to measure energy consumption and the internals that allow this to happen, allowing you to both use EACOF and how you can alter the internals EACOF to ensure it is the perfect fit for your target goal.
Finally will be a quick run through potential use cases for EACOF, and examples of where energy monitoring is currently in use.

So, firstly what’s the problem that EACOF’s trying to solve?
Software uses energy. It is easy to fall into the trap that it is only the hardware that is uses it, but in fact the only things using it are the programs running on the hardware. As the software becomes more complex, and utilises more hardware, the consumption of the hardware goes up.  This shows that in order to keep the energy usage of the hardware low, we need to ensure the software and applications are only using the minimum amount of hardware, and therefore energy, required to complete their task.

The main area you’re likely to feel the impact of software using energy is when using mobile devices. Be it your phone, tablet or laptop, they each have a battery contains a limited amount. Once it’s been used up, you’ve got to find somewhere to charge it. On phones and tablet, at least with Android (I’m not an iOS user), it’s easy to get a rough idea of how much energy is being used by looking in the Battery setting. On desktop systems, it’s a bit harder, though Apple’s introduced a bunch of new features to Xcode and Activity Monitor in OS X Mavericks which provide information about hardware wakeups and other activities which can impact battery life.

With this exposed information about which applications use the most energy, users can see if your application is misbehaving when their battery is going flat in a couple of hours. Ensuring that you’re as low as possible on the list of “applications using most energy” is a sign of respect for your users and can provide an additional tangible unique selling point. It’s not about reducing functionality to decrease energy usage - a video player needs to spin up the CPU to decode the video and a web browser needs to download content from the internet. What it is about is being sensible - decoding lower resolution assets if the screen’s got a lower resolution, batching network requests, letting the device sleep when not in use. Do these things and users (yourself included) will notice the benefits.

It’s all well and good wanting to do these things, but you need to have the data in front of you to be making the correct decisions. On mobile there are a few APIs here and there which can help you, but on the desktop, so covering laptops with limited battery capacities, it can be a bit of a pain.

These systems are badly instrumented. You’ll have the odd bit of data you can easily collect from hardware. In particular, you can measure battery drain when unplugged, with about a 1Hz resolution. Modern Intel CPUs can provide information about their energy consumption at about 1KHz. Beyond that, however, you’re falling into having to use some rather complex heuristics to determine energy consumption on a standard consumer grade laptop. If you have the budget, you can purchase energy monitors and put together a few fully instrumented systems to measure how much energy your software uses for each individual system component.

However, even if you have all these fancy systems, each system has its own API and learning a thousand and one different APIs to do pretty much the same thing is, well, tiresome and tedious. But, as I say, on a standard machine at the current point in time there are only a few pieces of hardware which let you directly measure their energy consumption.

As such, you’re possibly limited to heuristic measures to determine how much energy components are using, and getting this right is HARD. PowerTOP, produced by Intel’s Open Source Technology Center, attempts this task, though it’s not really something you want to be doing yourself.

I’m talking about heuristics being hard to get right, however currently this is the only widely available method of gathering energy data. More components are coming with ways of directly measuring how much energy they’re using, and you’ll want to be making use of them when they become available. If you’ve compiled some heuristic method into your software,  and a new version or an alternative is provided, changing the method involves recoding the current implementation and also with many different methods available on different platforms, impacts the portability of the code. What we want is a way to say “technology is moving fast and it is infeasible to continually update software to keep up with new tech, so can we not just gather the best data there is, without changing the applications code?”. A method that will let our applications improve over time with zero effort on our behalves.

What it sounds like we’re looking for is EACOF.
EACOF is an Energy Aware COmputing Framework designed for monitoring desktop power consumption.
It’s written in C to act as a lowest common denominator between languages. In theory this will allow bindings to other languages to be easily developed, though we’ve not got round to doing that yet.
It’s currently been tested on OS X and Ubuntu. However, due to the internal design, it is possible to replace a few function calls and generate a working version for Windows, but that’s not been a priority.
Going back to the problems I just mentioned about monitoring energy consumption. EACOF’s been designed to overcome them so that there’s one way to access energy data without having to worry about plugging in wires to odd looking hardware or learn a dozen APIs or altering the code to fit new technology that springs up in the future.
The aim is for it to be useful wherever you want to measure energy data, so it’s available under a BSD 2-clause license and can be downloaded from Github at github dot com slash eacof, which also has docs and other such helpful examples.  
Now that I’ve given an overview of the state of things, I’ll let James explain EACOF’s internals and how it can be used. James.

--- swap presenter

Thanks. So, you may well be asking, how exactly does EACOF work, and how does it solve the problems Hayden has mentioned?

Firstly I’ll give you a brief overview of EACOF. It is split into three components. The first is a Provider which collects energy information from the hardware or operating system. This passes the collected data on to the Central Authority, a component which keeps track of the current state of affairs and performs some internal calculations which I will explain later. Finally are Consumers. These are applications which, in some way, use the energy data that has been collected through the use of EACOF’s simple, yet powerful, API. Each of these components is compiled as a separate process, with communication occurring between them so that data can pass from Providers through to Consumers.

However, this isn’t it, this just shows the most basic scenario of just one energy monitor.
So, lets make this a bit more interesting. What if you wanted to access energy data from three different Providers in your application? Perhaps whole system information from the battery, along with more specific data from the CPU and hard drive. Ordinarily you’d have to use three different methods to get the data, while with EACOF the data flows through the central authority and provides you, as the application developer, with a single method of getting all of this different data. Simply tell EACOF the types of device you wish to monitor, and it will automatically gather all of the required data from those devices if present. 

But what if it’s not just you writing your one energy-aware application? What if there are many different applications wishing to access the same data? Maybe an email client, video player, and web browser. Normally this would instantiate many concurrent processes providing the same data. However, EACOF simplifies things. By using the Central Authority, the data only needs to be collected once, improving performance for the end user, while the application developers need not worry about the different sources of data being utilised.

But we’re not done yet, what if we wrapped this situation into a bundle on one machine and then realised there are lots of machines out there!

You take your multiple machines, each running EACOF, and connect them together. Because of the communication model, Consumers and Providers need not be on the same machine. This enables applications such as network-based energy-monitoring systems to be developed, allowing a system administrator to see who and what on their network is using the most energy. Server stuff

OK, so that’s the general idea, but how is it used?

First, lets take a look at a Provider. A basic Provider will be an application which periodically obtains energy readings, waits for a short amount of time, then obtains another reading. Still keeping it simple, this is an act which may continue happening constantly, or at least until the machine is powered off. This basic Provider will collect the information, but isn’t doing anything with it. Lets see how it looks when plugged in to EACOF.

As you can see, it takes the addition of just three function calls and an include to plug this Provider into EACOF and make the data available to any interested Consumers. So lets run through these line by line, The include, well, includes EACOF into the application.
initEACOF() performs the initialisation, allocating memory and connecting to any other EACOF-aware components. In this case, it’ll be letting the central authority know that it’s appeared, and is ready to transmit data.
createProbe() will create a Probe. This is basically a data structure stating the devices that a Provider can provide information about. So, if this Provider provides information about the combined energy usage of the CPU and hard drive, it’ll create a Probe containing two devices - the CPU and the hard drive. This data will be passed on to the central authority so that it can intelligently match Providers to Consumers based on current capabilities.
AddSample() will add a sample, which is an amount of energy in Joules. Each sample is matched against a Probe, allowing a single Provider to provide data about a range of sets of devices without needing to perform the same work repeatedly in different Providers.
OK, so that’s a look at Providers. Now lets take a quick look at an example Consumer.

A Consumer is essentially an application, so lets look at an example which does something really trivially silly lots of times. As you can see, it’s a hello world, but with a different output. Lets see what this code looks like when plugged in to EACOF.

This time we have the same include and the same initialisation function call, all components do that part the same. The next step is where the consumer differentiates from the provider. Instead of creating a Probe, we’re creating a Checkpoint.  
Checkpoints are a means of sampling the amount of energy used by one or more devices between two points in time. The first call, setCheckpoint() defines the devices being monitored and sets an initial point in time to take readings from. Hopefully there’ll be a Provider which is monitoring the specified devices, allowing a pairing to be made by the central authority. If not, errors propagate and can be handled accordingly.
With the Checkpoint created, we’re going to sample it. This will work out the amount of energy that was used since the checkpoint was either created or last sampled. In practice you probably want to put these around blocks of code rather than within a tight loop like this. This’ll reduce any performance hits and give you more meaningful results - current CPU measurement, for example, is limited to 1KHz with a resolution of 15.3 micro-Joules, which would often give you results of zero with the code as is. Finally comes the DeleteCheckpoint() function which stands to clean up the data structure within the Central Authority, ensuring no loose ends are left.

So that’s basically how you use it. There aren’t many functions to remember, even less if you’re only concerned about Providers or only concerned about Consumers. There are a couple of calls up there that weren’t mentioned, but they’re more for completeness in some odd situations rather than for everyday use. If you’re interested, they’re explained in the docs.

That Consumer example was a bit contrived, so lets look at something more real. We took some code which sorts the same set of numbers using different sorting algorithms, and plugged in EACOF, measuring the amount of energy used by the entire system in the time the algorithm takes to run. I expect you vaguely know of these different sorting methods, and their corresponding time complexities. You can see the output here, and it’s fairly expected. The more efficient algorithms used the least energy. Now lets take a look at another less trivial example.

Lets take a look at the basis of the source code to generate this test. Again, as you can see it is extremely simple to insert energy monitoring into the code, by using EACOF, we only needed to add five lines of code to our program, and that allowed us to measure the energy consumption.

So that those examples have shown how easy it is to implement EACOF and I have described the basic structure behind it.  To have a look deeper down, I will hand back over to Hayden who will explain the internals of each section.

--- swap presenter

EACOF is modular. As has been shown, Providers, the Central Authority, and Consumers are compiled as separate processes. This means that if a new data source comes along and a Provider is written for it, any EACOF-aware Consumers can make use of the new data without having to change their code at all. Seems pretty neat. There are a few additional overheads because of the separation of tasks, but they’re fairly small, and everything could be compiled into a single binary is so desired.

EACOF has a simple API. There are basically CRUD, create, read, update, delete, tasks for each Providers and Consumers, and, the Consumer side doesn’t even have an ‘update’. It really is that simple, while also being extremely powerful. Of course, the functions each take parameters, but we have ensured that they are intuitive and are well documented within the github repositry. Well, at least they should be. If they aren’t, create an Issue on Github or something and we can try to sort it out.

As well as being externally modular, EACOF is internally modular. The separate tasks are self-contained, allowing implementations to be swapped out for something more suitable for a given task.

The two key parts are the Network section and the Storage section. The Network code allows communication between components, Providers, the Central Authority, and Consumers. The current implementation uses something with sockets, allowing communication between different machines. If, however, the sub-millisecond latency there was unacceptable, it could be swapped out for something faster, such as direct single-machine IPC. Similarly, the system keeping track of the internal state uses a SQLite database. This is almost certainly suboptimal, so can be swapped out for something different. Since the wrappers for both these things are pretty solid, it should just be a case of swapping single files out and having everything work again. Fingers crossed.

So basically, EACOF is a powerful modular energy measurement, written in C and designed for Linux and OS X, that improves over time, providing a simple API and is available under the BSD 2 clause license. This sounds pretty good to me, so here are a few ideas and examples of where to use it.

As I said, use cases. This’ll first be a couple of examples of where the energy consumption of software is being measured at present, then some examples of where you could use EACOF.

Web browsers are a clear case for trying to reduce energy consumption on desktop. This is because they’re the sort of things that’s opened, then left open for long periods of time, so any increased energy consumption will be a perceptible decrease in battery life. Thankfully, browser vendors are clued in to this.

First lets, look at the browser we love to hate, Internet Explorer. In this instance, Microsoft was on the ball with their browser. One of the aims of IE9 was to lead the industry in power requirements, with hardware acceleration and other changes improving performance. Thanks to Microsoft being the home of Windows, they had the advantage of being able to use the same highly instrumented hardware that was used during the development of Windows 7. Looking at the figures in a March 2011 blog post, it looks like they achieved this ‘lead the industry’ goal, reducing the overall energy consumption of IE below that of other browsers and the time.

You can do this with EACOF. Take EACOF and a Provider designed to monitor the current battery capacity. Monitor the base consumption of the system without your application running. Run things again, but this time with the task you’re measuring being performed. Subtract one from the other, and tada, you now know, pretty well, how much energy the application under test was using.

While the IE team had a general ‘lower ALL the power’ goal, all the way back in 2011, other browser vendors are being more refined about it, at least publicly. Mozilla, on the desktop, is currently using energy monitoring for testing purposes. Since modern Intel CPUs have easy access to CPU energy data, their test machines have energy monitoring hardware built in. By taking readings from the CPU, it can be seen when energy usage spikes, allowing investigation into why this was happening. This can also be used to look for regressions between versions - if version 14 uses 10 joules to perform a task, but version 15 uses 12 joules, this can be noticed and hopefully reverted.

Testing for regressions and spikes in CPU energy consumption can be done with EACOF. Take a Provider based on the Intel Power Gadget API, run your program, and you know how much energy was being used. Because it’s pure CPU, and you’re hopefully testing on a fairly fresh OS install, the consumption can pretty much directly be allocated to your application. Simple.

The Chrome team is doing yet more different things and has recently stepped up their game with energy consumption - a Performance:Battery label was introduced to the issue tracker in early January, and the Speed and Infrastructure teams have been setting up machines with power monitors hooked up to them to help with the task. One thing happening here is testing the energy consumption of potential features and deciding whether to let them through based on how much energy is used. For example, voice recognition. Ordinarily, you’d only have voice recognition active when focused on a text input. If, however, you wanted to hotword, have actions happen based on a particular input that can happen at any time, ‘OK Google’ or something, this needs to be constantly running, and so it needs to be checked how much energy is used by performing the task.

As for deciding whether to turn features on by default or not, EACOF provides the data required to make an informed decision. Test the energy consumption with and without the feature, find the difference, and that’s what the feature was using. If it’s felt that was an acceptable level, the feature can be added, otherwise alternatives like keeping it disabled or performing further optimizations can be made. Without this data, a decision could be made, however, it would not be accurate as it would be based only on past statistics, and therefore may not match current usage.

Microsoft, Mozilla and Google are all doing things at a larger scale here, but they’re all tasks which can all be undertaken with EACOF. In particular, the Intel-CPU-based monitoring undertaken by Mozilla could be easily replicated by anyone in this room with EACOF and a similar CPU (Sandy Bridge or newer, I think - 2nd generation core).

Because of what it is, EACOF can also be used for other things.

EACOF allows a program to know about energy consumption at run time. This allows it to adapt based on current conditions. For example, when watching an online video you probably want to be able to finish what you’re watching before the battery runs out. An energy-aware video player may intelligently balance the energy consumption of various hardware components - network adapter for receiving data, CPU for decoding, GPU for displaying and upscaling images. By monitoring these values, the video player may adapt the quality of the video to ensure that it may be watched before the battery runs out.

While designed to work for energy, EACOF can be used for other tasks. The central system pretty much just moves data around. Therefore if you have one, or many, Providers of numbers that change over time and wish to get them to one or more Consumers that are interested, you can use EACOF. The two ends will need to know what the numbers mean, but once that’s sorted out, the communication can certainly be done.

So, that’s EACOF. It’s a modular Energy Aware COmputing Framework for Linux and OS X with a powerful, yet simple, API and is available on Github under a BSD 2-clause license.

Use it and let us know the cool things you use it for or find out because of it. Improve it if you’d like - we’re open to Pull Requests and all that. Do these things, and we should start seeing longer battery lives and more happy users!
Thank you for listening, we now open up to any questions you may have.


