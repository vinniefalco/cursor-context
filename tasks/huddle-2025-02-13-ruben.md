Transcript of huddle in #boost-beast2 on Feb 13 from 8:40 AM to 10:50 AM Pacific Time (US and Canada).

This transcript is auto-generated, so some information may be inaccurate. It won’t be surfaced in search results.
@Vinnie [18:44]: Very Very exciting times, huh? Lots of things happening.

@anarthal [18:51]: I, sorry, can you repeat?

@Vinnie [18:54]: I said very exciting times, lots of things happening.

@anarthal [18:56]: Yeah There is, yeah

@Vinnie [18:59]: Have you met Steve? I want you to meet Steve and Michael.

@anarthal [18:59]: Definitely Hi guys, nice to meet you.

@Steve Gerbino [19:04]: Hello, hey.

@Vinnie [19:06]: So, so this is Ruben, Ruben's been with us. How many years have you been with us now, Ruben?

@anarthal [19:10]: 4

@Vinnie [19:12]: Who Nice. Coming up on your 5 year anniversary.

@anarthal [19:15]: Something like that

@Vinnie [19:16]: I love it

@anarthal [19:17]: Huh Somewhere

@Vinnie [19:18]: I said, you're coming up on your five-year anniversary.

@anarthal [19:21]: I think I can.

@Vinnie [19:22]: How do you like, how do you like the job? How do you like the job?

@anarthal [19:25]: I think it's a 3 years and I think it's one of these days' anniversary. No, I'm, I'm good, you know.

@Vinnie [19:33]: Did, did you get interviewed by the documentarian team? Did they come to your house and like and interview you. They're gonna come, right?

@anarthal [19:34]: I. I think that's gonna happen in April and I'll, we're gonna arrange this, but yeah.

@Vinnie [19:43]: Wow. That's very exciting

@anarthal [19:46]: They spoke to me already.

@Vinnie [19:48]: So tell me, so all you need to do is just tell me about your experiences with how you ported the library. So just tell us like what you did. So you, you ported which library?

@anarthal [19:59]: I did, boost with this because I think that was easier to port. It has less API surface than my SQL. my SQL protocol is not the best thing to, it's not very enjoyable, but this is, it's better. So this is a very high-level library. This means that, in addition to parsing and generalizing messages, it offers a lot of functionality, so it means that it handles things like reconnections, like pings, like it automatically

@anarthal [20:29]: Pipelines requests when you do like multiple requests at at a single time. So it does like a lot of high-level things. So the fact that I have been able to put that is like a very good sign. So I wanted to say congrats. I know I've opened a lot of issues and, but, but like this is like very good news,

@Vinnie [20:51]: No, those issues, I love it. Thank you. Those issues are exactly what we need. Oh, I love, I love issues.

@anarthal [20:59]: Yeah, and, I'll probably, if I have the time, because at some point I need to do the, the The module thing because I'm giving a talk in like one month and I want to be like prepared and would like we've done all these in boost and, and they had a, but I want to port more tests, so maybe I can get some more issues for you guys. But my experience has been, yeah.

@Vinnie [21:24]: So let me ask Go on. No, but please

@anarthal [21:28]: Yeah, my, my experience has been good. the library was already written like a set of generators, so there wasn't like a lot of a scene composers because as you know, I think composing in Asia I've learned to try to avoid it as much as possible. So it wasn't exactly like beast, which has like a lot of intermingled IO with and everything was written like as a state machines, you know, I like the machines. So, the thing is that there were like a couple

@anarthal [21:58]: Of files that were pure as I and I reported this with the more or less equivalent primitive and things have just worked aside from the backside, I, I reported. I think, you've gotten cancellation better than Asayo, which is good. This constellation is in Nassayo is a pain. So there's been some code I've been able to remove from the library. Yes, because I put it into, to Porosio.

@Vinnie [22:28]: So you like to stop token better than the cancellation slots.

@anarthal [22:28]: And Yeah, much better because they got this stop token thing is that this top token is it's simpler. And it has state And the cancellations slot is much more complex because it can be any type, it, you've got it, it has like many complications and it doesn't have a state, which means that after a single A sync operation that you run, you need to check whether the, because the state is not stalled in the in the slot. It's stored in the composed operations. So after every single incorporation, you need to check the state in the

@Vinnie [22:42]: Yeah

@anarthal [23:11]: In the composed operation to see whether you were canceled or not, and as you can understand, it is super easy to just forget one. So you need like a lot of

@Vinnie [23:19]: Yes, definitely. So

@anarthal [23:21]: Every single time you run a synccorporation, you need to check and that's not the case here, which is really good.

@Vinnie [23:28]: Tell me about your API for redis. So that uses the ASIO's universal asynchronous model with the result of customization.

@anarthal [23:37]: Yeah, the, the old one use that and the thing I've written, it just poses like a copy task and go.

@Vinnie [23:46]: So now, do you still have the, does, does the port have the same APIs before, or do you, are you, are you co-routines only?

@anarthal [23:52]: Of course not. It's co-routines only like the thing is and this is one of the reasons why I was intentionally like pushing back on the copy thing. I couldn't have written this with just with copy, not a single operation because it, as I said, this is a high-level library and all the operations need at least timers.

@anarthal [24:16]: And that means that you can't write it with copy.

@Vinnie [24:20]: I see. OK, that's that's actually very interesting.

@anarthal [24:21]: Yeah. And the thing now, what's, what's interesting is that I, I Kind of, like I own my SQL and I'm like a commenttainer in, in Veddi. I kind of controlled code in both libraries. I couldn't have written my SQ, the my SQL library with Kay only, and there is this is actually interesting for, for you guys to know. my SQL has like two layers. It's like two really two libraries at once if you, if you want. There is like a connection layer which implements the basic algorithms which

@anarthal [24:58]: Is what copy should connect screen because my English is all broken and I have

@Vinnie [25:03]: Sure, I would love it I, I think I know, I think I know where you're going with this, by the way. I think what I, so if I would just take a guess, I think what you're trying to say is that there is the protocol layer that's expressed in terms of composing buffers of strings, sending and receiving those strings, and that also can be co-routine driven because you need to block sometimes, and then you have the other logic which has to do with timers, cancellation, making connections, reconnecting, detecting dead connections. Is that correct?

@anarthal [25:34]: This is the theory, but there is an exception in the protocol layer, and it's a very important exception which makes me not be able to use copying my SQL for every, for every algorithm I would expect to use copy. but I need to share a screen to show this because my English is not good enough.

@Vinnie [25:49]: OK Yeah, yeah, show the screen, OK.

@anarthal [25:56]: OK, 2 Can you see my screen here?

@Vinnie [26:04]: No. Now I see it. OK

@anarthal [26:10]: OK.

@Vinnie [26:11]: It's taken a minute

@anarthal [26:12]: I

@Vinnie [26:13]: Or every slack has gotten worse ever since Salesforce took over.

@anarthal [26:18]: Which has gotten worse?

@Vinnie [26:18]: While I wait while I leave for this, while I wait for the screen to come up, Ruben, I want, oh there it is. OK, I see it now. Question, did you use any, did you use any AI to help you port?

@anarthal [26:26]: I I did, I didn't, like I, had, I especially used it for the tests. What I, what I did is I put it a fragments, then I told course are like, you see what I've done here? replicate it for all the tests in the in the file because I got the best of the two walls in, in here because I wanted to intentionally take time in some places because that made me think of

@Vinnie [26:50]: Nice

@anarthal [26:59]: OK, but why like this? And then for the more mechanical tasks I could just rely on AI and I have to say it's a pain checking like when it's like super monotonous, like checking the output. I just tend to say accept, accept, accept, and I know it's, that's not good and yeah.

@Vinnie [27:06]: So Yeah, so what I'm hearing is, is that what, what I'm hearing is you, you write one of the tests once by hand the way that you want it, and then you have the AI take that pattern and repeat it through the other examples test.

@anarthal [27:19]: Anyway Yeah, yeah. which I, I'm doing like one file at a time because I want to check and there are like every single test is a little bit different. so it's, there are some tests, well, that adds things like this is doing I don't know, TLS and maybe these needs are a little bit special care, but, yeah, that's the idea. Like if, if a file has like 10 tests. I, I write the first one and then I say like, you see what I've written here

@anarthal [27:59]: And just replicate and

@Vinnie [28:00]: Ruben, what I would like to do is after, after this conversation is done, I want to, I'm gonna bring in the transcript, and I want to make a report and it's gonna be written as if it came from you, that it talks about your experience pouring over to Carrozio and all this information and then I would like to give it to you to edit, and then maybe you could post it on the mailing list. What do you think about that?

@anarthal [28:01]: I.

@anarthal [28:21]: OK, yeah. Yeah, that's, that's good. OK. So there are a bunch of operations. You can see my screen, right? This is the myocular connection. There's, there's a bunch of, this is like the the lowest layer, the protocol layer. There are a bunch of operations that the protocol defines. So for example, execute, which sends a query and returns the result.

@Vinnie [28:22]: OK Yes. Oh

@anarthal [28:44]: There's a lot of things here, but our first statement.

@Vinnie [28:45]: This, this all looks like we call this all looks like boilerplate.

@anarthal [28:49]: Yeah, but what I want to say here is like there are a bunch of of operations and this is like a request a particular frame using it to the server and you the result back. This is in theory, what Kay is designed for, right? So you can express, you could express this, all these as

@anarthal [29:12]: As in terms of copy streams, right?

@Vinnie [29:12]: I I, well, you mean the implementation?

@anarthal [29:21]: Yeah, you could implement. This is now Aayo, but you could implement all these operations in terms of copy we strings and

@Vinnie [29:27]: Well, I don't know. I, I haven't seen, I don't know, I haven't seen the code. You need to show me code that deals with buffers. I don't see any buffers here. There's no buffers here. Where's the buffers? What are the buffers, Reuben.

@anarthal [29:39]: Let's say prepare. My corso isn't faster.

@Vinnie [29:53]: It's a big library

@anarthal [29:56]: Yeah

@Vinnie [30:01]: Sanzao, I like that. What's in there? Oh.

@anarthal [30:04]: All, all the network algorithms.

@Vinnie [30:05]: Oh there's execute. Let's the execute, that must have some buffers, no? Read results set, that's not happen. Oh, rebuffer, they rebuffer. There's a rebuffer, that's got.

@anarthal [30:09]: Execute is not execute is not the. Red buffer is a buffer, yeah, but anyway, I think the best one is the prepared statement, because that may be the one, the simplest one.

@Vinnie [30:20]: OK, it's right there.

@anarthal [30:24]: So this is completely

@Vinnie [30:24]: All right

@anarthal [30:26]: This is completely, this is expressed in terms of this Fox routines, but this is completely portable to

@Vinnie [30:31]: Oh, so you're using, OK, let the record show Steve, this is fauxx routines. This is coke routines implemented with macros and it's actually very fucking clever. Chris is very smart, and this is absolutely beautiful because you get like you get like 90% of the syntactical benefits of co-routines without any language support. It's genius. I mean, look, you could yield. Look at that. It's, it's wonderful.

@anarthal [30:56]: OK. So this is like the simplest case. This can be implemented in terms of of cuppy streams because you are just reading reading data. These are some wrappers around my, my internal buffer types because the macular protocol is not and has this awful sequence number, but don't pay attention to that. The thing is that you're just reading a couple, you are at some point, writing.

@Vinnie [31:07]: Sure OK

@Vinnie [31:23]: From writing. I like this. This looks good.

@anarthal [31:25]: Then you're reading. So this is completely expressible. now, there is a very hard algorithm here. This is like by far the biggest algorithm which is handshake. Handshake is a chaotic. It's a pure entropy, but it's very important because this is the first thing you need to do with my SQL server,

@anarthal [31:47]: Before doing anything else

@Vinnie [31:47]: All right, let's see it. Don't keep me in suspense.

@anarthal [31:51]: So this is, this is handshake, right? And as you can see, this is mad, but at the end of the day,

@Vinnie [31:57]: Yeah, but none of this, none of this is, I don't see there, there's the buffers. OK, there's the buffers. Now we're talking.

@anarthal [32:01]: OK, there is a problem which is here. This is the problem. This is a problem because you are reading, writing data, and in the middle of it you are calling us as a handshake.

@Vinnie [32:12]: But that's not a problem

@anarthal [32:14]: Why not?

@Vinnie [32:16]: Cause that can be abstracted. You can make that a virtual method, and then, then you have an implementation that that does like what it needs to do. Remember co routine, a co routine can be a virtual function. You can return a task from a from a pure virtual. So what you do is you split it, you have your, you have your base class implements this. The derived class provides the handshake operation.

@Vinnie [32:38]: The task, let me explain, let me, let me.

@anarthal [32:40]: Yeah, I mean, I, yeah, no, no, no, no, I know what you mean. I know what you mean because I've read your SSL and SSL streaming, yeah, I know what you mean. Yeah, I could do that.

@Vinnie [32:50]: I wanna explain something. I wanna explain something for the transcript. OK. The co-routine frame is the most powerful form of type erasure that exists in the language because inside of a co routine when you write coa wait, everything, the whole awaitable becomes type erased. Whatever goes in the watable is type erased from the perspective of the of the caller that launched your task. So you, when you get a task, a capy task, the only

@Vinnie [33:20]: The only thing that it exposes is the return value. All of the implementation is hidden inside the co routine frame, so you can, you can have an abstract. Base. You can have a pure virtual member function that returns a task and it completely type erases the operation. It's very powerful and it's a different way of thinking

@anarthal [33:43]: Yeah, I could like, thing is that I think the SSL stream right now lives in, in Croatia. So I, I would need to find something for myself. Like, yeah.

@Vinnie [34:00]: Right. So like I said, so you split your class You have a, you have your base class has the logic and it has a pure virtual for the handshake, and then this line 347 that you're implementing right now as you say coa weight and then you call your virtual function. And then in your, in your code to create, you know, the object, you create the derived class, and then that that derived class owns the stream.

@Vinnie [34:25]: The base class only sees the type Easing wrapper. The derived class has the has the actual object. This is, this is implemented in Beast 2, so you can look at an example of how that works.

@anarthal [34:38]: Yeah, I could do that I could do that, yeah And yeah, so this is like the protocol logic that could be implemented in terms of And what I was looking into is the good thing about this sort of fox routines is that they are very easy to test, and you can place these anywhere.

@anarthal [35:04]: What I mean is that we have, we have something very similar in ineddis. I can show Maybe The writer FSM maybe.

@anarthal [35:22]: OK, so This is the, the, this is what we have in this today.

@Vinnie [35:29]: Oh wow

@anarthal [35:29]: Right? So it has a similar things. The thing is,

