# Architecture
I want to briefly talk about architecture here. This will set an understanding as to what we're trying to achieve with the various documentation and code bases we will explore.

It's important to remember that the goal of this book is to eventually discuss various aspects of architecture and which kind of designs you can adopt to deploy and deliver your products and services. However that being said, at this point in time we will cover a single architecture for a monolithic web application.

## The Monolithic Application
A monolithic application is a large, all-in-one code base made up of tightly coupled components that cannot work without each other. In such software projects it can often be difficult to separate components or replace them, but this doesn't always hold true. A well designed and maintained monolithic code base can, and often will, perform very well and be quite maintainable well into its life cycle.

The monolithic software design is also extremely common at the time of writing. In fact many experienced and well respected software engineers will suggest starting out with a large, single application before breaking it up into a micro service architecture.

## The Real World
In order to demonstrate the knowledge and tools being discussed throughout this book, we will look at deploying a real world application using everything we learn.

At this point in time, no application has been decided upon, but a few suggestions have been made, including:

* Wordpress;
* Magento;
* And mirroring Wikipedia;

I will eventually make a decision and run with it.
