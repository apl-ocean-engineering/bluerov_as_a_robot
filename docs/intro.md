---
sidebar_position: 1
---

# Introduction

We wrote this guide to share our experiences using the BlueROV as a robot, specifically reading sensor and state information, including video, from the ROV, and sending commands to the ROV.   We focus on the core "off the shelf" BlueROV / BlueOS experience; obviously there are lots of ways to extend and customize your ROV but we wanted to simplify getting started with the core vehicle.

This guide is not specifically tied to a particular software framework or project, and is more of a FAQ than a complete software stack.   Throughout we use examples from ours and others code.  If you are looking for a more complete robotic solution, please see our related [awesome_blueos]() repo.

We are working with the BlueROV; many of these lessons also apply to the BlueBoat, but we do not have a BlueBoat for testing.  Please feel free to [contribute content](contributing/) related to the BlueBoat.

We developed this guide working with BlueOS 1.4.x and Ardupilot 4.5.7.   The BlueOS ecosystem continues to evolve and we will update the guide as we discover new things.  Please [contribute](contributing/) if you discover new BlueOS capabilities.

## Structure

This guide has four sections:

* [BlueOS Basics]()
* A detailed discussion of working with [Mavlink]().   This describes interaction with the robot using [`pymavlink`](https://github.com/ArduPilot/pymavlink).
* A section on using [ROS2]() to interact with the ROV, largely through [`mavros`](https://github.com/mavlink/mavros)
* 

## Related Projects




# License and Attribution