@Vinnie [35:30]: OK By the way, I would, I just, can I just say something? I don't like what I'm seeing here. I don't like how, how you have to put the resume point and then the the the step number in the the parameter list, to me that pollutes the parameter list, but I mean I guess that's your choice, but I don't like that.

@anarthal [35:35]: Yeah. I, creases's choice was to not make that explicit, so he relied at some point on the counter macro and that created a lot of problems. So this is why I

@Vinnie [36:00]: Yeah, but wait, come on, hold on, hold on a second. Can you do me a favor? Can you, can you delete press delete.

@anarthal [36:03]: Yeah.

@Vinnie [36:08]: Now press, no, yeah, press double underscore line. Underscore

@anarthal [36:13]: Sorry, what

@Vinnie [36:14]: Type in double underscore The capital word line Double underscore

@anarthal [36:23]: Yeah, I could, I could do this. Yeah.

@Vinnie [36:25]: And you could put that in the macro.

@anarthal [36:27]: I could do this Yeah, I probably

@Vinnie [36:31]: I love it. This is I love this was Chris's idea. It's beautiful. It's beau it's absolutely beautiful.

@anarthal [36:31]: What's the reason why? Why did Greece use counter on Windows then?

@Vinnie [36:42]: I think because it's, it's optimized when you make a switch statement, if the numbers are sequential, then it produces better code.

@anarthal [36:52]: OK, yeah, yeah, fair.

@Vinnie [36:54]: Yeah

@anarthal [36:55]: So the thing is that this is in an internal implementation file like I don't care. For me the important thing is that if I repeat labels, the Compare this is an error because this is a switch case internally so if I cook up, it's gonna, it's gonna catch it. So, but anyway, the thing is, we had this written like this and actually in the current Korosio port, I haven't touched this at all.

@Vinnie [37:01]: I mean, you know, Of course

@anarthal [37:23]: So what I'm doing is in Connection That is

@Vinnie [37:32]: And also why are, wait a minute, so, OK, yeah, you put the code in the in the CPP file

@anarthal [37:32]: Well, as you can see Yeah. Yes, well, it's currently not a CPP file. What Marcello has a

@Vinnie [37:46]: But it could be you could now that now that you're returning, now that you're returning a task, you could put it in the TU

@anarthal [37:47]: As I could be completely be. So the thing is that we currently have this IPP files and we force the user to include a SRC.HPP into exactly one translation unit. So to all effects this is a translation unit.

@Vinnie [38:07]: I got you. Yeah, yeah, yeah. OK.

@anarthal [38:08]: So this is, this is, this is a position unity. So,

@Vinnie [38:10]: I don't want to, by the way, I don't, I don't like that anymore. I know I'm, I know I started it. I copied Chris, but I don't like that anymore. I don't do that. I don't make my libraries that way.

@anarthal [38:20]: The, one of the things I've told my fellows that if we fork the library, we can't get rid of this and we can do a like a regular CPPs.

@Vinnie [38:30]: Yeah

@anarthal [38:31]: But the, the important point.

@Vinnie [38:32]: This looks this looks really good, but this looks amazing.

@anarthal [38:36]: The important point here is that I've had to modify just very, very few things, and if you see I'm just creating the writer FSM, which is the code we've just written here, then I'm just calling it and and invoking the Coosio primitive that I need to to do this, to, so if

@Vinnie [38:57]: I love it

@anarthal [38:59]: If I need to write, I just go whatever I you think I need and, and the good thing about this is that I, with the same code, I can actually supportao and Corrosio and anything as long as it provides the right primitives.

@Vinnie [39:17]: I have a question

@anarthal [39:18]: Yeah.

@Vinnie [39:18]: What, how difficult, what, what did you encounter when you had to translate the ASIO buffer sequences to the corrozio buffer sequences.

@anarthal [39:30]: I, we really don't, so we just have a very, very simple buffering strategy here. So we just use a single big, contiguous buffer. So really none because there is not much to translate.

@Vinnie [39:48]: OK

@anarthal [39:52]: So yeah, I'm, I'm same thing in I Think there is some optimizations in here in Redis because I, I haven't had the time to look into that. but in my SQL what we do is we do a clever thing of the MySki protocol has like a fixed size header. So we realize everything into the buffer and we leave space for the header and then everything is contiguous and this is actually faster than using awafa sequence. These are things contiguous. It's just simpler for the color, for the colonel to hand.

@Vinnie [40:19]: Yeah. Very good, very good. OK. What do, what, what did you what did you encounter in terms of performance?

@anarthal [40:31]: I haven't measured that at all, but I could do that because I think Marcelo has a benchmark. So I could, I could have a look, but I, I guess it will be more or less equivalent. What, something I found with with this kind of database code is that

@anarthal [40:51]: Perform this is actually this is not the same kind of performance, Like you don't, you can't have the same mind as when you are dealing with high frequency trading, for example. Here, the most important thing is making the user super easy to do as, as less round trips, round trips to the server as possible.

@anarthal [41:14]: So, things like, this will just run a pipeline for you and you don't need to go like the, you, you need to reduce the number of times that that you go to the server because that's the expensive thing. You know what I mean

@Vinnie [41:28]: Yeah. That makes sense. You want to do as much work as you can per server, Paul.

@anarthal [41:32]: Yeah Yeah, something I found very, very good, like very easy to use, is the when any and when all and this is like Super important to have like.

@Vinnie [41:43]: OK

@anarthal [41:44]: This is critical and in Asayo they are, they cause a lot of trouble because to begin with the experimental, there are, I don't know if that's the case anymore, but last time I looked into them, they were not compatible with a one of the defiant Chris Hutt, like the old the old TS executors and then you've got libraries that require that macro so you can't really use these things. So having this and having them reliably, that's like super, super

@anarthal [42:14]: Important. And if you look at this, I've implemented this Kamella and cancel after these are improvable. I've

@Vinnie [42:15]: Go you What the hell is this

@anarthal [42:26]: This is, I actually opened an issue for a feature request. you know the cancel after and cancel out completion tokens in Asayo. You need to know all this.

@Vinnie [42:36]: OK

@anarthal [42:37]: This may be like the most useful addition Chris did in the last 5 years.

@Vinnie [42:41]: So you're, OK, this is very important, so we need cancel after is what I'm hearing.

@anarthal [42:47]: We don't, I mean, I've worked it around, because, it's really easy to implement.

@Vinnie [42:53]: No, no, no, no, no, it's not cancel after it sounds like a good thing. And by the way, you shouldn't have to, you don't need make error code there, at least you shouldn't.

@anarthal [43:02]: I probably can clean up this a lot. Like I've just tweaked it up to make it work, but, yeah, the thing is.

@Vinnie [43:10]: I'm just, I I'm saying, all I'm saying is, is that If the return type is the return type has an error code in it, so you can just return the enum and the implicit constructor of error code and will know.

@anarthal [43:22]: Yeah, it should

@Vinnie [43:23]: Yeah.

@anarthal [43:25]: Be like Yeah.

@Vinnie [43:27]: OK, so, all right, so let's talk, I wanna talk about this. So, so this is like a utility that you made. This is for cancel that and we, we need, we need cancel that clearly. Also

@anarthal [43:33]: Yeah

@Vinnie [43:37]: I noticed that you're doing your, your hard coding to task. I just want to point out that The design of When any

@anarthal [43:46]: Yeah

@Vinnie [43:47]: Is that they

@anarthal [43:47]: I know what you are gonna say and I just hard coded it like this because that was the quick thing, but this is what I mean by this is improvable because these should accept any kind of a waitable.

@Vinnie [43:55]: Absolutely Yeah, yeah, yeah I just want to say and I just want to say it for the transcript. The Kay defines a weightable concepts, so you're not tied to the task, so algorithms like cancel after, they can accept a heterogeneous or a homogeneous range of awaitable task types of a way of Iowa latables actually. They don't even need to be tasks.

@anarthal [44:02]: And this

@anarthal [44:21]: So the idea is that this thing I've got highlighted in blue, which is a wrapper coutine function I've had to create to make, to produce, Kapi Ayo task here. This shouldn't be required. It was just for me it was easier to not think and do this, the workaround and implement it properly.

@Vinnie [44:42]: Wait, what do you mean what, what shouldn't be required?

@anarthal [44:45]: That shouldn't be required. It is required right now because my utility is shit and it requires a an IO task instead of arbitrary or waitables.

@Vinnie [44:55]: Oh, but no, you're showing me, go back, what are you showing? What do you say, how can we improve this code that you have highlighted?

@anarthal [45:02]: This should be Like this

@Vinnie [45:09]: What did you delete

@anarthal [45:09]: So If we deleted a lambda that was wrapping this resolve.resolve. We now this is not an IO task. This is not a copy task anymore. This is a resolve a waitable and my utility function doesn't handle this.

@Vinnie [45:26]: Oh, I see what you're saying. OK, I'm sorry. Yeah, yeah, yeah, yeah, yeah, I got you, I got you. You had to create a wrapper to create a task.

@anarthal [45:34]: Yeah So this

@Vinnie [45:36]: OK.

@anarthal [45:37]: I know that this can be done. I just didn't find the time to do it, but in the final product like what you guys write this obviously should be handled correctly instead of this sheet I've written. It was just to try.

@Vinnie [45:53]: So we need to cancel, we need to.

@anarthal [45:54]: But yeah, cancel that and cancel after are very used, like are very useful. So, You could have a look at the signatures that Chris has in in Isaiah and I've kind of copied them here.

@Vinnie [46:12]: OK

@anarthal [46:13]: That's

@Vinnie [46:14]: This is very good stuff

@anarthal [46:17]: Yeah, and the other thing I wanted to, say I'm there was one of the reasons I was arguing with you about, well, this is the, the, the socket opens up the socket closed. We, we talked about the day that isn't handling the IT.

@Vinnie [46:32]: We have, we have a little, we have a little work, we have a little work to do defining how sockets are opened and closed. I realized that, yeah.

@anarthal [46:39]: OK, and well, one thing that we are, we are not that the original already supports and we are not supporting at the minute, and I know that for our users it's important is Unique sockets, which I've

@Vinnie [46:52]: Yeah, yeah, yeah, that's gonna be a, that's gonna be a separate type.

@anarthal [46:52]: Been aish about So I, wanted to highlight that I, I, I know that you probably know this, but there are like a many socket types that you can't just have classes for them all. Like you probably, it just makes sense like having TCP UDP, and, and Unixs probably, but there are like fos of protocols. So maybe having this kind of generic so sockets. I know that you don't like the idea, but I

@Vinnie [47:15]: Yeah.

@anarthal [47:26]: Don't think it's that bad of an idea at the moment,

@Vinnie [47:30]: OK, so let me, let me stop let me stop you there. Let me stop you there. So I want to write for the, I want to create for the transcript. So what Ruben, what Ruben is saying is this idea that we in Corozio, we should refactor the socket so that the reed and the right operations are in a base class that can be reused, so that means that like the TCP socket, we break that up. We put the read in the right, which is, which just works on a file descriptor. We move that into

@anarthal [47:30]: There is also, yeah. Yeah.

@Vinnie [48:00]: A different class so that we can subclass it and reuse the reading and writing machinery.

@anarthal [48:07]: Actually, I would like to also see connect machinery in this generic socket because you can have, for example, a Bluetooth socket. That's a thing. And you probably don't want to have like a Bluetooth socket in Corrosio, because it probably has like one user somewhere.

@anarthal [48:25]: So you probably don't, but you probably want to have the machinery to create this kind of sockets and manage this kind of sockets, you know what I mean?

@Vinnie [48:33]: The problem, but I do, but the problem is, is that If it's portable, if it's portable, then it needs to be a concrete type, and if it's non-portable, then we, then it would, it would only be available if you use an escape to get the platform handle.

@anarthal [48:50]: Yeah I know what you mean, yeah,

@Vinnie [48:54]: And we can, we can only implement the, we, we can only implement things when we have the use case, like I don't, I think it's a design smell to try to like implement for the general case without having the example

@anarthal [49:00]: Yeah.

@Vinnie [49:08]: So we can do, we're gonna do the Unix socket, by the way. We're definitely going to do it. I don't know that we're going to do it for the review. we're gonna have UDP, of course. I like the idea of refactoring the socket in the read and write to be in a separate class. I think that makes a lot of sense.

@anarthal [49:08]: Yeah, I, I Yeah

@Vinnie [49:24]: And that'll make it easier for us to provide different socket types, but I want to push back on the connect. The connect part has to be different. We, the generic connecting doesn't work because there's no endpoint object that can hold everything.

@anarthal [49:24]: Yeah, but I

@anarthal [49:41]: I would have a look at what Isayo does because they have a generic socket which are generic endpoints that handles everything. I don't know what's caused that.

@Vinnie [49:47]: Yeah, and they're connect and they're connected. They're connected diff they're they have a different connect. When you connect to a Unix socket, you don't provide an IP address.

@anarthal [49:57]: Yeah, what I mean is that Asayo has literally something called generic sockets with a generic endpoint. You know this

@Vinnie [50:06]: Yeah, yeah, but then it's up to you and then it's up, no, I get it, but, but in, in Karozio what you would do is you would derive from this class that I'm talking about that has the reading and writing.

@anarthal [50:07]: Because I do use that in my SQL, yeah.

@Vinnie [50:16]: And then you would put your stuff in there because the generic socket cannot be connected without writing some non-portable code. How do you connect a generic socket in ASIO.

@anarthal [50:25]: Yeah, yeah, I, I know what you mean. I actually use it to mean either a TCP socket or a Or a unique socket, so I don't have to do it because now if I want to support here, you know, I'm highlighting for the transcript. I'm highlighting a member in this red stream, which is an implementation detail of connection, which is a TCP socket. So if I now want to support Unique socket. I need to write stood variants.

@Vinnie [50:58]: No, no, no, no, no, no, no, no, no. That's not how you do it. Sorry, no, no, what you do is you use the

@anarthal [50:59]: Of How do I do it then

@Vinnie [51:05]: We just go take them, get the variant out of there. What you do is you pass a corro, put a new line, make a new line. Corozio Io_stream Ampersand. That's your generic, that's your generic socket.

@anarthal [51:21]: How do I? How do I connect this

@Vinnie [51:25]: Well, you need the derived, you have to have a separate derived class.

@anarthal [51:31]: Then I

@Vinnie [51:31]: You have to, you have to, well, here's what you do is you template, read a stream on the stream type, and then you refactor all of the code that doesn't care about the connection type. You refactor it to use the IO stream ampersend. And you put that in the bass class. Then the, so the derived class encapsulates the difference between the sockets while the base class has everything that's the same because let's be honest, the amount of code that's the same between Unix socket, Bluetooth socket, generic socket is enormous. The code that that is different is very small. So what does the code different have? It has construction. It has connection, and it has closing. You put that you separate that.

