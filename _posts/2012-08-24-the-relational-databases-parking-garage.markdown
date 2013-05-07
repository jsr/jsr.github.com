---
author: jsr
comments: true
date: 2012-08-24 06:13:38
layout: post
slug: the-relational-databases-parking-garage
title: The relational database's parking garage
wordpress_id: 126
categories:
- NoSQL
---

So we're building a parking garage and we assign the task to our friend the database architect. Only, he builds a parking garage unlike anything we've ever seen before.

When people pull their car into the garage, it is disassembled into all of its component parts and labeled with the id of the car. There's a bin of spark plugs, and a bin of license plates, a bin a steerings wheels, ... .

When someone comes and asks for their car by its license plate number, we look through the bin of license plates until we find the one they asked for. Affixed to the license plate is the id that was assigned to the car. Now we can go through all of the other bins and collect all of the rest of the parts, re-assemble the car, and deliver it to the driver.

This is what a relational database does to our data.

Sometimes a car is just a car. I can put cars in my parking garage just fine. And yes, I can still find your car by its license plate number just fine. In fact, if I keep a sheet of paper listing all of the license plates (e.g. an index) and which spot the car is in, i can find your car just as fast, if not faster, than when it was in the bin.
