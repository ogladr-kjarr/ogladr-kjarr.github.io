---
layout: post
title: Study Summary - 001
date: 04/06/2022
categories: Programming
tags: Java
---

Having spent a couple of months studying Java, I decided I'm going to make summary posts to keep track of what I've been doing. This will include books read, videos I have found useful, and any projects I have been working on.

# Books

**Java - A Beginner's Guide (6th Edition)**: This book covered up to Java 8. It was a good introduction to the syntax of the language and with exercises that were easily approachable. It covered many of the needed topics like control structures, interfaces, IO, exception handling, generics, and lambdas/method references.  It read very much like a set of lectures, each chapter a new lecture. It was surprising that NIO package only received half a page of text, compared to the entire chapter on the older IO package.

I feel like this is the best book for a beginner I've read. It covered the main topics and had plenty of code to explain what was going on.  I did find the concurrency chapter a little mysterious, I understood the mechanics but not the how of implementing in a real program.

**Learning Java (5th Edition)**: This book was the second introductory book I read. Unlike *A Beginner's Guide* this felt more like a set of tutorials rather than lectures, although there were no code puzzles or questions at the end of the chapters which I think was a shame, as programming challenges can really help cement ideas through doing. A lot of the topics covered were the same as *A Beginner's Guide* as to be expected, however there were some topics that were completely new, these included: regular expressions, logging, the collections framework, the new date time library, NIO package, the net package, servlets, history of generics, a brief introduction to the modern concurrency helper features, and an interesting note on the procedure of how Java evolves over time.  These topics may be in newer versions of *A Beginner's Guide*, I don't know. 

This is another good book, I think it compliments *A Beginner's guide* as it feels like a very different style, so it doesn't seem like you're just reading the same information in the same way. The only issue I had was that the IO/NIO chapter was very dense, and while I got the gist of it, I'll need to get some hands on practise before all the combinations of objects and byte/character types make sense.
 
**Java in a Nutshell (7th Edition)**: 

This was the third introductory book I read. It was generally more dense than the first two books, moving at a quicker pace and providing more detailed information on each topic, going past perhaps what a beginner needs to know. For example the chapter on class loading and reflection is one where I couldn't see the importance of the knowledge as a beginner, similarly with the chapter on Java memory use and garbage collection techniques. It's the first book I've read that covered the module system, which was worthwhile. I still enjoyed this book and think it is a good resource, however I think this book is suited to an intermediate programmer from anther language, as many of the techinques and details seemed esoteric to me as a beginner. 

**Real-World Software Development**: A Java book that explores SOLID and general good programming practices through a project-orientated approach.  This was my first introduction to SOLID practices, and it made sense with examples to demostrate the different properties of SOLID. It also gave an introduction to unit testing, lambdas, exceptions, and build tools.

Though it made sense while reading, I think this will be a good book to go back to once I have more experience to really appreciate the lessons that it teaches. I felt on my first read-through, because I'm lacking in experience, I didn't fully appreciate the why of all the examples.  I also found the later chapters to be quite challenging, to the point I felt I wasn't taking in the lessons they were trying to teach. I still think this is a good book with good code examples and end of chapter code challenges.

**Test-Driven Java Development**: This book really drilled home the *Red Green Refactor Process*. So far for some toy projects I have tried implementing the TDD approach and I enjoyed it. The book gave me the confidence that what I had implemented worked, and gave a safetly harness for refactoring,introducing JUnit and other tools. I think this was a good choice of book to pick up as I have never used TDD in any role or university module, but I think it is the way I'll try and approach programming as I learn more.

**Java 8 Lambdas**: This was a book dedicated to lambdas and method references. It really bolstered my knowledge of the streams orientated way of doing things. Having read this book I now feel much more comfortable reading the Java Docs on the streams components and methods than I did after reading the more general introductory Java books.