@anarthal [52:05]: This

@Vinnie [52:12]: And then you express the rest of your algorithm, like that stream, yeah, you see.

@anarthal [52:15]: Thing is that This stream only is, is about this, so it just has like a right sound which this can be simplified, you know, don't need the switch anymore, but you have this to make it a red stream and with wide stream and then you just have to connect. So this class is about this.

@Vinnie [52:24]: Yeah Yeah, so that goes in the derived class. You make that you make that you make.

@anarthal [52:39]: There is no base class then.

@Vinnie [52:41]: You make connect a virtual function in the base that's abstract, and then the derived class implements it and then you have a derived class for each type of socket. You have a TCP derived. You have a UDP or you have a Unix derived. Separate, you have the concrete object encapsulates the socket type, and then the the base class implements all the business logic and it dispatches to the derived class to do the things that are different, like, for example, closing the connection is different between SSL and a regular socket, so that becomes a pure virtual that returns a task.

@Vinnie [53:17]: You have to layer, you have to layer your, your, your code. And then you don't use them very. I hate the variant. The variant sucks because when you put a variant now you have a meeting point. When you have a variant, that one line of code is now ties together many different things. You have to see the SSL. You have to see the TCP. You have to see the units. What I prefer is a separate translation unit. I want to have the SSL derived class B and 1 CPP file. I want to have the TCP derived class B in another file. So this way if the user only uses one

@anarthal [53:21]: Yeah

@Vinnie [53:47]: Type, they only link to that code. If you use the variant, now you're linking in all that bullshit. You don't give the user the option to only use one.

@anarthal [53:56]: That doesn't make sense because the whether to use one string type or the other is a one-time thing. And this is how what we just like it.

@Vinnie [54:03]: Of, of course I totally understand, I totally understand that. And, but even, but even that can be made different, right? Like, for example, OK, let me give you the perfect example. So we have the IO context and we have a constructor where you tell it the number of threads, and we want to have an optimization where if you tell it that you're using only one thread, then we're going to instantiate a template of the scheduler that doesn't have a mutex. Now if the user uses that constructor, then they're going to link in both of the implementations. They're going

@Vinnie [54:33]: To link in the single threaded. They're going to link in the multi-threaded. However, what we're going to do is we're going to have another constructor that has a tag, single threaded tag, and then that constructor only instantiates the one reactor that has no mutex. That's how we avoid linking everything in, so you could do something similar. Now, of course I understand that like with Redis and MySQL you're usually going to need a, you're probably going to need SSL. Maybe, maybe not. But if they're not having SSL, then you can give them a way to not link it.

@anarthal [54:59]: Yeah

@Vinnie [55:03]: And you can do it cleanly without macros, without special headers.

@anarthal [55:06]: This This is my, I haven't done this yet, but this is what I would like or I think this is like If you're gonna Netflix.

@Vinnie [55:17]: Oh my God, I hate it already. You wanna have a factory factory to.

@anarthal [55:21]: Yeah, I, I hate every time you say virtual and I don't say anything, so.

@Vinnie [55:26]: Well, you have to, you have to accept the virtual.

@anarthal [55:30]: No, I don't have to

@Vinnie [55:31]: You got to accept it. If you want, if you want the benefit, if you want, if you wanna go to the, if you want to go to the next level.

@anarthal [55:33]: But Yeah, there's a roo, tealestre. Should have this

@Vinnie [55:44]: Oh yeah, OK, yeah. No, OK, yeah, I see what you're saying. No, I get it. We're gonna, so we're gonna, we're gonna, we're gonna give you that function. So, Ruben is saying is that what what Ruben wants is the ability he would like an interface, a factory to produce an a TLS stream, but the, the which stream is chosen becomes a runtime decision that's abstracted from the library.

@anarthal [56:10]: What I want is people not using TLS, not requiring OSSL because this has been a common complaints from our users. This is a real thing. I don't care about linking unique sockets in or out because I don't think this has a real relevance, but this is important.

@Vinnie [56:21]: They So they want WolfSSL

@anarthal [56:31]: No, they won't if you're doing, there are many people doing plain text, like doing plain text. And this is because if you're running your thing in a trusted network like a Docker container. this is much faster, much, much faster, and people

@Vinnie [56:37]: Oh, I see what you're saying. OK, so what, so what Ruben is saying, what Ruben is saying is, is that if the user wants to use only regular TCP connections with redis or MySQL. They should not have to link SSL, open SSL in with the library.

@anarthal [57:01]: Yeah, this is very simple.

@Vinnie [57:01]: That's a different, that's a, that's a different thing than what I was saying. That, that, by the way, that's very different. It's somewhat related, but, but I think that speaks more to your code Doesn't it?

@anarthal [57:11]: Yeah, completely. This is, I mean, this can be solved with this. Trivially.

@Vinnie [57:18]: You have to do that on, you have to do that on your end.

@anarthal [57:22]: Yeah, this, this line I've written, the factory thing I've written, this is, this would completely eliminate the, the Completely eliminates the dependence on OASSL. So, so this is another thing I want to say is that I am happy with how TLS is written in, in Corrosio.

@anarthal [57:44]: Because doing this in Asao is nearly impossible because of the templates. By doing this in Corrosio is as easy as doing this function which you can, you may like or not, but this works and is relatively clean.

@Vinnie [58:00]: But I don't, I don't agree with, I don't agree with this. I think I think, well, OK, the, the function's fine, OK, to be fair, the function's fine. But what I'm saying is, is that I think you should restructure your class, like your your object that does the thing. I think it needs to be layered. I think you need to have the base class has to carry the stream, sorry, the derived class carries the concrete stream type, and then the you have a base class which implements everything in terms of that, you know, in terms of the IO stream part. So that means

@Vinnie [58:34]: The derived class handles connection, handles the handshake, and it handles the shutdown. And then the base path handles all the business like.

@anarthal [58:37]: But Really, This is This is sort of how it's implemented already because what you've been, I mean, it's not exactly this object oriented as you've told it, but it's separate, it's layered because this only takes care of reading, writing, and connection establishment and close down connect here with we'll clean up if there's anything left. but

@Vinnie [59:04]: But wait, sorry, go back up to line 68. So I feel like those two, like line 69 and line 70. You have a TCP socket, you have an open SSL stream. I believe that those should go into a two different derived classes.

@anarthal [59:21]: Thing is that

@Vinnie [59:22]: And then, and then, and then the user, when the user wants to decide if they want OpenSSL or not, they choose which class to instantiate.

@anarthal [59:32]: These OK, and how does do I connect the thing and then.

@Vinnie [59:41]: That happens in the derived class. The derived class does it.

@anarthal [59:46]: Yeah, but the user will have to pass something in. And if you have a pass a TCP stream and then they pass in a Unix address.

@Vinnie [59:55]: So why does it wait, why does the user, I don't, why did, no, no, that, but it happens in the derived class. So the TC, so the derived class that does the TCP socket, it has a function that takes the the IP address in port. Same with the OpenSSL, but that one will also take a TLS context. And then for your Unix derived class, you have one that accepts a path. So in other words, all of the differences is encapsulated in the derived class while all the similarities are pushed into the base.

@anarthal [1:00:09]: But Yeah, I know what you mean. I think I know what you mean.

@Vinnie [1:00:28]: And, and then, and then the user's decision of whether to use OpenSSL or not is here I'm going to type, can you, can you switch to the chat and, and can you switch to the B2 channel? Just switch to the Beast 2 channel

@anarthal [1:00:39]: OK, yeah

@Vinnie [1:00:41]: So then what what happened is the user will write. They'll write the SSL what reddish stream. Or TCP reiss. Game. That's what the user will write

@Vinnie [1:00:59]: Go, that's, you see what I did there? Like the user decides

@anarthal [1:01:02]: Yeah Now, if the user wants to switch between this dynamically they have to write much more code than because this this is increasing API surface because this SSL already streamed, isn't public.

@Vinnie [1:01:16]: But no, wait, that's not true, that's not. No, wait, what?

@anarthal [1:01:21]: The red corrosiore stream I've written there. This is not public. This is an implementation detail.

@Vinnie [1:01:29]: Well

@anarthal [1:01:31]: The only public thing, I, one thing I like about Redd is that the public API is super small. The only thing they configure is whether they have this should be a sort of a variant but whether a TCP address or UniSocket address, and then the library does creates the objects internally as required.

@Vinnie [1:01:47]: I I, I can't, I can't speak, yeah, I do understand that. I can't speak to your, to your code specifically if you're telling me that this is an implementation detail, and this is an implementation detail. However, I'm still going to say that from the user from the user perspective, I think it's best to express

@anarthal [1:01:50]: You know what I mean

@Vinnie [1:02:05]: To express the choice of stream in the type system, the way that I wrote it, the way that I showed you in that channel. And then also, let me observe, if you have an SSL stream, There's a socket inside, so any SSL stream you can design the code to make it allow it to also be a plane stream just by bypassing the handshake and then passing a reference to the internal socket up to the base class, right? Every SSL stream carries a regular socket which you can get to. So you get, you get the, you get the dual interface naturally

@anarthal [1:02:34]: Yeah.

@Vinnie [1:02:43]: So if a user asks for an SSL stream, they can choose at runtime to use it in plain mode if they want to.

@anarthal [1:02:50]: Yeah, the thing is that With this, you get to choose in runtime whether to use SSL and whether to use TCP or Unix socket. The thing is that it is very common in database databases that you will get like a connection Configuration in a sort of a you were, you were out and this kind of things are completely dynamic. So

@Vinnie [1:03:12]: OK, I'm gonna repeat, I'm gonna repeat myself. You can still do that. OK, let's go back to the chat.

@anarthal [1:03:17]: I, I know, I know, but yeah.

@Vinnie [1:03:19]: Let's go back to the chat in B2, right?

@anarthal [1:03:22]: Yeah.

@Vinnie [1:03:24]: OK So now I've created a new stream called the Multiredi stream, or let's just say connection, and that one has all three of the things, right? Again, you encapsulate in the derived class what the the concrete objects and that abstracts the differences for the base class, so you get what you want, but the user decides if the user wants the ability to switch between TCP, SSL, and Unix domain sockets at runtime. They choose the multi-stream if they want only TCP, they choose the

@Vinnie [1:03:56]: TCP stream. If they want SSL only, they choose the SSL stream. You see how that works? They you everything that, everything that you want is possible, but you structure the code in a way that lets the user decide which capabilities are there.

@anarthal [1:04:02]: Yeah, I think. I mean, the advantage I see with this is that this, yeah, kind of solves the, the don't link to OSSL problem.

@Vinnie [1:04:20]: And, and I want to stress, no variant, please, no variant. I don't like the variant. I don't like what you did. Go scroll down a little bit, go to line 90. I don't really like that, the switching on the type of.

@anarthal [1:04:33]: No, this This is completely improvable. You don't

@Vinnie [1:04:38]: Yeah.

@anarthal [1:04:39]: Need it

@Vinnie [1:04:39]: But I mean, I understand this is like this is the natural way to implement it. Like, you know, this is the obvious way. This is what comes if you were to think about the problem and and for the first time and you want to implement multiple transports, the most obvious way to do it is to add an enum, and then deal with it at each point where there's an, there's a difference, like in the right sum, you know, use your like that. That's the natural solution, but it's unfortunately

@anarthal [1:05:03]: This is inherited from Masao in Masayo they have different types and what what I think write some returns is different. So the thing is that here my idea was just because you can't just convert any of these to a copy read stream and you could just, I mean this is, this is improvable so you don't need to look at this.

@Vinnie [1:05:21]: Right Yeah OK

@anarthal [1:05:27]: The thing is, yeah, I see what you mean and I'm gonna have a thought, have a thought about this, in my SQL is having a little is a little bit more complex because the protocol itself requires some cryptography operations, so it's not just

@Vinnie [1:05:27]: Yeah

@Vinnie [1:05:45]: Oh

@anarthal [1:05:46]: Yeah, you know, so if we go to handshake, for example, and, handshake handles, different kind of

@Vinnie [1:05:46]: Interesting Yeah

@anarthal [1:05:57]: Different kind of Of authentication methods, and there is one that just requires It requires encrypting something with with, with a, with an imme an asymmetric key that the server gives to you. So,

@Vinnie [1:06:16]: Yeah

@anarthal [1:06:19]: This is much more difficult to abstract away. I mean, I'm sure it can be done. Probably for my SQLV2 version. But yeah

@Vinnie [1:06:30]: OK

@anarthal [1:06:31]: So this is, this is one of the reasons why I started with Reddy is because it's much simpler. The protocol is much simpler.

@Vinnie [1:06:36]: So I think, yeah, so looking at this code base, it definitely, this can be reorganized in ways that will make it easier to maintain. The asynchronous parts will be easier to easier to understand with co routines. You can abstract the difference between like the different types of sockets in a way that minimizes the amount of code that has to be different for the sockets, and it will make invisible the fact that the derived classes can make, can have different transports. This is the, this is, by the way, this

@anarthal [1:07:04]: OK.

@Vinnie [1:07:07]: Is the fundamental feature that the T type berraor of code routines naturally provides, and this is why Cappy by itself without Carrozzio is such an important library is because it allows you to efficiently type erase the away the differences in buffered algorithms, right? Like, Cappy is what allows you to have very performance implementation that doesn't care if it's on an SSL stream, and it does it without templates where ASIO, you have

@Vinnie [1:07:37]: Long compiled times, you have the problems with the linking. You have templates everywhere and you can't hide any of the implementation. You have no ABI stability. With Cappy, all of that goes away. Everything is in the TU. You get polymorphism at runtime with no templates.

@anarthal [1:07:53]: I want to stress out something here which is this writer and the reader objects. This is what actually has the business logic. so, this stream, this is the object that had a variant you hated. So the thing is that these objects in a way, but it obstructs away what transport are you using. So, so this business logic doesn't care about the, so this, this obstruction is already there.

@Vinnie [1:08:21]: Yes

@anarthal [1:08:21]: I don't have a switch here in the business logic. I have a dedicated class.

@Vinnie [1:08:24]: No, but it will be made, it will be made at it Will be made more efficient because you won't be going through the layer, the function that does the switch.

@anarthal [1:08:32]: You would be going through the virtual code instead. I don't think this is really a, a different at all. I mean, I don't think this is, is really having an impact, real impact. It can, it can be better like better implementation, but that's an implementation detail. I think it's much more relevant, the fact that if we can get rid of the OASSL linkage through the method you said like this is, this is what's important.

@Vinnie [1:08:57]: Yeah That's fair

@anarthal [1:08:59]: Now Yeah, no, I think, do you have anything else we want to speak because I, I had like one kind of crazy proposal, which I

@Vinnie [1:09:09]: Sure, tell me, let me know.

