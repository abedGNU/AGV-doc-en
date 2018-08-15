
*******************
AgvManager
*******************

Overview
=========

:ref:`figAgvManager` have 2 software components: AGV manager it self and AGV configurator.
In AGV configurator are set some parameters like the map file directory, script directory, communication with the AGVs controllers, PLC communication and IO definition, database communication, emulator enabling, etc.

AGV manager will load the parameters set by Agv configurator, and main execute the script written for the specified plant. In AGV manager can be shown the map and motion simulation and modify the script. The script is written in XScript language (Robox scripting language) and executed by AGV manager.

.. _figAgvManager:
.. figure:: images/agvmanager/agvManager.gif
    :align: center
    :name:
    :figwidth: 600px

    AgvManager

    AGV Manager main window

Installation
=============

The installation of AgvManger is straightforward, like any program in Microsoft Windows. AgvConfigurator is installed automatically with AgvManager.

In order to get the report from AgvManager a database should be installed. You can install MySql community version.

Once installed, a schortcut to AgvManager and AgvConfigurator have to be done. In the properties of the application, in ``Start in`` put the location of the folder where your projects are located, :ref:`figagvappproperties`.

.. _figagvappproperties:
.. figure:: images/agvmanager/agvappproperties.png
    :align: center
    :name:
    :figwidth: 400px

    Application properties

    Change the ``start in`` field to your projects directory

AGV configurator
=================

AGV configurator is a standalone program that create a ``.fdoc`` configuration file for AGV manager. One folder can contain more than one configuration file.

Project folder should be placed in the directory specified in the field ``Start in`` of the application properties, otherwise it will not work. By default the start point of the application is its installation directory.

The script file should be in the first level of the project folder, it can't be placed in subdirectories, for example ``appStartDir/Project01/scripts/main.xs`` is not allowed, it should be ``appStartDir/Project01/main.xs``.

The following animation illustrate how to create an :ref:`fignewProjectAgv`:

.. _fignewProjectAgv:
.. figure:: images/agvmanager/newProjectAgv.gif
    :align: center
    :name:
    :figwidth: 600px

    AGV new project

    AgvConfigurator, creating new project

In AGV configurator, General tab, the ``.xs`` script file is selected. The script have to be created by an external editor. It is enough to give any name to the executable file with ``.xb`` extension. When AgvManager compile the ``xs`` file, the ``xb`` file will be created automatically.

.. _figConfigScript:
.. figure:: images/agvmanager/agvConfigGeneral1.png
    :align: center
    :name:
    :figwidth: 400px

    General tab

In AGV configurator, Map tab, the ``.map`` file is selected. In order to view the user points, you have to set the width and height of it, otherwise user points are not viewed. It is convenient to represent user points as rectangles, not squares. If you need to rotate the map you can change ``line 1 angle``.

.. _figConfigMap:
.. figure:: images/agvmanager/agvConfigMap.png
    :align: center
    :name:
    :figwidth: 400px

    Map tab

AGV configurator. AGV tab, where communication parameters with the AGVs are set. Note that ``ip adresses`` are sequential.

.. _figConfigAgv:
.. figure:: images/agvmanager/agvConfigAgv.png
    :align: center
    :name:
    :figwidth: 400px

    AGV tab

AgvManager interface
====================

AgvManager have one menu bar, one :ref:`figtoolbar`, one status bar, map visualization and different tabs [Fx].

In the :ref:`figtoolbar`, we can find the button: Vehicle status, Commands insertion, Access level, color type, Zoom, Agv emulation, user define forms.

.. _figtoolbar:
.. figure:: images/agvmanager/toolbar.png
    :align: center
    :name:
    :figwidth: 400px

    tool bar

In the status bar we can see some message from the script, date and time, and current access level.

AGV emulation
---------------
If the flag ``emuagv.dll Vehicles emulation`` in AgvConfigurator is checked, we can emulate the AGV in AgvManager. The windows of the emulation can be opened via the button ``agv emulation control``.

The :ref:`figAgvEmu` shows the status of the Agv, groupbox ``Status``, where are viewed the 32 Vehicle Status flags (XVehicleInfo.uStatus). The first 4 flags are written by the vehicle controller and the others can be defined by user in the vehicle controller.

.. _figAgvEmu:
.. figure:: images/agvmanager/agvEmulation.png
    :align: center
    :name:
    :figwidth: 400px

    Emulation window

    We can see Agv status flags and other informations. The progress bar indicate the consumed power of the battery, not the remaining power, e.g. 100\% is battery empty.

The first 4 bits (flags) are:
	- Power enabled
	- Execution command
	- Charging battery in progress
	- Load present, or Unit load present

These flags correspond to:

.. code-block:: none
  :caption: Vehicle status fags
  :name:

  // Vehicle status flags, bit mask 2^n.
  // 4 least significant bits
  $define VST_POTENZA_ATTIVA   1 // Power active, mask bit 0
  $define VST_EXEC_COMANDO     2 // executing command, mask bit 1
  $define VST_CARICA_INCORSO   4 // charge in progress, mask bit 2
  $define VST_CARICO_PRESENTE  8 // load present, mask bit 3

The Battery box, indicate the amount of power consumed, not the remaining one. For example if the status is 100%, this mean the battery is empty, if the progress bar indicate 20%, this mean the remaining power is 80%. The value of the remaining power is shown in the ``Battery capacity`` progress bar in the tab ``Vehicle informations [F3]``.

To emulate the Agv, first the Agv emulation should be active, state shown in tab ``Vehicles`` in the window ``Agv emulation control``. Then in the ``Vehicle x`` tab, where ``x`` is the AGV number, the operating mode can be selected, and the status can be emulated. The Agv should be in automatic and power is enabled in order to move the Agv.
For example, if we set the flag ``Load present`` to one, the agv behave depending on the script logic. If the load is a loading unit and there is a load in some station to take out, the Agv will go to that station. Or for example if the load is properly the final product the agv may transport it to some unloading station.


Point windows property
-------------------------

User points can be viewed as rectangles, where the dimensions are set in AgvConfigurator. A ``CBat`` (Battery point) is shown as a battery icon. On a double click on a user point or battery point a window is opened, see fig.:ref:`figLocStorageInfo`.

.. _figLocStorageInfo:
.. figure:: images/agvmanager/locStorageInfo.png
    :align: center
    :name:
    :figwidth: 400px

    Storage informations tab

    Storage informations: Load information are shown, timestamp of loading, load type

In the :ref:`figLocStorageInfo`, changing the type we can see the description associated to it, e.g. Type 1 is an empty ``loading unit``. The spin box (numeric updown control) is used to show only the type, the type is a combination of the checkboxes.

Properties assigned by the function ``AddIntProperty()``, are shown in the :ref:`figLocationProperties`.

.. _figLocationProperties:
.. figure:: images/agvmanager/locProperty.png
    :align: center
    :name:
    :figwidth: 400px

    Properties tab

    Location window: Storage information and properties

Terminology
---------------

- AGV
- Vehicle
- Loading unit
- Load
- Path planning
- Route
- RDE
- RAT
- Controller
- Mission
- Emulation
- Operation
- Event
- Callback
- Debugging
