+++
title = "Low level system design — Parking Lot Design Part — I"
description = "On approaching the problem statement"
date = 2023-01-24T01:24:33+05:30

[taxonomies]
tags = ["oop", "object-oriented", "lld"]

[extra]
toc = true
+++

![](https://cdn-images-1.medium.com/max/2000/0*jpy7urdeHfT3JLsV)

In this article, we are going to focus on how to approach a given problem statement using the following steps.

- Understand the system requirements and constraints.

- Identify the objects in a system.

- Establish relationships between objects, and

- Implement based on Object-Oriented principles

## Understanding requirements:

Let’s take a look at the system requirements for creating a Parking lot system.

    - The parking lot should have multiple floors where customers can park their cars.
    - The parking lot should have multiple entry and exit points.
    - Customers can collect a parking ticket from the entry points and can pay the parking fee at the exit points to the parking attendant or automated exit panel
    - Customers can pay via both cash and credit cards.
    - The system should not allow more vehicles than the maximum capacity of the parking lot. If the parking is full, the system should be able to show a message at the entrance panel and on the parking display board on the ground floor.
    - Each parking floor will have many parking spots. The system should support multiple types of parking spots such as Compact, Large, Disabled, Motorcycle, etc.
    - The system should support parking for different types of vehicles like car, truck, van, motorcycle, etc.
    - Each parking floor should have a display board showing any free parking spot for each spot type.
    - The system should support a per-hour parking fee model. For example, customers have to pay some amount based on the Vehicle type.
    - Admins should be able to add parking floors and parking spot.

## How to approach the problem statement ?

The next step is to convert those requirements into design diagrams to get a better understanding of the system we are going to develop. A [UML](https://www.uml-diagrams.org/) diagram is a way of visualizing a software program using a collection of diagrams.

- **Top-down approach —** The major focus is on breaking the bigger problem into smaller ones and then repeating the process with each problem.

- **Bottom-up approach —** Primarily focuses on identifying and resolving the smallest problems and then integrating them together to solve the bigger problem.

## Top-down approach

In this phase, we are converting the problem statement into smaller components without going into nitty-gritty details. Along with [behavior diagrams](https://www.uml-diagrams.org/uml-25-diagrams.html#behavior-diagram), like sequence diagram helps in understanding the overall functionality of a system in its initial stages.

![](https://cdn-images-1.medium.com/max/4288/1*SL8PfJe5gYBvHncVMb1eCA.png)

![1. High level architecture and 2. Sequence diagram](https://cdn-images-1.medium.com/max/2154/1*hs365Ct3XfMnklPhayZM0A.png)

## Bottom-up approach

When it comes to implementation, it is always recommended to follow a bottom-up approach, in which [structure diagrams](https://www.uml-diagrams.org/uml-25-diagrams.html#structure-diagram) like class diagrams come in handy.

### Parking spot

![Parking spot — class diagram](https://cdn-images-1.medium.com/max/2154/1*bGWduZd4dmXEVRhmaoKLYQ.png)

- The parking lot should support multiple types of parking spots, such as compact, large, disabled, and motorcycle spots.

- The system should support parking for different types of vehicles, such as cars, trucks, vans, and motorcycles.

### Parking floor

![Parking floor — class diagram](https://cdn-images-1.medium.com/max/2154/1*MuH6pJZk41_732Yr0KprZw.png)

- Parking floor should have access to Parking spot and each floor should have access to Display board.

### Parking lot

![Parking lot — class diagram](https://cdn-images-1.medium.com/max/2154/1*lqE5GY8net8gVShq5zfFNw.png)

- Parking lot should contain list of Exit panel, Entry panel and Parking floor available in the system.

- Entry panel and Exit panel makes use of the Parking Ticket service.

### Overall Architecture

![Overall class diagram](https://cdn-images-1.medium.com/max/2154/1*iC4OeF2GBh2B8Zu8uS6ClA.png)

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/low-level-system-design-parking-lot-design-part-i-7567d510da1d)

🌟 🌟 🌟 **The source code for this blog post can be found here** 🌟🌟🌟

[GitHub - madhank93/learn-system-design](https://github.com/madhank93/learn-system-design)

</center>

### References:

[1] [https://github.com/tssovi/grokking-the-object-oriented-design-interview](https://github.com/tssovi/grokking-the-object-oriented-design-interview)

[2] [https://www.youtube.com/watch?v=nnpT0WXifLk](https://www.youtube.com/watch?v=nnpT0WXifLk)

[3] [https://www.youtube.com/watch?v=tVRyb4HaHgw](https://www.youtube.com/watch?v=tVRyb4HaHgw)

[4] [https://www.youtube.com/watch?v=7IX84K9g23U](https://www.youtube.com/watch?v=7IX84K9g23U)

[5] [https://www.youtube.com/watch?v=DSGsa0pu8-k](https://www.youtube.com/watch?v=DSGsa0pu8-k)