@anarthal [1:09:11]: Yeah. So, one thing that I'm, I'll be like writing this copy code. I'm a bit sad about is, I think I already kind of mentioned this, but in testing. So, let's say for example, prepare a statement, right?

@Vinnie [1:09:31]: Yeah

@anarthal [1:09:31]: So,

@Vinnie [1:09:32]: Oh, by the way, one other, OK, one other thing. So, there's another, there's another, by the way, there's another advantage. You you don't want the variant and you don't want the switch, trust me. Here's the biggest advantage of separating the code the way that I suggested. You can create a new derived class for testing that uses a CAE or that uses, yeah, that uses a capy mock stream, and that stream has a very special feature you can inject errors and ex

@anarthal [1:09:33]: We have this Yeah

@Vinnie [1:10:02]: Cept ions into your code.

@anarthal [1:10:03]: Yeah, I know about this. I know about the fuse. the thing is that

@Vinnie [1:10:07]: Yeah, use the fuse.

@anarthal [1:10:10]: I know about the fees. the thing is that, to test most of this code.

@Vinnie [1:10:11]: The fuse.

@anarthal [1:10:15]: Which does much more than just reading and writing. I don't think this

@Vinnie [1:10:19]: You need connections and you need time, you need connections and timeouts. I get it.

@anarthal [1:10:24]: Yeah, you need, I had an idea which is separate to what I'm gonna say next, which was I think it would be very good if you could express your tests in terms, your, your in terms of mocks and expectations like you do in GMOC, you know JimMOC?

@Vinnie [1:10:42]: No, I don't know GMO

@anarthal [1:10:43]: Google Temo

@Vinnie [1:10:44]: I don't fuck with GMOC.

@anarthal [1:10:46]: OK, so in essence you define a mock object which marks your functions and you say, I'm expect this function to be called this times and with these arguments, the idea, I had is like something similar to that, but then this expectations are in terms of core routines. So you say it's like a

@Vinnie [1:11:09]: Yeah

@anarthal [1:11:12]: I probably need to write some code to show this because the thing is that here.

@Vinnie [1:11:17]: No, I hear you

@anarthal [1:11:19]: To properly test this kind of thing. I, I need mock timers. I need to mock the timers. I probably need, at some point I will probably need to mock my own objects because you get too high in the level of obstruction if I'm testing something that depends on, for example, in MySQL,

@anarthal [1:11:41]: Who Info

@Vinnie [1:11:45]: So So, so let me, let me, let me respond to that. I think What, what I do is I go I go as far as I can in terms of the testability and like injecting the errors and having the mock objects, but I don't go so far that it becomes a burden on my implementation and making timers mockable is for me, a one step a little too far.

@Vinnie [1:12:14]: Because the the amount, the amount of

@anarthal [1:12:14]: You

@Vinnie [1:12:17]: The amount of change you need to make to be able to like express your algorithm in terms of like generic timers. It's a lot

@anarthal [1:12:25]: That's why I did the state machines because I don't test this kind of code directly. I don't test this kind of code directly because smoking timer is, I did in my school. This is, it's nuts. so I test the state machine and then I write integration test with the actual server that cover this. So the integration test will will cover the main cases and then the unit test will cover the smaller cases, the, the, the more the edge cases.

@Vinnie [1:12:45]: Yeah

@anarthal [1:12:58]: What I mean about mocking my own objects is, for example, this is the implementation of a connection pull in my SQL and at some point I need this is my square connection and you see how this is running in terms of my SQL functions, a synect, a syn ping, and a synchron pipeline.

@anarthal [1:13:19]: To be able to test this efficiently either I just test the state machine alone, or I need to mock my own functions because I don't need to implement all the protocols, all the.

@Vinnie [1:13:27]: Yeah I pay dude, that's a pain in the ass.

@anarthal [1:13:34]: I know, I know. So I've

@Vinnie [1:13:35]: That's a pain in the ass

@anarthal [1:13:37]: This is why I like this state machine functions because they are very easy to test. And So,

@Vinnie [1:13:44]: What I would do is, what I would do is I would let me push back on that is I would create a mock MySQL server. It's what I do, what I do in Beast, I make a websocket server. I make an HTTP server and I test the client and then I do the reverse. I know it's different.

@anarthal [1:13:53]: But then

@Vinnie [1:14:03]: It's easier with we

@anarthal [1:14:03]: I don't think that is good testing practice. You've heard about the testing pyramids. Which should be unit tests, many at the bottom, and I think this is going to up in the pyramid to test two low things.

@Vinnie [1:14:18]: I know, I know a book, man. I know the theory. Listen, look, I know the theory, I understand unit tests versus integration tests, but at the end of the day we need to be practical like, you know, there, there's an art to writing the tests. You got to know when to break the rules.

@anarthal [1:14:19]: It Smell And I, I think this state machine thing. This was this is like a very practical way of I mean, of, of writing things. What I'm what I'm sad is that this kind of

@Vinnie [1:14:47]: I agree

@anarthal [1:14:53]: This kind of state machines are written in in macros and this is not very practical. There are many things that you can't do with that. So,

@Vinnie [1:15:00]: No You know what though, hey, I, I have an idea. Hey, Steve, I've got a great idea. Why don't, why don't we create IO timer. An abstract timer, we move all the functions into there, and we have the impel will be clean with the virtual function, and then in the derived class for the timer for the for the concrete timer, we can derive from the impol and that's where you can put your bullshit. You can put your, your variables there, and then other someone else, now someone can create a timer that works with our system just by subclassing the the io timer part. That would be a very clean solution and that would solve the problem for Ruben.

@Vinnie [1:15:40]: Ruben could create a mock timer by deriving from IO timer, and it would, it would be perfect. He just has to refactor his code to use the IO timer reference instead of a concrete timer.

@anarthal [1:15:53]: This is, this is quite cool. if you're gonna do

@Vinnie [1:15:56]: Let me show you, let me, can you do, can you go to the channel? Can you go to the B2 channel? And I'm gonna

@anarthal [1:16:01]: Yeah, I'm, I have the best channel open in my other screen. You're not seeing it, but I'm seeing it.

@Vinnie [1:16:06]: OK, well, I mean it would be OK, wouldn't best for them to see it. But And then we have This would be very good

@Vinnie [1:16:29]: Boom, there we go. Steve, look at the chat. This is perfect

@anarthal [1:16:39]: Now, if you're, if you're gonna do this like there is, you know, just. You you guys know jest

@Vinnie [1:16:47]: Wait, one more thing So that's what your code would use, that's what your code would use. You would have an Io timer reference, so you would slice it, you slice the timer and you use the reference and then in your derived class where you because the pattern, this is the pattern, Ruben. You put the derived class as the socket. It has the timer, and it has your logic for connection and all that. Well, now you have a derived class for the test where you have your mock timer, you pass the Io timer reference in the constructor for the base, and now your algorithm is expressed in terms of Io timer refer

@Vinnie [1:17:21]: Ence

@anarthal [1:17:22]: You know me and I would probably write a function taken an Iotimer ampersand and because I'm not very fond of this extreme objected, but that's at the end of the day the same thing. Yeah, I know what you mean.

@Vinnie [1:17:34]: It's a very good way to strengthen. I hear, I hear what you're saying, but it's a really good way to structure the code. It's very natural. It's a good use of the the type system and the inheritance that way.

@anarthal [1:17:46]: Now if you're gonna do this, you know, just. You guys know just

@Vinnie [1:17:50]: No, what the hell is just? Is that that stupid hassle?

@anarthal [1:17:52]: So just Just is a testing framework for JavaScript, and they have a very, no. With E.

@Vinnie [1:18:02]: Oh, just, OK

@anarthal [1:18:05]: They, they, I think they do, pretty well with mocking. They, it's very enjoyable and, and in particular, you can you can, well, it's the thing it's very dynamic, right, but you could have the equivalent of mock timer service where you can use your timers regularly and then you just say, OK,

@anarthal [1:18:30]: My time is now, advanced the time to this time point and then your timers, the timers that are scheduled to fire, they will fire. So you have deterministic testing timers. You know what I mean?

@Vinnie [1:18:42]: Yes That sounds very good, but I mean, I don't know how much of that framework we're going to include, but in theory, you know what, in theory, Corozio could offer mock timers, so that's very much in scope. So my philosophy is when you, when you, when you provide a framework to the user, you should also provide the tools for testing the code that they write. We see that in beasts beast has the extras. It's got the test stream, right? And if you look in Cappy right now, we have all of like those text test fixtures. So I think Corozio Ma

@anarthal [1:19:03]: Completely

@Vinnie [1:19:14]: Ck timers is very much in scope.

@anarthal [1:19:16]: OK. So, Something that I like is writing this kind of of something I like about this code is that every time it it needs to read. It will yield, it will exceed the function, and I can intercept that in the test. So this is trivial to tests. You don't need a mocket. You don't need a fuse. It's very natural if you

@Vinnie [1:19:33]: Yes. Fair

@anarthal [1:19:43]: It's Well, I think they're even better, the ones in eddy. I've been improving with time. 91.

@Vinnie [1:19:51]: Huh

@anarthal [1:19:53]: Despite a

@Vinnie [1:19:54]: I like it

@anarthal [1:19:56]: So this is This is very easy to test. You create FSM. Then you they say Everson resume. And you say, OK, it wants to write. Then you check the state of the system. Then you say, OK, the rights completed successfully.

@anarthal [1:20:14]: It now we want to, it wants to wait. This tests are very easy to write. You want to To simulate an error. Cheer for error

@anarthal [1:20:30]: No need to use even ause nothing, You can just say

@Vinnie [1:20:35]: Oh Sure Yeah, you're testing your

@anarthal [1:20:38]: It was canceled.

@Vinnie [1:20:40]: You're testing your state machine directly, I get it.

@anarthal [1:20:42]: So this is, this is super easy. So something I

@Vinnie [1:20:45]: I don't, yeah, I don't like that.

@anarthal [1:20:48]: I know you don't like it, but I kind of do. So, some

@Vinnie [1:20:52]: I would prefer my, my preference is to test Through some type of stream.

@anarthal [1:21:00]: I, that's like I I think it's much easier to because for example, we diffuse we've got locks and the locks here are important. Because people, yeah, people like locks.

@Vinnie [1:21:13]: Logging

@anarthal [1:21:16]: People like having, and we test that.

@Vinnie [1:21:16]: Hm Look, look, to me, look, look, to be fair, to be fair, the code that I write is different than the code that you write. My code is sockets. My code is like streaming IO buffers. Your code is different. There's complicated business logic, so it wouldn't surprise me that your testing strategy has to be different. There's no single testing strategy that's universally appropriate for every domain, and what you're describing is you're telling us what testing strategy you've discovered through trial and error works best for you.

@anarthal [1:21:47]: Yeah, so, and the point I wanted to make here, and you are free to say me like go fakov Ruben is that this handmade generators are shit with C++20 core routines, I'm pretty sure we can do much, much better. So if Kapi could have something that would help me write this kind of code, but without being this shit, it would be really helpful for me.

@anarthal [1:22:13]: You know what I mean

@Vinnie [1:22:14]: But that's just C++ 20 co routines. How is that? What can we offer that's above C++20 co routines.

@anarthal [1:22:20]: I, I like, some kind of generator, I guess, class. I don't like C++23 generator.

@Vinnie [1:22:25]: 00 yeah, OK, you mean like a generator. That's easy. I thought C++ already had a generator.

@anarthal [1:22:31]: There is C++23 generator, but that's shit. I don't want this. Like, this is just

@Vinnie [1:22:36]: We'll just opening it, that's a different thing. Just open an issue.

@anarthal [1:22:40]: So what I was trying to figure out is When you I'm, let's see here.

@Vinnie [1:22:47]: You can have a generator opening. I'll, I'll, I'll produce a generator in 3 minutes from one prompt.

@anarthal [1:22:48]: All right. So what I, the, the thing I had in, in mind and this may be Was Let me see because I've written some test code which of course doesn't work and doesn't work, but I think that was Kay, no, that was HTTP what's the post of PASA?

@Vinnie [1:23:14]: Some cafe. The Who?

@anarthal [1:23:18]: What's the pose of parser in HDP, the

@Vinnie [1:23:22]: Perializer

@anarthal [1:23:23]: Serializer. serializer is right enough. That was part of it. To boost HDP pass, yeah. So this is your reed, the reed algorithm that

@Vinnie [1:23:38]: Nice

@anarthal [1:23:39]: You know, that's the Asian you wrote, And no, this is the retailer, the retailer algorithm you wrote, and this is Very similar algorithm, but instead of using a socket directly, it uses the generator sheet I've written which doesn't really work, but this is the kind of code I had like in mind and I would love singing in Kapi because they

@anarthal [1:24:06]: It's not as straightforward as just having a socket. Having a socket is just more straightforward but.

@Vinnie [1:24:11]: Listen, I please enough, don't, you don't have to overexplain it. Just open an issue with what you want, and I have to tell you something. The function that you're looking at was generated by AI.

@anarthal [1:24:21]: OK, and the good

@Vinnie [1:24:22]: Wait a minute, wait, whoa, whoa, whoa, whoa, what the hell? Wait a minute. Whoa, whoa, whoa, whoa, whoa. What's this Cay generator, you made that.

@anarthal [1:24:23]: Yeah This is I've, I've written this.

@Vinnie [1:24:30]: Oh!

@anarthal [1:24:30]: This is the kind of bullshit I was, this is what you wrote.

@Vinnie [1:24:33]: What the hell is this? What's this?

@anarthal [1:24:36]: I wrote this. I, this is not, this is not ready. This is not because it has compossibility problems right now. You can't rate it like this, but I would love seeing this kind of code because this is really Sansao. You could, this is like Sansayo and sans how.

@Vinnie [1:24:47]: No I got bad news for you. I have very bad news for you, Ruben. This type of code, this kind of protocol level parsing of RFC like HTTP WebSocket frames, mime, multi-part form, chunked encoding. These types of grammars.

@anarthal [1:24:59]: What was

@Vinnie [1:25:14]: They are, especially if you want them streaming, you, you must create a hand rolled state machine if you want the best performance, period. Peter for the last like week has been doing experiments. He wrote a Jason serializer. So Jason, imagine serializing Jason using a co routine, right? You switch on the type and then you coawe and then you write and then if you have an object, you go, you recursively coa away into it a serializer for Jason is very easy to express with co routines.

@Vinnie [1:25:44]: However, what we're seeing is that you cannot get the same level of performance that you get if you do a hand rolled state machine that uses buffers, and what you're looking at in the HTTP parser, that's a hand rolled steak machine that takes buffers of input and it like and it it does what it needs to do. So I hear what you're saying, but like, but that and of course we're gonna, all you have to do is open an issue, we'll give you your generator. You can have it, but you need to know that there's a trade-off, there's a cost.

