---
layout: post
title: Study Summary - 2
date: 11/07/2022
categories: Programming
tags: Java
---

This month I decided to try and make some small projects to put what I've read to practise. I also picked a project to look at the source code and try and understand what was going on under the hood.

# Project One - Atmospheric Measurements

Having been used to data handling in R and Python, I somewhat naturally went for an open dataset to try programming with. I though that it could make an interesting project, learning to download from the web, work with files, to interface with a database, and to provide the data via Kafka.

When I started this I tried to use TDD, and for the first implementation I made some tests and then wrote the code. This was satisfying and it made sense, however this was when I had manually downloaded files and had the CSV loader class as a collection of static members. I then spent a month not coding and when I came back to it I then decided to start refactoring the project: first to download the CSV files through code, and second to make an instance of the CSV loader. In doing this I got carried away with just getting in and coding, and forgot to both make tests first and to commit changes liberally. In retrospect I'm not sure how I would have tested a lot of the functionality. For example, how does one test the downloading aspects? I assume with Mockito, but I wasn't sure, and I'd pressed on with the changes so by the time I thought about testing I'd already finished.

I used Log4j2 here, initially I created the loggers and set each logging level in the class the logger was created in just to get up and running quickly. However this was unsatisfactory as it missed the point of being able to control loggers without having to edit every class they're used in. Reading through the documentation was dry, but I got the XML configuration setup working.

I used a PostgreSQL database to store the data once downloaded, and I also added functionality to send the measurements to a Kafka instance. The Kafka functionality is due to wanting to be able to process the observations as a stream of values in a program like Apache Flink or Spark.

The biggest takeaway I had from this project was how useful Maven is. I needed to access PostgreSQL, so I just added that to the pom file and that's it, I'm using it. I needed a logger, copy paste to the pom and now I'm logging. It's so quick and easy. Plus the newest version of IntelliJ IDEA makes writing code so smooth, it's so much better than the RStudio or the Python notebooks I am used to.

# Project Two - Movies

Initially changed the Movie class to a record as that seemed to be the best choice, each movie is an immutable object. However this wouldn't work with Hibernate, so I then found Project Lombok, which make the declaration of a class similar in simplicity to a record, but allowing for a no-arg constructor like Hibernate requires. Lombok was really easy to use, and once I allowed annotation processing in the IDE I could access the getters in lambda expressions and method reference calls. I still need to setup the use of Hibernate to persist the entries.

# Reading Code - PgJDBC

As I have a background in databases I thought this would be a good project. I spent a day or two going through it, and my main takeaway is that it is very difficult reading others' code. It wasn't that any particular part was difficult, it was more the sheer volume of information to understand was too great. My next approach will be to create a project using as much functionality of the library as possible, and then digging into the specific source code of what I've used to understand how that part in isolation works.

# Summary

I did some coding, but I also took around three weeks off. Now that I know the basics of Java syntax I thought I would take on data structures and algorithms. However before I start that I want to improve my maths skills, especially with discrete maths.
