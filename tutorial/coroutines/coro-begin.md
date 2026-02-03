
Search
Write
Sign up

Sign in



BYJU’S Exam Prep Engineering
You're reading for free via Faheem un nisa's Friend Link. Become a member to access the best of Medium.

Member-only story

Coroutines for beginners: the only article you’ll need to get started!
Faheem un nisa
Faheem un nisa

Follow
7 min read
·
May 2, 2021
711


3



Press enter or click to view image in full size

When we shifted to Kotlin at 
Gradeup
, we realised that the www was pretty unfriendly for a coroutine beginner- there was a lot of information available but one had to really go through a ton of articles for Coroutines to start making any sense! I often ended up fumbling through the internet, having words like scope, dispatchers, builder, asyncawaitsuspendaaaaaaaah! thrown in my face, and after I could finally decipher Coroutines, I decided to use my bad experience to create a good one for all the fellow Coroutine-noobs out there. So here’s presenting this comprehensive article!

We shall be covering the following:

What are Coroutines?
Suspend functions.
Coroutine Builders.
Coroutine Dispatchers.
Coroutine Scope.
Coroutine Jobs.
Error handling in Coroutines.
1. Say, what are Coroutines?
Coroutines are “light weight threads” : if you say that one more time, I am going to hang myself, pun intended :D

If you have searched for Coroutine tutorials online, chances are that you must have come across this oversimplified statement: Coroutines are light weight threads. What does that even mean? How did the good old thread lose so much weight that it ended up with a new personality?

Well, following are a few cool facts that will help you understand the basics of coroutines, the difference between threads and coroutines and how the two are linked :-

Co + Routines: as the name suggests, coroutines are ‘routines’ (or functions) that work co-operatively
Coroutines are NOT a replacement of threads; they only provide us with a framework to efficiently manage the good, old thread that we are already familiar with.
Multiple coroutines can run on a single thread, such that one thread is utilised in a far more efficient manner. This is helpful when a thread is sitting idle and can execute a few lines of another function, instead of just being a couch potato!
Threads are limited in number but we can have as many coroutines as we want.
Threads are managed by the OS while as the coroutines are managed by the user
Threads consume a considerable amount of memory (1–2 MB per thread) and switching between them is quite expensive. Coroutines, on the other hand, do not map on the native thread (lightweight threads), so they do NOT require context switching on the processor and hence are faster.
Coroutines are available in many languages and, basically, are of two types: Stackless and Stackful. Kotlin implements Stackless coroutine- which means that the coroutines don’t have their own stack, and so don’t map on the native thread.
With that out of our way, let’s see what a typical coroutine looks like.

You might encounter unfamiliar words (marked in bold in the following code block) but have some faith; we shall circle back to all those in this article!

suspend fun fetchDataFromServer(){
    Scope.launch(Dispatcher + Job + errorHandler){
          DoSomeWork()
      }
}
2. Suspend Functions
You can think of the “suspend” keyword as sort of a declaration that the function is time consuming. A suspending function is simply a function that can be paused and resumed at a later time. They can execute a long running operation and wait for it to complete without blocking. The syntax of a suspending function is similar to that of a regular function except for the addition of the suspend keyword. It can take a parameter and have a return type. However, suspending functions can only be invoked by another suspending function or within a coroutine.

Under the hood, suspend functions are converted by the compiler to another function without the suspend keyword, that takes an addition parameter of type Continuation<T> . Here is a nice, detailed article on suspending functions, in case you want to delve a little deeper.

suspend fun doSomething(){
   //makeApiCall
}
3. Starting a coroutine with Coroutine Builders
A Coroutine can be started using either of the following functions

Launch
Async/Await
These are also called Coroutine Builders.

Launch: is a fire-and-forget type of coroutine that does not return any value.
GlobalScope.launch(Dispatchers.IO) {
    fetchDataAndSaveInDatabase() 
}
2. Async: async{}returns an instance of Deferred<T>, which has an await()function that returns the result of the coroutine like we have future in Java in which we do future.get() to the get the result.

GlobalScope.launch(Dispatchers.Main) { 
    val dataA = async(Dispatchers.IO) { fetchFirstData() } 
    val dataB = async(Dispatchers.IO) { fetchSecondData() } 
    showUsers(dataA.await(), dataB.await()) // back on UI thread
 }
withContext
withContext is used for context switching. Say you are on Dispatchers.Main but you want to shift to Diapatchers.default, you can use withContext. It is nothing but another way of writing the async where we do not have to write await().

suspend fun fetchData(): User {
    return withContext(Dispatchers.IO) {
        // make network call
        // return data
    }
}
Let’s use withContext in our async example of fetchFirstData and fetchSecondData in parallel.