@anarthal [1:26:02]: OK, but Yeah, so, what I want to make clear here is that I haven't rewritten the parts and the prepare and the committee of methods of the of the passer. These are state machines as they are. I've just expressed the thing that you had in terms of a coin of IN I task as a generator. That's the only thing I've done. It's just.

@Vinnie [1:26:40]: But why did you just, what, what are you, what do you generating

@anarthal [1:26:44]: I'm generating, so each time you yield you yield a, a buffer pair. A buffer to read things in and every time this resumes, this, you get the error code and the number of bytes that the operation has has produced. So the only thing I've done here is taken.

@Vinnie [1:27:05]: You're telling, wait, wait, wait, wait, wait, so you're telling me that when you you cowe this, it gives you the buffer that you write into.

@anarthal [1:27:14]: When you call it because you don't really cowait it, but yeah, I, what I've just tried to do is, write the same.

@Vinnie [1:27:21]: OK, let me stop let me stop you there. This is already implemented, and I use, there's a whole concept in Cay for exactly this. It's called the buffer right sink. It's that's the pattern. This is a 0, this is a zero copy pattern, and it's already implemented and I think, do you have the latest version of HTTP?

@anarthal [1:27:33]: But Yeah

@Vinnie [1:27:42]: It's there. If you already have it, look for a sync for. Sync_4 Yeah No, sorry, let me

@anarthal [1:27:53]: No, because this should be sourced for.

@Vinnie [1:27:56]: No, yeah, source 4, there it is. Return a source Well, that, that's for the body. I didn't do the header. The header, the header is Handled. Actually, I did do the header. I did do the header. Sourcescore.

@anarthal [1:28:07]: OK. The thing is that this depends the the only challenge I'm doing here is that this involves with the stream. And the generator thing I was proposing doesn't involve any red stream because you are yielding

@Vinnie [1:28:23]: Wait, hold on, hold on. Hold on, hold on, hold on. This doesn't look right. This doesn't look right. You don't have the la, you don't have the latest HTTP.

@anarthal [1:28:29]: What doesn't look right I pulled it like 3 hours ago.

@Vinnie [1:28:37]: Because there's a buffer right sink. Oh yeah, I did it. No, I did it. I did the buffer right sink. Let me find it. Sorry, parser, right? Parser. Concept buffer, no concept, read stream, right sync, oh.

@Vinnie [1:28:53]: I never did it for, I didn't do it for the, OK, I'm sorry. I only did the serializer, so that's on my to do list is to implement the buffer sync. Can you open up, can you open up the documentation for Cappy?

@anarthal [1:29:03]: But You Oh, you mean the web-based?

@Vinnie [1:29:11]: Yeah. You know where it is. OK, under concepts Keep going, scroll down, scroll down, scroll down, scroll down.

@anarthal [1:29:25]: I think something's going wrong here, right?

@Vinnie [1:29:29]: In the left, on the, on the left in the table of contents. Scroll down, down, down, down, stop. What? Wait, no. Design, no, no, no. Keep going. Here, buffer sink

@Vinnie [1:29:51]: So this is the zero copy This is the buffer sink Get ridable buffers from the sink's internal storage. This is what you, this is what you're doing is you're But this is a so Cappy has an opinion on this.

@anarthal [1:30:06]: So,

@Vinnie [1:30:06]: Buffer

@anarthal [1:30:08]: Yeah

@Vinnie [1:30:09]: Look at that sentence, buffer sync inverts the oko. Look at the move the cursor up, buffer sync inverts the ownership. The sink exposes its internal storage, and the caller fills it in place. That's what you're doing. You've implemented buffer sync, but the janky way, you've implemented the ghetto buffer sink.

@anarthal [1:30:26]: But the thing is that the difference is that this doesn't, the, the, the generator, yeah, the buffering guy I wrote doesn't really care about the stream in any, in any way. Like it doesn't touch a stream.

@anarthal [1:30:44]: For me this is a good thing because you are even more separating concerns even more. Now you have like

@Vinnie [1:30:50]: Well, you don't, the wait, the buffer sink doesn't care about the stream either. Go back to the concept So the parts are, the parser will return this object, a buffer sink, and then you call prepare To get the buffers. See that? You get ridable buffers

@anarthal [1:31:07]: Yes.

@Vinnie [1:31:08]: And then you cowe you and then you write your data, and then you co away or commit. And that does the actual writing into the parser.

@anarthal [1:31:19]: OK, but the, the writing happens as part of the algorithm.

@Vinnie [1:31:24]: No, you do but you do it, no, you do it yourself. You call prepare and then you buffer copy the data into the the buffers that it gives you, and then you call commit. This is what you wrote. You wrote this code.

@anarthal [1:31:24]: You don't like So, hang on, hang on, can you show me an example of this?

@Vinnie [1:31:43]: Sure. I think

@anarthal [1:31:45]: Or where I can see.

@Vinnie [1:31:48]: There

@anarthal [1:31:49]: OK

@Vinnie [1:31:50]: Zero copy transfer. So we call prepare

@anarthal [1:31:56]: We could prepare, yeah

@Vinnie [1:31:58]: Yeah And then we buffer copy the data.

@anarthal [1:32:02]: So where does the socket read or socket write happen? And does it happen in commit?

@Vinnie [1:32:03]: And then we It happens in the call to commit.

@anarthal [1:32:10]: OK, so the difference is that it happens as part of the field sink algorithm. So if you want to test that you need to have somehow a fuse or a mock or whatever and in the thing I wrote, which I'm not saying it's better or it's just different. I like it more, but you

@anarthal [1:32:30]: This doesn't actually write the data. It it returns you the buffer and you are the user is will, will need to perform the writing themselves. This is just completely

@Vinnie [1:32:39]: That's exactly what the, that's exactly the code that you just saw. Go back to the doc. That's exactly what it is right there, buffer copy.

@anarthal [1:32:47]: No, but you do go commit and you told me commit writes the data over the network.

@Vinnie [1:32:52]: No, no, no, no, wait, there's, OK, stop, stop, stop, stop, stop. There's a confusion here. There's a confusion here because there's two data, there's two streams.

@anarthal [1:32:52]: Has to be.

@Vinnie [1:33:00]: There's the sink, which is where the data goes, and then there's the data that you have, which is what you provide. In this function, the data that is being provided is that string view. You see that string view?

@anarthal [1:33:12]: Yeah

@Vinnie [1:33:12]: That's the string view. So this

@anarthal [1:33:13]: But at the end of the day, the sink does right the data to the network.

@Vinnie [1:33:18]: Yes

@anarthal [1:33:19]: But you can see here I'm not writing any data anywhere. I'm just returning, the,

@Vinnie [1:33:26]: Yes you are

@anarthal [1:33:28]: No, I'm not. Where do you see me writing?

@Vinnie [1:33:29]: Yes, lying, no, stop. Li line 546, you're writing the data.

@anarthal [1:33:34]: 5:46. No, I am No.

@Vinnie [1:33:38]: You're transferring the data into the parser, that's doing parsing.

@anarthal [1:33:42]: Yeah, I'm doing pausing, but I'm not, I mean in this case, this is a, this, I should be reading data from the network, right, because it's it's retailer. I'm not reading any data from the, from the network. here I yield from the function and they use the, the, the color will have to issue a stream that reads them and then resume the generator with this.

@Vinnie [1:34:03]: Yeah, exactly, sorry, my bad. I'm sorry, go back to the code. Go back to the documentation. I, I misspoke So, OK, let's, let's have an understanding here. The sink In this example is the parser. The output of the parser is an HTDP message which in memory, which you can access as an object. You can look at the fields, you have the body. OK, so that's what the sink is. So the buffer sink is the capy API for zero copy transfer. This is the API where an where an endpoint that wants to receive data can give you its buffers.

@anarthal [1:34:22]: OK.

@Vinnie [1:34:42]: The parser has an internal buffer, and the buffer sync API is Cappy's opinion on how the parser should deliver those buffers to the caller in order for them to be filled. That is what this is and that is what you wrote. You wrote that.

@Vinnie [1:34:59]: So in here when we call sync.commit Down there, right where your cursor is. You see, you see where, no, yeah, down. You see that sync.commit?

@anarthal [1:35:05]: Yeah.

@Vinnie [1:35:08]: That writes that delivers the data to the parser.

@anarthal [1:35:08]: Yeah. Yeah.

@Vinnie [1:35:14]: This is what, this is exactly what you wrote By the way, look, if you look at the code, bring the code side by side. It's you wrote the same thing. There it is. Look, see it?

@anarthal [1:35:31]: OK. So

@Vinnie [1:35:32]: You've got You Yeah, see, that's the same thing. It's called that look, commit GOF. Commit N You did the same, no, yeah, commito on there right down there. Look, commit EOF.

@anarthal [1:35:45]: OK But then why are these Why do I need to cowait this. Is it this doing any kind of network transfer or is it just because to to to support that, but Because there is a difference, right?

@Vinnie [1:36:01]: Yeah, just a point exactly, that's just exactly, that's to support that. The parser will have an awaitable that does all of the everything in the awaitable. There's no suspension taking place.

@anarthal [1:36:12]: OK, so this is really synchronous.

@Vinnie [1:36:15]: Depending on what you pass, like if you pass, if you pass a sink that's a socket, you're going to suspend. If you pass the parcer sink, you're going to get, you know, you're not even going to get a code routine frame. You're going to, it's going to be as if you were calling the parser directly. That's the value of the abstraction.

@Vinnie [1:36:33]: The await the parser's awaitables don't suspend. They do all of their activity in the awaitable. They don't create a co-routine frame. They're not the parser the waitables are not co-routines. You should look at, you should look at serializer instead of parser because I did, I implemented it there. Let's look at serializer. Serializer does the reverse. It gives you the buffers.

@anarthal [1:36:47]: OK, Yeah

@Vinnie [1:36:57]: Instead of, it doesn't fill your buffers, it gives you the buffers. It gives you direct access to its own buffers. People don't understand the power of this. Like, nobody understands this. Yeah, I don't even know if Steve really understands this, but this is, this is new. Nobody has seen this before. This is all new. This is where the power is. OK, we're in the serializer there, a sink.

@Vinnie [1:37:17]: That thing, scroll down. Boom. Serializer, sync. That is whatever. I don't know what kind, I get confused with the concept. I don't even understand my own code. What can I say? But this is again a buffer sink.

@Vinnie [1:37:36]: It gives you the buffers You fill them And when you commit,

@anarthal [1:37:40]: Oh, but you've got a stream here

@Vinnie [1:37:42]: Yes

@anarthal [1:37:44]: I don't have any stream

@Vinnie [1:37:45]: That's right What?

@anarthal [1:37:47]: I don't have any stream in my code.

@Vinnie [1:37:50]: No, that's, that's, sorry, no, that's a, that's the that's the serializer stream We have to serialize somewhere, where are you going to serialize to?

@anarthal [1:38:02]: You are going to yield it so the user, I mean, you're going to serialize it to your, to your buffer, you're right, because that's there is an internal buffer, right?

@Vinnie [1:38:09]: Is that what you want? Hold on OK, look for sync 4. Look for sync 4. Sync_4 No, you know, S I N K. OK, see this? OK, let me explain this function.

@Vinnie [1:38:29]: This function takes a stream And it returns a sink. Where when you write to the sink, it will serialize the data and the serialized data will go to the stream. So it's like a, it's like a pipe. It's a pipe on the in the input side of the pipe, you have your HTTP body that you want to render, you know, your, your Jason, your whatever you want, and then the output side goes to the socket. It could be a socket, whatever it is, whatever you want to put there, this creates a pipeline and the parts, the serializer is in the middle of it and the API for the serializer sync is a buffer sink. So if you want to ask me, Vinnie, what's the, what's the serializer's API for providing the

@Vinnie [1:39:14]: Body data, I'm going to say it's a buffer sink, and if you go to that page in the documentation, you will see what it is.

@anarthal [1:39:21]: Yeah, but, the, what I'm trying to tell you, I, I understand this, and I think this is actually a great idea.

@Vinnie [1:39:28]: Yes.

@anarthal [1:39:28]: What I'm trying to say is that When you do all of this, you've got a stream to write the data to because that's a pipeline.

@Vinnie [1:39:38]: Yeah. Yes

@anarthal [1:39:40]: OK, we agree on this. What I'm saying, what I'm, what I was trying to show with this code is that you don't have a stream here. The stream is owned elsewhere. You're separating the business logic from actually the reading or writing. You could, I mean, you probably don't want to do this, but you could write a sync synchronous function using synchronous SIV1 doing this.

@Vinnie [1:40:06]: And that's exactly what I'm going to do. I haven't done it yet, but I'm going to do the same exact thing to the parser of what I did to the serializer. The parser is going to provide what's called a buffer source. And what will happen, and that's, and that's a pipeline on, on the beginning of the buffer, on the beginning of the pipeline is your socket that goes into the parser and then the user has a buffer source where they can retrieve the data of the HTTP body, but they receive the buffers that are produced by the parser. Let's let's go back to the box.

@anarthal [1:40:17]: But you will need

@anarthal [1:40:36]: Yeah, I I, I completely understand what you're trying to tell me. but I think I'm doing like a very bad job explaining myself here, because the difference here, I'm not saying this is better than what you're doing. I'm just saying that this is different in the sense that this is not doing any kind of IO through any kind of stream. This is like agnostic to the to to any kind of string concept. this is leaving the IO to the user, to the color of this

@Vinnie [1:40:41]: Yeah.

@Vinnie [1:41:04]: Sure

@anarthal [1:41:08]: Beheader.

@Vinnie [1:41:08]: No, I, I get it

@anarthal [1:41:11]: And this is different

@Vinnie [1:41:11]: But I would argue, I would argue, I would argue that what's the point of that? That's not a

@anarthal [1:41:15]: The point of that is that, this would enable why I want want to do that is, because I would be able to use my testing strategy, which I think is not a point for you, but it would be for me. I could use this with any kind of this is a fundamentally very similar to the kind of state machines I've been written.

@Vinnie [1:41:39]: But I'm sorry, so I'm sorry, but what if, if all you're doing is putting data into the parser, then how does the test look at the output?

@anarthal [1:41:50]: The test can look at the output because in every time it reaches this code yield, you you get the control transferred to the testing code to what to the color code, which in this case is

@Vinnie [1:42:01]: OK, and then what does it do?

@anarthal [1:42:04]: It will look very similar to this.

@Vinnie [1:42:07]: No, no, don't, I don't, don't tell me how it's similar. What will it do? Will it look at the header? Does it look at the request or the response? Like, what does it do? This is HTTP. The result of parsing is the headers and a body. So where, where, what, what are you doing with it? You're feeding the parser, but where's the data? Where's the headers? Where's the body?

