---
title: 'Robotics I: ROS in a Nutshell'
description: >-
  As I am working in the field of robotics, and also it is one of my most
  passionate subjects, I decided to start a new series of articles on…
date: '2022-01-27T11:43:16.318Z'
categories: []
keywords: []
slug: /@erdemuysalx/robotics-i-ros-in-a-nutshell-c36c27acebe7
---

As I am working in the field of robotics, and also it is one of my most passionate subjects, I decided to start a new series of articles on robotics. This series will relatively focus on robotics software and control rather than the hardware side. I thought writing an article as an overview of ROS would be the most appropriate way of starting this new robotics series, even though there are lots of resources on ROS. Content of this article going to be as follows:

*   Overview of ROS
*   ROS Concepts and Design Patterns
*   ROS vs ROS2

![](https://cdn-images-1.medium.com/max/800/1*9qtOzaq_LiIJ33lqaw5X8g.jpeg)

### Overview of ROS

What exactly ROS is? Well, it is not actually something like it sounds. Although it gives some utility functions like a general-purpose operating system, such as abstraction from the hardware layer, low-level hardware control, interprocess message-passing, package management, it is not a real operating system, but a middleware suite. ROS as a middleware suite offers developers a collection of different software frameworks, libraries, and helper tools in order to develop robust robotics applications with heterogeneous hardware & software clusters.

### ROS Concepts and Design Patterns

#### Nodes[​](https://husarion.com/tutorials/ros-tutorials/1-ros-introduction#nodes "https://husarion.com/tutorials/ros-tutorials/1-ros-introduction#nodes")

Atomic building blocks in ROS are called nodes. Nodes are responsible for handling devices and sensors control or computing algorithms, ideally each node for a separate task. These tasks can be either low-level or high-level processing purposes. Nodes can communicate with each other through topics or services. There are two different types of nodes which are publishers and subscribers. As their names refer to while publishers are publishing messages to the topics, subscribers are subscribing to a topic for getting messages from there. ROS structure is based on distributed software packages.

#### Topics[​](https://husarion.com/tutorials/ros-tutorials/1-ros-introduction#topics "https://husarion.com/tutorials/ros-tutorials/1-ros-introduction#topics")

Internode communication can be done using topics. The topic is a concept for describing the data stream used to exchange information between different nodes. Topics are used to send a stream of messages with one type. This could be either a sensor’s data information or a camera image. Nodes publish messages to the topics or subscribe to them in order to get messages. The number of different nodes publishing or subscribing to a topic is limited by system resources, RAM in particular. Every topic has a unique name with a defined message type. And the relationship between nodes and topics is managed by the ROS master in ROS1 and by DDS in ROS2, which is a kind of management middleware in ROS.

#### Services[​](https://husarion.com/tutorials/ros-tutorials/1-ros-introduction#services "https://husarion.com/tutorials/ros-tutorials/1-ros-introduction#services")

An alternative communication method between nodes is services that resemble the server-client architecture model, a form of remote procedure call (RPC). A client sends a request, and then a server sends back a response to the request. here both client and server are actually ROS nodes. Thus it is bidirectional communication however unlike topics, services are one-time communication. Services are generally used for invoking an action.

#### **Actions**

In order to invoke services but for time extended tasks, we have a concept called actions. A ROS action can be defined by three basic ROS messages which are; goal, result, and feedback. While the goal is representing the state which is aimed by action and the result message represents the actual output after performing the action. And the feedback message is a message that is sent periodically for tracking the progress of the task.

#### Parameters

In ROS, the robot’s environment and runtime variables are kept in the ROS parameter server. Parameters are comprised of key-value pairs like dictionary data structures. As they are not designed for high-performance, it is best used for configuration parameters. such as static, non-binary data.

#### ROS Package

A single package is usually developed for performing one specific task for instance control, navigation, etc., and can contain one or multiple nodes, preferably messages.

### ROS vs ROS2

In recent years, intensive work has been carried out in the OSRF’s ROS community to make ROS more durable and real-time sensitive. Today, these studies have started to bear fruit and the new generation ROS has begun to be distributed under the name ROS2. The main difference of ROS2 from its ancestor ROS is that the underlying publisher/subscriber communication architecture of the system is taken from the TCP/UDP multicast-based ROS master and completely replaced with DDS (data distribution service). The DDS offers really powerful functions and strengthens ROS for designing production-grade real-time robotics systems. Additionally, ROS2 can be used with more operating systems like Windows, Mac OS, and RTOS.

![](https://cdn-images-1.medium.com/max/800/1*S2Aeb4GlN-Y7QKMcU1t5Bg.png)

#### ROS Computation Graph

Let’s assume that we are designing a robot that follows a particular object using its onboard sensors. This robot then needs a camera device to see the object, a perceive system to process the camera images to figure out where the object is, a control system to decide what direction to move, and some motors to move the wheels to allow it to move toward the object. Using ROS we might construct the system like so:

![](https://cdn-images-1.medium.com/max/800/1*PnKf4vFYdGXq8h0VDFr-cQ.png)

Each subsystem can be thought as a ROS node. Therefore we will have four nodes, i.e. for executable pieces of codes. One is the camera driver which fetches raw images from the camera device, then published these raw images to the relevant topic. From there, the object detector node subscribes to the topic in order to process frames and extract a piece of useful information about the object to be followed. Let’s say it is examining the location of the object and publishing this information to the relevant topic with a sensible message type. Then target follower node, which calculates the what command should be applied to the motors in order to follow the object, subscribe to object location topic and using that information it produces some information and publishes that to the relevant topic. Finally, the motor driver node, which is a low-level motor control block, gets this information and steers the wheels.

We have come to the end of the ROS in a Nutshell, the first article of the robotics article series. I am planning to write about the ROS file system and the ROS navigation stack in my next posts. See you later.