suspend fun fetchdata(){
   GlobalScope.launch(Dispatchers.Main) {
      val dataA = withContext(Dispatchers.IO) { fetchFirstData() }
      val dataB = withContext(Dispatchers.IO) { fetchSecondData()  
      showUsers(dataA, dataB) // back on UI thread
       }
}
When we use withContext, it will run in series instead of parallel (as with async).

On

Alright, but when to use which one?
Both withContext and async can be used to get the results, which is not possible with launch.
Use withContext when you do NOT need parallel execution.
Use async only when you NEED parallel execution.
Use withContext to return the result of a single task.
Use async for results from multiple tasks that run in parallel.
A big difference between launch and async is how they handle exceptions. Async expects that you will eventually call await to get a result (or exception) so it won’t throw exceptions by default. That means if you use async to start a new coroutine it will silently drop exceptions.
4. Coroutine Dispatchers
Dispatchers help coroutines in deciding the thread on which the work has to be done. There are majorly three types of Dispatchers, and we can use either:

IO dispatcher : is used to do the network and disk-related work.
Default dispatcher: is used to do the CPU intensive work.
Main dispatcher: is used to do work on the UI thread of Android.
override fun onCreate(savedInstanceState: Bundle?) { 
   super.onCreate(savedInstanceState)
   GlobalScope.launch(Dispatchers.IO)
      {
       fetchDataFromServer()
      } 
}
5. Coroutine Scope
In Kotlin, Coroutines must run in something called a CoroutineScope. A CoroutineScope keeps track of your coroutines, even coroutines that are suspended. To ensure that all coroutines are tracked, Kotlin does not allow you to start a new coroutine without a CoroutineScope. A CoroutineScope keeps track of all your coroutines, and it can cancel all of the coroutines started in it. This fits well with Android development where you want to ensure that you clean up everything that was started by a screen when the user leaves.

There are basically 3 scopes in Kotlin coroutines:

Global Scope: When Coroutines are launched within the global scope, they live long as the application does. Coroutines launched in the global scope will be launched in a separate thread. Since the lifetime of this coroutine shall be decided on the basis of application lifetime and not the life of the activity, it can lead to memory leaks and hence must be used judicially and only after proper deliberation, if and whenever required.
GlobalScope.launch {
   //doSomething
}
2. LifeCycle Scope: The lifecycle scope is the same as the global scope, but the only difference is that all the coroutines launched within the activity also die when the activity dies.

lifecycleScope.launch {
    //doSomething
}
3. ViewModel Scope: The coroutine in this scope will live as long the View Model is alive.

viewModelScope.launch {
    //doSomething
}
We also use a so-called top-most CoroutineScope when we want to control the lifecycle of the Coroutines we launch, so that we can cancel them and handle any exceptions.

CoroutineScope(Dispatchers.IO + job + errorHandler ).launch{
    //doSomething
}
6. Coroutine Job
A Job is a handle to a coroutine. For every coroutine that you create (by launch or async), it returns a Job instance that uniquely identifies the coroutine and manages its lifecycle. We can also pass a Job to a CoroutineScope to keep a handle on its lifecycle.

A Job can be used to wait for the coroutine to do some work or it can be used to cancel it. Coroutines can be controlled through the functions that are available on the Job interface. Some functions out of many that job interface offers are as follows:

start()
join()
cancel()
A Job can go through a set of states: New, Active, Completing, Completed, Cancelling and Cancelled. While we don’t have access to the states themselves, we can access properties of a Job: isActive, isCancelled and isCompleted.

val job = GlobalScope.launch(Dispatchers.Default) {
//doSomething
}
We have the following options while creating a job:

Job: the cancellation of any one of the children causes the whole Job to fail.
2. SupervisorJob: Children of a supervisor job can fail independently of each other. A failure or cancellation of a child does not cause the supervisor job to fail and does not affect its other children; so a supervisor can implement a custom policy for handling failures of its children.

7. Exception Handling
The article shall not rest in peace if it digresses from the thing that makes Coroutines fool proof— exception handling!

While using launch, we can handle exceptions in 2 ways:

Use the good-old try-catch block
CoroutineScope.launch(Dispatchers.Main) {
 try {
     doSomething()
   } catch (exception: Exception) {
    Log.d(TAG, “$exception handled !”)
   }
}
2. Use CoroutineExceptionHandler

For this we need to create an exception handler like below:

val handler = CoroutineExceptionHandler { _, exception ->
 Log.d(TAG, “$exception handled !”)
}
Then we can attach the Handler as follows:

CoroutineScope.launch(Dispatchers.Main + handler) {
   doSomething() 
}
When Using async, we need to use the try-catch block to handle the exception like below.

val deferredUser = GlobalScope.async {
    fetchUser()
}
try {
    val user = deferredUser.await()
} catch (exception: Exception) {
    Log.d(TAG, "$exception handled !")
}
The world of Coroutines is vast and this article merely touches the surface of the ocean; nevertheless, I hope this simplifies things for you and helps you get started in your journey of migrating to Coroutines!

Happy coding, peeps!

Fin!

If you would like to send a little encouragement my way, here is the link for that :)