@anarthal [1:42:07]: And I

@Vinnie [1:42:25]: Show me where's the button? Where's the?

@anarthal [1:42:26]: You will in It will, you will you will place the data in this buffer.

@Vinnie [1:42:34]: OK, so you put the data in the buffer and now you've parsed it and now.

@anarthal [1:42:34]: The buffer You will use copy buffer

@Vinnie [1:42:38]: Wait, hold on, I get, I understand what you're saying. So you, you put the, you put this, you put the data in there, and now you've parsed it and then what?

@anarthal [1:42:47]: And I will call the function again saying that OK, we've reached successfully and I've read 20 bytes, and then for example, I can test very easily short reeds.

@Vinnie [1:42:56]: OK

@anarthal [1:43:00]: Or I can

@Vinnie [1:43:00]: No, but it's no, no, no, no. What do you do with the message?

@anarthal [1:43:02]: See.

@Vinnie [1:43:06]: Where's the message? Where's the request line? Where's the response line? Where's the body?

@anarthal [1:43:13]: The test will have the request line all the data, it will put it in this MBS and this is a way to test this function that the whetailer does would

@Vinnie [1:43:22]: OK, no, you're not hearing me. You're not, you're not listening The, the caller.

@anarthal [1:43:26]: I'm not understanding, I'm listening.

@Vinnie [1:43:28]: The caller is providing serialized data. They're providing the raw characters of the HTTP message, and they're putting it in the parser. But, and, but the parser parses. So when it's done parsing, it builds a request or a response object and then it provides the body. So for example, if your input is chunked in coding and the parser will parse that. It will give you the headers and then it will deliver the body with the chunking removed. So where's your code that does that?

@anarthal [1:44:00]: I just written this to the header, the code that does that is in the pars and the Godheader and these, I haven't touched this.

@Vinnie [1:44:09]: But those are my functions.

@anarthal [1:44:11]: Yeah, I know

@Vinnie [1:44:13]: OK, so when you're done, so after you parse the message, where's your code to inspect the parsed result.

@anarthal [1:44:13]: What the I haven't written any of these. I've just, my point was, you

@Vinnie [1:44:26]: All right, let me explain, let me explain something to you, since you haven't written it. So here's what's going to happen when you try to actually use this. You're going to feed the parser and you're going to give it an HTTB body and then you're going to say, OK, now what does that look like? And then you're going to want to have an API where you can retrieve that data, inspect the fields of the HTT message and then inspect the body. Well, the moment that you try to do that, you're going to realize, but wait a minute, the body is buried inside the parser. There's no real way for me to get it out, and you're going to

@Vinnie [1:44:56]: Say, Vinny, how do I get the data out of the parser? and I'm going to say, well, you know, you need to construct the right sink, and then you're going to say, where's that? I'm going to tell you, I'm going to say you need a right sync that puts the result in a string, and then you're going to use the function like what in the serializer that I showed you that's going to connect your buffered source in and then you're gonna go through the parser and it's going to go right to your sink, which is going to fill up a string and you're going to get back a string with the body. You're going to compare that and make sure in your test that the body that came out was the body that you expect. What you're showing me

@Vinnie [1:45:26]: Here is insufficient for the tests because it's only half of the code. You need the other half which receives the body, which gets the result. That's why you need a pipeline, the serialized data goes in and then the output is the structured data that's what you want to test.

@anarthal [1:45:41]: OK, positive has An API to get the body, right?

@Vinnie [1:45:45]: What

@anarthal [1:45:46]: The parser has what you are telling me is that the parser, well, I've just written the the part for the headers, right? So let's focus on this.

@Vinnie [1:45:54]: Yes

@anarthal [1:45:56]: The parser has a way of retrieving the past headers, right?

@Vinnie [1:45:56]: Yes Yes

@anarthal [1:46:03]: We'll have, I guess

@Vinnie [1:46:05]: If there's an object, you can get a, you get a structure, you get a container. There's a container that holds the headers, but there's no container, there's no container that holds the body because the body, it's a streaming parser. The body comes in pieces and it's very inconvenient for the test. You don't want to get pieces in the test. You want to get the whole thing. So you're going to come to me and you're going to say, Vinny, how do I get the whole body out of the parser, and I'm going to say, well, you need a right sink that accumulates the data and you're going to have to implement this concept, and I'll probably add it to Keppy as a convenience that that that like

@anarthal [1:46:09]: OK Then my test

@Vinnie [1:46:35]: A sink that collects everything into a string. And then you're gonna have the same function that that you're seeing in the in the documentation which where you have to, you pass the parser and then you pass the destination where you want the body to go to. Now you have a suspend point, maybe.

@anarthal [1:46:54]: OK, so I guess it depends on what kind of because I think you've got a function to retrieve the entire body if the entire body has been red, right?

@Vinnie [1:47:02]: No Yes, you can get, you can get to it, yeah, in that special case, you can probably get to it.

@anarthal [1:47:09]: OK. So it depends on what you are trying to pass, and what you're trying to test, sorry. if what I'm trying to test is that I don't if, if that

@Vinnie [1:47:20]: Well, what matters, what matters is the size of the parser's internal buffer. Which you have to provide on construction. If the buffer, if the parser's internal buffer is smaller than the body, then it's going to deliver it in chunks.

@anarthal [1:47:35]: OK, so what depending on what I'm gonna, what you are trying to test here. I guess you will provide one body, one kind of buffer like one size of like you, it just have control over it and they will be different depending on what you, the, the kind of statements you want to to test here, right?

@Vinnie [1:47:57]: I didn't understand anything of what you said.

@anarthal [1:47:57]: I mean Like, I may not want to get the entire message. If I just want to try HIF want to test this function and maybe I don't know, I haven't written the function. So I don't know what the test.

@Vinnie [1:48:16]: Well, OK, let me, let me help, let me, let me, let me help you. So does Cappy test streams allow you to restrict the amount of data that gets transacted in every IO. So if you want to make sure that your algorithm supports the other end of the pipeline only consuming a limited amount of data perio, you can easily do that.

@anarthal [1:48:42]: Yeah, I mean, it's just feels. I don't know. Like You know, I'm convinced. But yeah, I mean, it's

@Vinnie [1:48:51]: Wait, did you, wait, well, sorry, you said you're unconvinced or convinced?

@anarthal [1:48:52]: I Yeah, I'm, I'm convinced, or that about the testing, about the

@Vinnie [1:48:56]: Of what? What are you, what are, what, what are you unconvinced of?

@anarthal [1:49:01]: I'm, I'm convinced that the, that if I like for me it's like if you can't avoid Injecting a mock. That means that your code is probably better than if you have to inject a milk.

@Vinnie [1:49:16]: That's not true

@anarthal [1:49:16]: For me. But I guess that's for me

@Vinnie [1:49:19]: That's not true. So that, OK, so now we now we've identified the The design, the design skill gap. When whenever you that use words like always and never, that's like a design smell. You need to be careful. You gotta use the right tool for the job. In this case, because of the way that the parser is designed, in general, you're gonna have to use like some type of object that can accumulate the partial results. That's just going to be a fact of life.

@anarthal [1:49:29]: Well.

@Vinnie [1:49:44]: So the API that I'm I've developed at Cappy where you have the buffer sink and the buffer source, is that is the API and so there will be something on the other end which is capable of receiving all the body data and assembling it back into a string to make it convenient for testing.

@Vinnie [1:50:02]: Because the body is streaming, that's the the key insight here is that the HTTP body is streaming. It comes in pieces. And you want to be able, and you need that, you need to handle that in that you want your tests to be able to handle that.

@anarthal [1:50:10]: Yeah, I mean, the thing is that

@Vinnie [1:50:19]: So you have, you have to have, you have to have an algorithm that does that does that, that has the pipeline.

@anarthal [1:50:25]: If your test can control what's it's a level of what I feel is that with this kind of streambos, your test, a lot of times you really don't control what you are. I mean, you have like more limited input, more limited control to what you input to your tests than what I have in

@Vinnie [1:50:44]: But let me, let me Let me show you what your test is. Let me show you what your function's gonna look like. I'm writing in this channel

@anarthal [1:50:55]: What's the question, sorry

@Vinnie [1:50:56]: I'm, I'm posting in the B2 channel. This is probably what you're gonna have

@Vinnie [1:51:19]: See it. Like, you provide the data, you provide the bytes that we're going to be, you know, parsed. And then it will parse the whole thing and it'll give you the response header and then you get a string with the body, and that's the end of it.

@Vinnie [1:51:37]: Or an error code maybe, I don't know, whatever. None of the piping stuff will be visible. Why it doesn't need to be visible? Why would you make it visible? You don't want to deal with HD. You're not testing HTTP. You don't want to test HTTP. Do you?

@anarthal [1:51:52]: Yeah, that's, that's the point.

@Vinnie [1:51:54]: Yeah, you're not testing ATP. In fact, you even have a body? Do you have a body?

@anarthal [1:52:00]: I don't know what that means.

@Vinnie [1:52:03]: HTTP messages have two parts. They have the header and they have the bottom.

@anarthal [1:52:06]: Yeah, I know, but what doesn't mean do you have a body in which context?

@Vinnie [1:52:10]: In your tests, are you, do you have only the header or do you, are you also transmitting the body?

@anarthal [1:52:14]: I'm not testing HDP at all, like I'm just, I was, this was a kind of a proof of concept of the kind of thing I wanted and I wanted to do an HDP because I think you know it better than if I start speaking about Reddies. I think that could be like something we could it could be like better to speak.

@Vinnie [1:52:28]: Oh Oh, so you're, this is not even a real.

@anarthal [1:52:36]: I'm not writing it to be this, you know.

@Vinnie [1:52:37]: This is not even a real use case.

@anarthal [1:52:40]: My, my use case is this

@Vinnie [1:52:41]: So let me, let me tell you something. Let me explain something that it, in, in, in, in boost HTTP, the containers like the request of the response containers. They have a constructor that will construct from the message, you don't even need a parser. You can just construct the container directly from the message.

@anarthal [1:52:42]: What I want is That is not like what I told is what I showed isn't.

@Vinnie [1:53:00]: No, I understand what you're trying to show.

@anarthal [1:53:02]: Yeah, it isn't a real use case Mar my real use case is that I have this writer FSM which I may be able to find it here. This writer has said.

@Vinnie [1:53:12]: Well, what I'm what I'm saying is this, is that we have, we have the buffer story covered, like we, Cappy is a very agile with buffers. We have, we could read them, we could write them in multiple ways and Cappy is also very savvy with zero copy. If you want to go 0 copy, you can do it. Not only can you go zero copy, but you could go 0 copy with zero suspension. The concept that Buffy, that Cappy offers allows you to implement a buffer, sinks and sources that don

@Vinnie [1:53:42]: ' t suspend. They're completely synchronous. So if you want to write parsing code or if you want to write serialization code in the co routine style. You can do it and the other end of that can be an object that doesn't suspend. It's very efficient. That's all well tested. The concepts are very well defined. It's very well covered by tests. We have the type erasing streams. You can type erase any of those concepts easily, and they're very performant. They've been optimized. The story for what you're talking about has been written and it's good.

@anarthal [1:54:11]: It's, I completely get that. So my see if you want like my real use case, I want to be able to rewrite this, without this ugly macros like using real coins, and many of these things do things beyond just reading and writing. They do not.

@Vinnie [1:54:29]: So what's a commit what is a commit right? What's the MPX? Let's see, let's, let me, let me see your MPI.

@anarthal [1:54:34]: MPX is a is is something equivalent to the parser. It's a multiplexor

@Vinnie [1:54:42]: Well, then you would, what you would do is you would give it an API that that, that, that meets the, what that satisfies one of the.

@anarthal [1:54:47]: And this is This is what I

@Vinnie [1:54:49]: Captain Buffer concepts

@anarthal [1:54:51]: This is what I were, we kind of depart on, on points of view like I don't want to like for me providing this kind of synchronous and like What I want to do is I don't want to have to inject, start injecting mocks of timers and it's here because

@anarthal [1:55:13]: I have it like I want it to be better than what I have today.

@Vinnie [1:55:16]: OK

@anarthal [1:55:17]: And if I now have to inject mok timer. This is going, I, for me it's gonna be like worse than I'm gonna have today.

@Vinnie [1:55:22]: But I almost said anything about timers. I was talking about streams, and now you're talking about timers. Why did you change the subject?

@anarthal [1:55:29]: Because, Because the thing is that the approach I was showing demonstrates like that it doesn't need any kind of mocking to write the tests. And what you're saying is, OK, don't worry because we've got mock streams.

@Vinnie [1:55:46]: OK, when you said you want, but you

@anarthal [1:55:49]: And we may implement mok timers, and what I say is that why implement something if you can completely avoid it with another.

@Vinnie [1:55:58]: I'm not if you don't need it, look, if you don't need it then you don't need it. We don't need to belabor the point, but I'm looking at your code and what I'm doing is I'm looking at line 67 where you're doing a prepared right, and now I'm looking at line 76, where you're calling right some, and now I'm looking at line 79 where you're saying commit right, and to me that looks exactly like the capy buffer sink.

@Vinnie [1:56:18]: That is, you've, you've literally reimplemented the API of buffer sync.

@anarthal [1:56:25]: OK, so,

@Vinnie [1:56:27]: Like that you have that, you have, you have that pattern in your code already.

@anarthal [1:56:27]: If let's say

@Vinnie [1:56:32]: And Cay.

@anarthal [1:56:32]: Let's say I rewrite this, right

@Vinnie [1:56:35]: Cappy for Cappy formalizes the pattern that you've created. Like it formalizes that.

@anarthal [1:56:39]: Let's say I I end up accepting a, a, was it a sink?

@Vinnie [1:56:45]: Something like that. No, no, no, no, no, sorry, no, you're not accepting a sink. The MPX is exposing a sync interface. Whatever that MPX is, what the fuck's that MPX?

@anarthal [1:56:56]: Multiplex

@Vinnie [1:56:58]: No, show me the, show me the Declaration. Let's see the declaration. Let's look at these functions. Let's see what the shape of it is Prepare there, there, prepare right. OK, what does that do?

@Vinnie [1:57:17]: Return the number of calls requests, OK? And then commit right. So where's the functions that that transact in the buffers.

@anarthal [1:57:17]: OK

@Vinnie [1:57:25]: Where's the buffers I don't see the buffers

@anarthal [1:57:31]: So this gets to a Right buffer

@Vinnie [1:57:35]: Where's the buffer I don't see them

@anarthal [1:57:39]: Right

@Vinnie [1:57:40]: So don't, don't switch files on me, show me in the API where there are the buffers. Oh, there we go. OK, wait, how can you return a string view? That's not writeable.

