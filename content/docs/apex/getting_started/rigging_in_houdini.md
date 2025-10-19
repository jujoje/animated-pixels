---
date: '2025-10-18T17:16:10+11:00'
title: 'Rigging in Houdini'
weight: 1
---
Rigging in Houdini consists of two overlapping systems:

- **KineFX**: KineFX uses geometry to define a skeleton and is used as the basis for the SOP based rigging tools, motion retargeting and crowd agent setup.
- **APEX**: APEX runs on its own execution engine, and is focuses on providing a procedural system for building rigs, tied to an animator friendly viewport state that efficient evaluates the rig. 

SideFX often use the two terms interchangeably and the two systems are fundamentally interrelated;  

## Skeletons and KineFX
KineFX rigs consist of points and polygon lines. The point contain the joint transform matrices and attributes (transform, local transform, scale, joint name), while the line defines the hierarchy of the skeleton. 

One of the nice things about skeletons just being points with a transform matrix is that all of Houdini usual geometry tools can be use to manipulate them, for example creating a skeleton based off a curve, or procedurally adding joints using a resample, or constraining joints to a surface using a `primuv` lookup. 

## APEX
While APEX is most often associated with rigging, it stands for *All Purpose EXecution* framework, and is also used for Houdini's texturing and compositing framework Copernicus etc. It is an execution framework that runs outside of houdini, with the emphasis on speed. 

For rigging purposes, it provides a fast environment for executing the rig and deforming geometry, tied closely the the animation state.  According to SideFX, the main advantages of APEX are that it offers:

- Flexibility by storing the entire graph as geometry.
- Delayed invocation - evaluate just what you need when you need it, as well as partial evaluation.
- Self healing - you can change the graph at any time
- Compiled on the fly - performance orientated
- Designed for live, real-time procedurals in the viewport.
- Designed with traditional and ephemeral rigging in mind.