**97 things**: This is a good book for a beginner, maybe someone between first and second year at university. A lot of the topics are how to approach programming, highlighting modern coding techniques, attitude, tooling, and JVM language options. The length is a little frustrating, as some of the articles would make excellent longer pieces. The biggest take away for me for learning to code is to make the smallest possible changes, using TDD, and commit each small change. I thought commits were for bigger or large changes, for full features, but this isn't so.

# Videos

**Write Better Functional Java Code - Brian Vermeer (YouTube)**: This video helped a lot with understanding what is good practice with lambdas, especially how to use the Optional type which the books I've read didn't go into, a very helpful talk.

**Java Streams: Beyond The Basics - Simon Ritter (YouTube)**: This video, like the first one, helped with general understanding what is good practise with lambdas, what type of code to avoid, a very enjoyable talk.

**Exploring Collectors - Venkat Subramaniam (YouTube)**: This was a masterclass, lightly touching on the common stream functions, but focusing mostly on the collectors and the myriad ways in which they can be used. It is highly in-depth and had a lot of takeaways to think on further, and probably I will use this for reference in the future.

**Back to Basics: Threads - Adam Dubiel (YouTube)**: This was an interesting talk on the way Java threads run on the Linux server, how you can identify and track what threads are doing using standard Linux process analysing tools. It also touched on how Java threads are created and performance issues that may occur. I was looking for a talk that explained the use of threads in Java more from an application programming point of view, but this talk was interesting with great tips on how to better understand threads as you use them.

**Java 17: New and Exiting! - Simone Bordet (YouTube)**: An interesting talk listing the many new features that have appeared in Java version 8 to 17. Most haven't been covered by the introductory books I've been reading, due to the language moving fast for the newer books not to have them, and buying out of date books because they're cheap. The features I want to look into include: the module system, reactive Flow class, new HTTP client, expression switch, text blocks, pattern matching instanceOf, records, and sealed interfaces/classes.

**Parallel Streams, CompletableFuture, and All That: Concurrency in Java - Kenneth Kousen (YouTube)**: A very good speaker, went over stream parallelisation which was easy to follow having read about it and watched other videos. However he then introduced completable future, a completely new concept to me. There was a demo at the end of parallel downloading and processing JSON data that at the time went too fast for me to follow, but the repo is on GitHub so I'll be checking it out as I think it would be good to learn from.

# MOOC.fi Java Programming I and II

After reading many posts in the *learnjava* subreddit, I decided to try the MOOC from the University of Helsinki. It uses their modified version of Net Beans, where you log onto the course in the IDE and it downloads the exercise classes for you, downloading the next sections as you get to the end of the current one so I guess you're not overwhelmed at the start. The IDE integration with their servers for checking the correctness of the exercises is fantastic to get real time feedback on exercise solutions. A lot of work has gone into this course and it is very good.

The only issue I had was in part 4 where I couldn't run local tests for some exercises, which was due to using Java 17. I used the internet checker from that point on and it was fine. Some code style points stuck out, like using -1 as a return value to indicate no values, or returning null to indicate empty. Some of the books I've read and videos I've watched make me think the Optional type would have been better in these situations.

I think this course would be a very good place for a beginner to start learning Java, before reading any introductory books. It covers the basics well, and uses the exercises to really drive the points home. Coupled with the feedback from testing it provides a good starting platform. There was nothing in the course that wasn't covered by both *Java - A Beginner's Guide* and *Learning Java* combined, so if you already have read those books all you'll get is a reinforcement via exercises of what you've already learned.

# Areas Needing Further Study

I think the three main topics/technologies I want to get a better handle on are: concurrency, Mockito, and annotations. 

# Next Steps

I'm going to create some small programmes that make use of external libraries to get used to working with other people's code. Once I've used other libraries, I think I'll try looking into their source code to better understand what they're doing. I have read that programmers spend much more time reading code than writing, so I figure the best thing to do is learn by reading the source of other people's projects.

Somewhat related to Java, I watched a video where Brian Goetz said Clojure was his favourite language on the JVM after Java. After watching that I decided to have a look into Clojure for when I want a break from Java.