@anarthal [1:57:47]: So I This is a right buffer, so you don't need to write to you, you just get the you get the

@Vinnie [1:57:58]: Oh

@anarthal [1:57:59]: The view and then you write it to the network.

@Vinnie [1:58:02]: You got the names backwards. OK. All right. It's a common, it's, I get it. OK, so the right, OK, so right buffer is data that's coming out, and the read buffer is data that's going in.

@anarthal [1:58:15]: Yes

@Vinnie [1:58:16]: OK, so this implements so.

@anarthal [1:58:16]: It's going right to the server, read from the server.

@Vinnie [1:58:19]: I get it. So this implements, this is you've exactly, this is exactly the pattern that I'm talking about. You, you know you have a pipeline here. You understand you have implemented the same exact thing. You have a pipeline. MPX is in the middle. Your right buffer is on the

@Vinnie [1:58:35]: It's on the, it's on the output and your read buffer is on the input. This you've exactly implemented the pattern

@anarthal [1:58:42]: You get that.

@Vinnie [1:58:42]: In this entire this entire thing can be expressed with one capy concept.

@anarthal [1:58:47]: You get that writer doesn't just manages the right buffer and doesn't it and doesn't touch the reed buffer, right?

@Vinnie [1:58:56]: Of course, I get it

@anarthal [1:58:59]: So let's say I wanted to use this Capy concept here.

@Vinnie [1:59:01]: You've been you've implemented, yeah, you've implemented the concept.

@anarthal [1:59:06]: Let's say I want to implement a convert this MPX and say it it implemented this would be a source. Sink.

@Vinnie [1:59:16]: Yeah. Something like that. I can't remember which is which

@anarthal [1:59:17]: Or Let's say I end up coweight. Copy Something

@Vinnie [1:59:25]: Well, yeah, OK.

@anarthal [1:59:26]: OK. I may replace all this by this copy something. OK, so after I've written the data to the server. I need to, get here. So here, this is a

@anarthal [1:59:42]: A timer that may, you may either wait for some time. Or get notified. So if the user asks us to write a request to the server. It, the timer gets gets canceled. If there is an

@Vinnie [1:59:56]: So you were the the this is the, this is a co8 point.

@anarthal [2:00:01]: Yeah, so I will need to cowait here again, right?

@Vinnie [2:00:04]: How is that, how is that a function though? Isn't that, shouldn't that be a member function? or is that a base class?

@anarthal [2:00:11]: This is a, this is an action type. This is a a very like

@Vinnie [2:00:18]: Is that a free function? How can that be a free function?

@anarthal [2:00:20]: No. Yeah, this is a class, right.

@Vinnie [2:00:23]: So it's a base class

@anarthal [2:00:24]: That. No, it is simpler than they said.

@Vinnie [2:00:26]: No, no, don't, don't switch out, don't switch out on me. How Go back

@anarthal [2:00:30]: Right direction. You told me right action write some.

@Vinnie [2:00:34]: Oh, it's ecstatic. How can it be static? I don't understand.

@anarthal [2:00:37]: Because it just returns what it has, it doesn't do it, it returns what it needs to do. It says you need to write some with this time out. It doesn't do it.

@Vinnie [2:00:43]: Oh

@anarthal [2:00:47]: That's a key point here.

@Vinnie [2:00:48]: Why didn't you just put that in the braces, what like why put off, why don't you just put an initialize oh I see. All right, OK, I got you I got you.

@anarthal [2:00:59]: Now, I can replace this by something like a weight, copy,

@Vinnie [2:01:04]: Yeah, Co wait No, no, there's no Cappy here.

@anarthal [2:01:09]: No

@Vinnie [2:01:09]: So that's your, that's your thing. You need to write that, no, you write that. That's yours. You have to make your awaitable. You have to write, you have to author and awaitable here. That's your job. Cappy doesn't do that. That's your business logic.

@anarthal [2:01:18]: So Look, look, look This is in connection info

@Vinnie [2:01:26]: This is getting to be a long conversation. We're gonna have to wrap it up so I can get the transcript.

@anarthal [2:01:30]: This What it does is this you see this cancel. This is what I was writing copy.

@Vinnie [2:01:39]: Oh, you're sending the data

@anarthal [2:01:39]: So. So,

@Vinnie [2:01:42]: You're sending the data, right, you're sending the data.

@anarthal [2:01:45]: Good

@Vinnie [2:01:46]: This is the pattern. This is the pattern that I was talking about.

@anarthal [2:01:49]: But I What I've, what, what is important here is that I have it separate. The sending of the data happens separate to the logic of I need to send this. So if I merge the two, I can merge the two, but if I merge the two, it's like an argument similar to the argument I made before about merging Kapi and Corrosio.

@Vinnie [2:01:59]: Of course, of course

@anarthal [2:02:10]: I don't want to merge it. I like them separate because they are two different things and they are good tested as two different things because then my tests don't need to take any, they don't need to think about the actual implementation of this. You know, my, you know what I did, the point I'm trying to make.

@Vinnie [2:02:28]: Yes, I see what you, you don't want to use the pipeline Well, you can implement it, you can imple if that's what you want, then you can make resume, take the, you can make the resume function take the output side. So that you can pass it in

@anarthal [2:02:42]: What So really, the request I had was like I don't know if this is in scope for for copy, but I would like this kind of functions to be written to be easier to write, like more enjoyable to write.

@Vinnie [2:03:00]: That is insult, you have a, you have, so you have, you're gonna have one side, 11 side is going to be a capy concept. And the other side is going to be whatever you what you want this thing with this, these functions that you call to get the data out so you could send it.

@anarthal [2:03:00]: Maybe it's Yeah, that, so I want the copy things to leave here in this connection.

@Vinnie [2:03:18]: Where's the, where's the right go back, sorry, where's the right happening? So stop. Let me see this. What is this thing?

@anarthal [2:03:22]: Here Right is here

@Vinnie [2:03:25]: Well, how did we get here? How did we get here?

@anarthal [2:03:28]: So this coin creates these FSM.

@Vinnie [2:03:30]: Sure

@anarthal [2:03:32]: Then it's called resume and they say, OK.

@Vinnie [2:03:35]: Just do me a favor, very, very slowly go back to that other code. How do you go back to that other code?

@anarthal [2:03:35]: I got an action

@Vinnie [2:03:42]: Very go very slowly cause I'm getting confused. So it helped me understand though But writer action weight just returns destruct.

@anarthal [2:03:51]: Yeah, it returns what is really like a Hans grown variant because I don't like ras.

@Vinnie [2:03:56]: But I don't understand, how can, how can returning a struck to do anything

@anarthal [2:04:01]: It doesn't do anything per se.

@Vinnie [2:04:02]: Or the power, so you're telling me that this that go back that that line returns and then the caller does the right sum.

@anarthal [2:04:09]: Yes, that's what I've been trying to say for the last hour.

@Vinnie [2:04:14]: Why didn't you say it that way? It would have been easier to understand. So what happens is the function returns.

@anarthal [2:04:18]: That's yielding

@Vinnie [2:04:20]: And the caller implement item can't understand this. OK, let me think about this. Go back to.

@anarthal [2:04:23]: I've been trying to say this for last hour. This is what I was.

@Vinnie [2:04:26]: Throw up again, throw up again.

@anarthal [2:04:29]: What do you want to see

@Vinnie [2:04:32]: I want to see the function signature.

@anarthal [2:04:35]: Oh

@Vinnie [2:04:37]: Yeah, OK, this is the this is definitely the pattern. So what, what happened is You would you would pass in the stream to write to. Yeah, that makes, yes. So, and that line, scroll down. Yeah, line 76

@Vinnie [2:04:54]: So line 76, you would co away, you can write this if you want, put it in there. Co we Out. Right And then why is it some? Why is it right sum? You do write some or do you do a full right?

@anarthal [2:05:13]: I do write some

@Vinnie [2:05:14]: OK, then say I do write some. And then your, your buffer Would go there And then we would have to

@anarthal [2:05:23]: But this will

@Vinnie [2:05:25]: Yeah, then we have to

@anarthal [2:05:25]: Actually do the writing

@Vinnie [2:05:27]: Yeah, but then we have to put the The logic for this health check. So what you do is you create, well, no, no, stop, stop, go back, so there's a, so there's some bullshit in there where it's this health check interval, you're doing something with the timer, right?

@anarthal [2:05:44]: Yeah, but that happens in, in the color.

@Vinnie [2:05:45]: So

@anarthal [2:05:47]: In connection

@Vinnie [2:05:48]: Right, so that moves, so exactly, so, so, so co routines invert the flow of control. So instead of returning something for the caller to do, what you do is the caller passes in an object that does the thing that it would do if it had returned, but instead does it in the awaitable. Let me repeat that. Co routines invert the flow of control, and they invert the responsibility. So instead of returning an object which informs the caller of what to do. The caller passes in an object that this algorith

@Vinnie [2:06:18]: M caused coal weights into and then the awaitable that it's co-awaiting implements the business logic that previously you would have had in the return value. So this would be expressed as a template or as an abstract interface, and in that call to right sum is where you put the health check interval logic.

@Vinnie [2:06:36]: Your logic moves out of the collar into the stream.

@anarthal [2:06:42]: Yeah, I'm not very fond of it, but I know what you mean, yeah.

@Vinnie [2:06:45]: But that's, but that's how it's done. And let me tell you what the, the benefit of that is now you get a component. This is actually perfect because you, now you get another testable component. That stream that you're implementing that has the right sum, that has the business logic that currently you have in the collar. Now it's factored into a separate object

@Vinnie [2:07:05]: That you can test You can't test the health check interval code without this thing that we're looking at right now. But if you factor it into a capy stream that implements the interface and has your timer. now you have another object you can test.

@Vinnie [2:07:22]: This is the beauty of co-routines is it allows you to decompose into testable units.

@anarthal [2:07:31]: What I'm hearing is, I mean, I, I know, I know this. I know how proteins.

@Vinnie [2:07:31]: You see You see it, you see it, right? You understand? You get it, right? You get

@anarthal [2:07:37]: I've, I mean, I've I, I got the, the, the, the, the, I've, I've, I got, I know how quarantines work. Thing is, there are two options. Either you do it in line or you do it in a state machine and and this has like different trade trade-offs and you need to choose between one. You do it like.

@Vinnie [2:07:58]: But this is, but this, but you don't, no, no, I'm, let me push back on that. You don't need to choose this gives you the state machine, a co-routine is a state machine. Every call weight point reflects the state, so you still have a state machine. You're just inverting the responsibilities. You express your code routine as a template or with an abstract interface, and the caller injects the behavior of the of the thing at the other end. Right now you're, look, you've, you've done the same exact thing. You are

@Vinnie [2:08:28]: Injecting behavior into this function by choosing which caller invokes it. The co routine is exactly the same. You pass in the object that that that customizes the behavior of this function, but instead of having it returned, you coawaited. It's the same thing as what you wrote. Let me repeat that. You have customized this function by changing the behavior of what happens when it returns that value. Co-routines is exactly the same thing. When you write oi on that out. See that little thing, see that out? Put

@Vinnie [2:08:58]: The cursor on it. Put the cursor on line 70, that out that thing, that's the caller. That's the change that, change, put the word collar there.

@anarthal [2:09:06]: I know, I know, I know this. The thing I can't do, the thing I can't do with this is intercept this and say, OK, it's suspended here.

@Vinnie [2:09:07]: That's the powers.

@anarthal [2:09:19]: Now I'm gonna do some stuff and I'm gonna I'm gonna check the state of the connection. I'm going to modify the state of the connection to simulate some sort of condition. That's the

@Vinnie [2:09:28]: Wait a minute, wait, sorry, excuse me. Say that again. Of course you can do that

@anarthal [2:09:35]: When I go

@Vinnie [2:09:36]: What do you mean? Of course you can do that. What are you talking about? You have the co routine handle. You could call resume You, you can invoke it and then you can, when it suspends, you get control. Now you're, now you're at that point. Now you do what the hell of whatever you want to do, you do anything that you want when you want to run it again, you call resume.

@Vinnie [2:09:55]: It's perfect. You have complete control over the state machine. Just don't, from the call it's just construct the task and then call resume on it when whenever you want and you can, you could step through your whole state machine.

@anarthal [2:10:10]: Is this the handle even public in on task?

@Vinnie [2:10:12]: Of course it's public. Come on. You, you read the paper, I will wait, I owe runnel exposes the handle.

@anarthal [2:10:18]: Yeah, it exposes the hand, yeah, How do I know it's in right sum? With the handle?

@Vinnie [2:10:24]: How do you know, how do you know now?

@anarthal [2:10:27]: Because it yields a truck same right some. I know by by design.

@Vinnie [2:10:30]: So what did I go back. Go back, go put the cursor over the word out. Change that to the word collar. Whatever you're doing now with the return value You can do in right sum.

@Vinnie [2:10:47]: So if you want, you implement the caller, so that you just put the struct in the stream at the other end. So when you do the co weight, it knows where you're at. So after you get control again when the co routine yields, now you have some information that tells you where it's at if you want. I mean, if that's how you want to do it.

@anarthal [2:11:05]: What I think I would satisfy me and I think this is the, the, the core mock I wanted to write is having the logic, having Like having every time that that that like

@anarthal [2:11:25]: Every time you get to this.

@Vinnie [2:11:26]: Let me help, let me help, let me help you, let me help you. In your object in right sum, increment, a counter, and then you know which where you're at.

@anarthal [2:11:35]: Yeah, well, I don't think that doesn't work like this because you need to know like which function was called and which with which arguments, but it that's doable.

@Vinnie [2:11:42]: We then add it, add it to the front, add it to the parameter list if you want. What can I say?

@anarthal [2:11:44]: That's doable. That's doable.

@Vinnie [2:11:49]: Yeah, it's all doable. Everything is doable. We don't return anymore. We don't, we do, we do. We don't talk, we don't talk about what we're doing, we do it. Line 76 top line 76 talks, line 77 does.

@Vinnie [2:12:06]: Line 76 says, please do this for me. Line 77 says, I'm doing this now.

@anarthal [2:12:07]: Yeah

@Vinnie [2:12:13]: That's the advantage of co-routines. Are you going to have to refactor your code? Yes, yes, cold routines are viral. The the call wait propagates upwards. It's going to force you to rewrite everything. That's no, there's no question, there's no question about that. It's gonna happen. And you know what, it's great because AI can help. It's very good at that.

@anarthal [2:12:26]: Yeah, and that's something. So, for me, I'm kind of missing a testing piece, but I'm gonna come up with that myself.

@anarthal [2:12:44]: Because I know how I like to test and we have very different ways of testing. So I'll end up coming up with a piece that satisfies me.

@Vinnie [2:12:52]: Have I convinced you that I have I convinced you yet of the utility of expressing this as a pipeline?

