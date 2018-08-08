
Overview
**********

What is an AGV
==============

Applications
============
Robox AGV's controller use three kinds of guidance:
  - Laser
  - Inertial
  - Magnetic tape

It allow OEM companies to implement Robox AGV's software easely.

.. figure:: images/agv/agv1.jpg
    :align: center
    :figwidth: 600px

.. figure:: images/agv/dep1_FEIN.jpeg
    :align: center
    :figwidth: 300px


.. figure:: images/agv/agv3.jpg
    :align: center
    :figwidth: 300px

.. figure:: images/agv/trastemuletto.jpg
    :align: center
    :figwidth: 300px

.. figure:: images/agv/nav_877E1.jpeg
    :align: center
    :figwidth: 300px

.. figure:: images/agv/nav_877E1.jpeg
    :align: center
    :figwidth: 300px

AGV Robox softwares and tools
==============================

Robox's AGVs are commisioned through 3 softwares: RAT, AgvManager and RDE.

Maps are designed using RAT software then converted to an ASCII file with \textit{.map} extension. This file is used by the other two softwares: AGV manager and RDE.

AgvManager has two main parts: script and kernel. The specific logic of a plant is written in a script using XScript language, and given to the kernel that execute it. The kernel handle also the communication with the motion controller and eventually a plant PLC or database. AgvManager handle automatically path planning, dispatching and anticollision.

RDE is Robox IDE for motion control programming. The motion control part is written with R3 languagged in RDE. The kinematic and the motion control is already wirtten by Robox. The map is compiled by ICMap and read by the RTE.

.. figure:: images/agvDiag.png
    :align: center
    :figwidth: 500px

    AGV block diagram. RAT give a .map file as output. The map is read by AGV manager. The map is comiled by ICMap then read by RTE. AGV manager may communicate with a PLC or a database as an interface to the plant IO, and with RTE for motion control.

In the following chapters we will illustrate Robox software in order to draw a map and assign missions to Agv. Three softwares are needed : RAT, AgvConfigurator and AgvManager. Note that maps can also be edited by a text editor. AgvConfigurator is a part of AgvManager and are installed togethers.
