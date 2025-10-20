+++
title = "Low level system design — Parking Lot Design Part — II"
description = "Code implementation for the Parking lot system design"
date = 2023-01-24T01:25:33+05:30

[taxonomies]
tags = ["oop", "object-oriented", "lld"]

[extra]
toc = true
+++

![](https://cdn-images-1.medium.com/max/2000/0*jpy7urdeHfT3JLsV)

In continuation of my [previous article](https://medium.com/@madhankumaravelu93/low-level-system-design-parking-lot-design-part-i-7567d510da1d), in this post we will look at the code implementation of the Parking lot system, which was solved using [Typescript](https://www.typescriptlang.org/) programming language, and following [Test Driven Development (TDD)](https://en.wikipedia.org/wiki/Test-driven_development) using [Vitest](https://vitest.dev/) unit test framework.

### Vehicle:

![](https://cdn-images-1.medium.com/max/2000/1*OIoTK7O8kbap325ItNroYg.png)

![](https://cdn-images-1.medium.com/max/2000/1*n5ma2uHDrUJHk74Hp24V7w.png)

![Vehicle spot implementation](https://cdn-images-1.medium.com/max/2000/1*fgApaaeWq1SK7DAsQD7AzA.png)

- **Vehicle** class contains all the common behaviours which are applicable to other vehicles like in this example **MotorCycle** and **Car**

### _Parking spot:_

![](https://cdn-images-1.medium.com/max/2000/1*5t8syCb8Kn_3YdLVIqo4LA.png)

![](https://cdn-images-1.medium.com/max/2000/1*etFCNvcCcmTZ04WAXtfZIw.png)

![Parking spot implementation](https://cdn-images-1.medium.com/max/2000/1*oLG4lKNhcTNBmYzbK7AajQ.png)

- **ParkingSpot** class contains all the common behaviours which are applicable to other Spots like in this example **CarSpot** and **MotorcycleSpot**

### Parking floor:

![Parking floor implementation](https://cdn-images-1.medium.com/max/2066/1*G09eFX0yvxnmsedzHE92gg.png)

- **ParkingFloor** class has the access to parking spots and display board.

- The **getSpotTypeForVehicle** private method determines which ParkingSpot should be mapped to a given vehicle.

### Parking lot:

![Parking lot implementation](https://cdn-images-1.medium.com/max/2000/1*i__QaXZY2NaU5PyEDE0Bdg.png)

- The **ParkingLot** class serves as the system's entry point.

- The **ParkingLot** class is declared using the singleton pattern, which limits class initialization to ensure that only one instance of the class can be created.

<div align="center">* * * *</div>

<center>

Originally published on [Medium](https://medium.com/@madhankumaravelu93/low-level-system-design-parking-lot-design-part-ii-ab5f4efab90)

🌟 🌟 🌟 **The source code for this blog post can be found here** 🌟🌟🌟

[GitHub - madhank93/learn-system-design](https://github.com/madhank93/learn-system-design)

</center>

### References:

[1] [https://github.com/tssovi/grokking-the-object-oriented-design-interview](https://github.com/tssovi/grokking-the-object-oriented-design-interview)

[2] [https://www.youtube.com/watch?v=nnpT0WXifLk](https://www.youtube.com/watch?v=nnpT0WXifLk)

[3] [https://www.youtube.com/watch?v=tVRyb4HaHgw](https://www.youtube.com/watch?v=tVRyb4HaHgw)

[4] [https://www.youtube.com/watch?v=7IX84K9g23U](https://www.youtube.com/watch?v=7IX84K9g23U)

[5] [https://www.youtube.com/watch?v=DSGsa0pu8-k](https://www.youtube.com/watch?v=DSGsa0pu8-k)