@anarthal [2:12:59]: I mean, I've, my question was like, are you open to generators and my, and the answer is no. It's like this is the way. I know, I know the tradeoffs, I knew the tradeoffs before you told you before you told me. I know.

@Vinnie [2:13:05]: Well, that's not what we can. So we can implement a generator, we can implement a generator. That's, that's easy, that's silly. It's a generator is nothing. I can, I could implement that right now. I'm going to tell Claude. Claude in a generator for Kay and it'll do it.

@Vinnie [2:13:22]: So it's not really a big deal You can open an issue will add it

@anarthal [2:13:26]: It depends on what the, it depends on what the, I mean a basic generator, yes, I copied one from Si reference and it works. And then I told I told Corso to upgrade it and to allow it like nested generators, for composibility and it kind of works, but it's dirty.

@Vinnie [2:13:46]: OK, so, so that

@anarthal [2:13:47]: If we are not So, I would say like I don't think this is the best use of, of the team's time, waiting a generator right now, because if we are very convinced that we are going down there, do it and not, yield it

@anarthal [2:14:07]: I don't think it makes sense right now. I think I can come up with some piece of testing machine that I may be able to contribute back to Kay if, if, if it ever ends up being a very different from what you have that allows me to write this and and, and be happy testing, mocking it, you know,

@Vinnie [2:14:30]: I think that's, I think it's good.

@anarthal [2:14:34]: The good thing, that this hat is that this is very universal in the terms that if you want, if you end up expressing all your protocol logic like this without an al weight and you want to, I don't know, use Python in the Python interpreter core routines, whatever they are, you can and you could probably write like the fastest python, HDP server ever with this, and this was, this is kind of

@anarthal [2:15:04]: Good point of this, but I understand also that this is much simpler and this is much more expressive.

@Vinnie [2:15:11]: Thank you. I appreciate that.

@anarthal [2:15:11]: You know what I mean

@Vinnie [2:15:13]: I put a lot of effort into optimizing that task, by the way.

@anarthal [2:15:17]: So if we are gonna focus on this, my question was like, are you open to this and I understand that the that the answer right now is like, not really because we are putting a lot of into this.

@Vinnie [2:15:28]: Oh wait, no, that's not, no, that's not, that's not fair. Are you saying, are you open to this? I'm open to whatever you, you can put whatever you want in your if that's how you want to write it, you could write it that way. I don't, I'm not opposed

@anarthal [2:15:33]: Yeah. Yeah, I mean, sure, but I mean, we are like we want this and not this. This is what I've understood.

@Vinnie [2:15:44]: I just don't understand how the relevance of line 7, what is the relevance of line 73 to Cappy. What's the connection How does Cappy support 173?

@anarthal [2:15:54]: That Saying something like creating this as a copy generator if this ever comes to comes to happen of, right?

@Vinnie [2:16:09]: So you're telling me the only, you're telling me, OK, so all you wanna do is just yield that struct

@anarthal [2:16:16]: It's yield, it's not as simple because then you May want to do like nested generators, generators that call each other and because a normal generator is I told Kororran it generated one. So if this is true though.

@Vinnie [2:16:28]: Yeah, but you do OK what that's all very nice and that's all very fine, but at the end of the day what you're talking about is you want to make this function that can yield values to a calling code routine. That's it. It's literally what you're talking about

@anarthal [2:16:40]: Yeah. Yeah But

@Vinnie [2:16:44]: Well, I mean, of course you can do that if you want.

@anarthal [2:16:45]: It But yeah, so let's leave it like it here we are, I'm gonna try to make myself comfortable with writing tests for this. OK? I'm gonna try to solve this in my end.

@Vinnie [2:17:00]: I think we can, I mean, I think, I think If you provide the use case for yielding values, we're, I'm very happy to implement it But I don't have a need for it myself, so it's hard for me to have any opinion. I mean, my opinion is is that Cappy should offer things that are useful, that is within its domain. Is is this within the domain? It probably yes, but without the specific use case, I can't address it.

@anarthal [2:17:25]: Yeah, that's, that's pretty fair. So for now, I'm just gonna not push forward with this because I don't think I can describe things enough the case like the use case good enough for you guys to implement something meaningful.

@anarthal [2:17:41]: So I'm not gonna push, I'm not gonna push on this. I just wanted to know like kind of your opinion, and I think I've got it. Now,

@Vinnie [2:17:41]: OK, that's fair. I appreciate that. Let me ask you a question. Oh, are you done? Wait, no, what's that? Tell you first.

@anarthal [2:17:52]: Yeah. Yeah. So the last thing I wanted to say is that I think that you should propose both CAI and Skorrosio, a separate libraries, but at the same time.

@Vinnie [2:18:04]: A little bit, please, please. I, I already posted 3 hours ago. I posted that 3 hours ago. You're behind. You don't have no. You need to turn notifications on and you need to turn off batching for the mailing list.

@anarthal [2:18:16]: Oh, I, OK, I missed that. Fair. Sorry.

@Vinnie [2:18:19]: That would be, that would, that was, that was a joke. Steve, are you subscribed to the Boost mailing list. Say nothing if you are. Michael, are you subscribed to the boost mailing list? Say nothing if you are. Perfect. They're both subscribed. OK, great.

@Steve Gerbino [2:18:34]: No, I'm not. I gotta subscribe, like, all right, you called me with my rasta.

@Vinnie [2:18:35]: Oh OK. Yeah Subscribe to

@Steve Gerbino [2:18:40]: I'm putting my pants on. Hold on, sir, I'm putting them on.

@anarthal [2:18:46]: I mean, I, I subscribe and they I just happen to be to I focused on the generator thing to, to see the, the email. Yeah, I've seen it now. See, so I'm done, yes.

@anarthal [2:19:07]: Beanie.

@Vinnie [2:19:08]: Yeah, sorry, I'm sending a message. OK, OK, are you done?

@anarthal [2:19:08]: Are you there Yeah, I'm done

@Vinnie [2:19:14]: Was this a good discussion

@anarthal [2:19:17]: Yeah, I think it was

@Vinnie [2:19:18]: Was I too hard on you

@anarthal [2:19:19]: Was it No, I mean, I'm trying to make my points. It sometimes takes me effort because I'm not very used to discussing unless in English. But yeah, you know

@Vinnie [2:19:31]: You did a great job. No, you, you, you did a great job. It was my lack of understanding And my follow on my end.

@anarthal [2:19:36]: Yeah, I Even if I like speak very slow and, and the like and some sometimes feel like I'm an undergrad, but like I tend to know what I'm talking about, but I'm sometimes just don't know how to phrase things.

@Vinnie [2:19:53]: That you did find the problem was on my end, you know, you're as a staff engineer at the C++ Alliance, you are very difficult with that endorsement. I'm just saying

@anarthal [2:20:03]: Yeah, I mean, if I'm like I'm and I'm, the thing is that when you are actually interested in something, I'm, I'm genuinely interested in in this because I'm, I, I do my skill and I do this and I'm going to do postgrads. So I'm like

@Vinnie [2:20:15]: Oh

@anarthal [2:20:20]: Maybe one of the biggest users in boost that you have. So I just want things like to be clear and where we are and, and yeah, I'm a difficult person. No one's gonna discuss that.

@Vinnie [2:20:29]: Listen, I, I. Yeah I apprec I appreciate it and I also don't, I also what I appreciate is that even though you work with the C alliance, you're, you're still intellectually honest. You don't endorse it just because it's one of our projects. You give real pushback, and I think, I think that's worth pointing out on the mailing list, you know, I, you know, I mean, you're, you're a traffic here.

@anarthal [2:20:49]: Yeah, I mean, I like, I like pushing back things because I think it generates very interesting discussions and I especially when I have actual code written for that, and I, I see things. I sometimes don't know how to phrase them, but I see things.

@Vinnie [2:21:07]: I understand everything perfectly now, and I think

@anarthal [2:21:10]: Now my question is, has this been useful to you?

@Vinnie [2:21:14]: Well, this is the most useful conversation that I've had in the I probably this is when, when you see the results of the transcript, you're going to just be amazed. This is absolutely fantastic. I think this validates the design of the library. I think it's very interesting to see that you wrote code that exactly duplicates one of the patterns that we formalized. That just, that gives me more confidence that the abstractions that we've discovered are useful. So, you know, looking at your use case, looking like looking at your code,

@Vinnie [2:21:44]: Has really helped to shape our priorities and to help us understand what we need to do to deliver value that we have to have a good ASIO story. We have to have a good co-routine story. We have to deliver what users want. We need to, it needs to be possible to port things, and it needs to be possible for people to keep their ASIO implementation and to be able to incrementally update. We have to deliver value. This is what's important.

@anarthal [2:22:08]: Yeah, that's the thing that I was trying to tell you like last time, but I, once again, I do like a very bad job of explaining things, which is like, even if you have AI it's

@Vinnie [2:22:16]: No, no, no, no, no. No, no, no, no, no. No, you did a perfect, I, you, I understood you perfectly and you were right. And that's why I wrote that example of ASIO. I did, I implemented ASIO on both sides of, of the conversion layer. There's a capy wraps asio and then Aio wraps Cappy. That's why I did, I did that because of your feedback.

@Vinnie [2:22:36]: But I didn't do enough because your need, your needs are greater than that simple example.

@anarthal [2:22:41]: Yeah I mean the executor thing for example is like a bit poor because you don't, I mean, I don't know if this is a limitation of the pattern itself, but in, when you dispatch, you actually do an an assa post. So this is like a bit

@anarthal [2:22:59]: You know

@Vinnie [2:23:00]: Show me if if after we get off, I think just show me that. I'll look at it and see if I can clean it up.

@anarthal [2:23:09]: How can I say this?

@Vinnie [2:23:09]: But there's gonna be, there's gonna be, there's gonna be friction at the boundary, right?

@anarthal [2:23:14]: Yeah, I know This is one of the reasons why I wanted to push the generator thing because then you have actually no boundary, but I understand this may not be as clean. Kay

@Vinnie [2:23:25]: Do you want me to implement the generator?

@anarthal [2:23:29]: Not yet because I can't give you like a concrete prompt. And if I can't give a, give a concrete front like I don't think that's for you. So how can I look for this code,

@Vinnie [2:23:40]: It's in the example

@anarthal [2:23:41]: It's in copy

@Vinnie [2:23:42]: Happy example

@anarthal [2:23:42]: Sample

@Vinnie [2:23:44]: Lay slash example. Yeah, there it is, you have it right there.

@anarthal [2:23:46]: Aaapi OK, so, There is copy streams and there is an executor somewhere as I executor maybe.

@Vinnie [2:23:57]: Yeah. It's probably, yeah

@anarthal [2:24:00]: Yeah, so

@Vinnie [2:24:00]: There Fair.

@anarthal [2:24:04]: This and this This is I don't know if this is because of the friction itself of, but this is not very It's not speaking very well of, of the interaction.

@Vinnie [2:24:17]: We'll just change it to dispatch.

@anarthal [2:24:22]: And I just do that

@Vinnie [2:24:24]: I don't know Try it. See what, see what happens.

@anarthal [2:24:28]: I mean, the thing is that, is it correct?

@Vinnie [2:24:32]: I think

@anarthal [2:24:32]: I don't have the, I don't have to configure the bills, so I

@Vinnie [2:24:36]: I mean, I don't know I don't know, I have to look into it. And also, are we, do we destroy the like how do we destroy the co routine handle? Do we have to have a destructor? Or what

@anarthal [2:24:45]: I have no, I have no idea. I have no idea.

@Vinnie [2:24:47]: Yeah, let me, let me ask Claude.

@anarthal [2:24:50]: The buffer thing, the buffer sequence thing. I like, and I've now, I like what I've seen. I think, we are somehow missing a concrete way of doing like typepora. Because we've got like 2 classes doing buffer sequence cyber racing at this 0.1 for Corrosio and I think there is some similar in Kapi and for me it is like something that smells a bit.

@Vinnie [2:25:20]: Well, you want to keep the buffer sequence intact as long as you can. You want to keep it as a template for as long as possible to enable composition and only at the very end of your co-routine chain, then you convert it into a spend. That's the best way to do it. And so Cappy provides one way to do it, and then Corozio provides a slightly different method.

@Vinnie [2:25:45]: Actually, I'm only, I think I got rid of the Corozio one. Let me double check that.

@anarthal [2:25:49]: No, the cholosio is there. I think you may have gotten the rid of the copy one. I don't know. I, I don't use buffer sequences enough to To have an opinion on this, but what I can tell you is when I was trying to write a generator which was gonna return a buffer sequence. I wanted to return a concrete type and I couldn't because

@anarthal [2:26:15]: And then you have the concrete

@Vinnie [2:26:16]: There's buffer there's there's buffer array now. Cappy has buffer array. You can return that.

@anarthal [2:26:21]: If you go to the

@Vinnie [2:26:23]: Offer on your array.

@anarthal [2:26:26]: To the serializer

@Vinnie [2:26:27]: Buffer underscore

@anarthal [2:26:28]: You've got like Yeah, I've seen that, I've seen that The serializer, this is boost HDP, yeah. This has somewhere Konspaus type

@anarthal [2:26:44]: And mutable buffer type, right?

@Vinnie [2:26:47]: Mhm. Pair

@anarthal [2:26:50]: It would be

@Vinnie [2:26:51]: That's no good. We need, we need to use a regular span there.

@anarthal [2:26:56]: We, it would be good if, I mean, maybe this is not possible because of performance or other considerations, but it would be good like having a single type or type that you could use everywhere. It would simplify things a lot. I think, but once again, maybe I don't

@Vinnie [2:27:10]: But No buffer, this, this is correct, line 84 is correct as written. That's the correct type. I mean, that's the right concrete type.

@anarthal [2:27:22]: But anyway, that's kind of, that was like my two sons.

@Vinnie [2:27:25]: When you tell me what, let's do this, when you, when, when you have a situation with the buffer sequence, ping me on Slack and let me help you try to find the right solution.

@anarthal [2:27:26]: I. Days.

@Vinnie [2:27:34]: Because there's a number of solutions and they are all dependent on what you're trying to do for the use case. So without knowing the specific use case, I can't tell you what's best.

@anarthal [2:27:41]: Yeah I don't have anything right now, but I've liked what I've seen of the circular buffer and the flat buffer types. I think these are genuinely useful.

@Vinnie [2:27:51]: Cool That's great to know Are we good? We're done. The long call.

@anarthal [2:27:55]: OK. Yeah.

@Vinnie [2:27:57]: All right, I need everybody to leave. Get the fuck out. Leave the, leave the huddle now, please. Michael, get out

@anarthal [2:28:04]: See ya

@Vinnie [2:28:05]: Goodbye, Reuben. Thank you. And we're done