Kotlin
Coroutine
Asyncawait
Launch
Kotlin Coroutines
711


3


BYJU’S Exam Prep Engineering
Published in BYJU’S Exam Prep Engineering
183 followers
·
Last published Aug 25, 2022
The Most Comprehensive Preparation App for All Exams


Follow
Faheem un nisa
Written by Faheem un nisa
201 followers
·
17 following
Senior Technology Manager @ JioCinema


Follow
Responses (3)

Write a response

What are your thoughts?

Cancel
Respond
Sahil Chopra
Sahil Chopra

Mar 15, 2022


This is wonderful. Everything summarised in a single article. Great work and really helpful.
1

Reply

Jonathan Campos
Jonathan Campos

May 4, 2021


Well done! It’s a very nice article about Coroutines, explaining everything in a simple way and super easy to understand.

1 reply

Reply

AndroidDeveloperLB
AndroidDeveloperLB

May 2, 2021


Hello I still have some questions:
1. So Coroutines are just a thread pool with work-stealing?
2. About "launch" vs "async" vs "withContext", how does the "showUsers" function look like? It's a simple function that runs on the UI thread?
3. What's the…more

1 reply

Reply

More from Faheem un nisa and BYJU’S Exam Prep Engineering
The simplest guide to understanding Gradle!
BYJU’S Exam Prep Engineering
In

BYJU’S Exam Prep Engineering

by

Faheem un nisa

The simplest guide to understanding Gradle!
Gradle: how often do we neglect this beautiful build system and treat it as a black box, blindly copy pasting code into it! While that is…

Feb 15, 2021
474
1
Upgrading to React 17
BYJU’S Exam Prep Engineering
In

BYJU’S Exam Prep Engineering

by

Bhoomika Chauhan

Upgrading to React 17
Everything you need to know about the new updates in React 17
Mar 21, 2021
101
Asynchronous Javascript (Event Loop)
BYJU’S Exam Prep Engineering
In

BYJU’S Exam Prep Engineering

by

Ishika Gupta

Asynchronous Javascript (Event Loop)
Javascript is a single-threaded language at runtime which means only one thing can happen at a time. That is, the JavaScript engine can…
Aug 13, 2020
512
6
Cracking the Kotlin Interview
Mobile App Development Publication
In

Mobile App Development Publication

by

Faheem un nisa

Cracking the Kotlin Interview
Are you preparing for an Android interview but don’t know where to start because there is just so much to cover? Or have you been preparing…

Jul 11, 2023
81
See all from Faheem un nisa
See all from BYJU’S Exam Prep Engineering
Recommended from Medium
6 brain images
Write A Catalyst
In

Write A Catalyst

by

Dr. Patricia Schmidt

As a Neuroscientist, I Quit These 5 Morning Habits That Destroy Your Brain
Most people do #1 within 10 minutes of waking (and it sabotages your entire day)

Jan 14
22K
378
Stanford Just Killed Prompt Engineering With 8 Words (And I Can’t Believe It Worked)
Generative AI
In

Generative AI

by

Adham Khaled

Stanford Just Killed Prompt Engineering With 8 Words (And I Can’t Believe It Worked)
ChatGPT keeps giving you the same boring response? This new technique unlocks 2× more creativity from ANY AI model — no training required…

Oct 19, 2025
23K
595
The AI Bubble Is About To Burst, But The Next Bubble Is Already Growing
Will Lockett
Will Lockett

The AI Bubble Is About To Burst, But The Next Bubble Is Already Growing
Techbros are preparing their latest bandwagon.

Sep 14, 2025
22K
924
I Thought I Knew System Design Until I Met a Google L7 Interviewer
Beyond Localhost
In

Beyond Localhost

by

The Speedcraft Lab

I Thought I Knew System Design Until I Met a Google L7 Interviewer
A single whiteboard question revealed the gap between knowing patterns and actually designing systems that scale.

Dec 21, 2025
4.6K
85
Apple Just Fired the Designer Who Made iOS 26 Unreadable. Here’s What Truly Happened.
Mac O’Clock
In

Mac O’Clock

by

Nov Tech

Apple Just Fired the Designer Who Made iOS 26 Unreadable. Here’s What Truly Happened.
Now he’s going to Meta, Stephen Lemay is taking over, and Apple employees are publicly celebrating. Here’s the full story.

Dec 21, 2025
7.1K
247
I Stopped Using ChatGPT for 30 Days. What Happened to My Brain Was Terrifying.
Level Up Coding
In

Level Up Coding

by

Teja Kusireddy

I Stopped Using ChatGPT for 30 Days. What Happened to My Brain Was Terrifying.
91% of you will abandon 2026 resolutions by January 10th. Here’s how to be in the 9% who actually win.

Dec 28, 2025
2.1K
75
See more recommendations
Help

Status

About

Careers

Press

Blog

Privacy

Rules

Terms

Text to speech