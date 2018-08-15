
Demo 02: Drag and drop example with loading and loading operations
*******************************************************************

In this example we will see how we can perform a drag and drop to a user point. A user point represent a working station, that could be machine or simply a position in a store. For example in an automatic store, a user point may represent the position where materials can be stoked or picked. A user point have a property called ``bPresenza`` that indicate the presence of material in the position designated by the user point or its absence.

:download:`Here is the complete project <listing/demo_drag_drop/02_agvDemo.7z>`

OnAgvDroppedToPoint()
---------------------

In the function ``OnAgvDroppedToPoint()``, after the verification of requirements, we will register 3 missions depending on the case if the point is a user point or generic point, if the agv have a load or the user point have a load. In listing :ref:`lstDrag` the code and explanations are shown.

The code that verify the conditions: vehicle exist, in automatic, not enabled, no mission in progress is not shown here. I can be find in the complete example.

Listing :ref:`lstDrag` can be found in the file ``agvEventFucntions.xs`` in the callback function ``OnAgvDroppedToPoint()``.

.. code-block:: none
  :caption: Drag and drop to user point and generic point
  :name: lstDrag

  XSiteInfo sInfo ; user point information strucutre
  XVehicleInfo vInfo ; vehicle strucutre information

  ; if user point and vehicle exist
  if (AgvGetSiteInfo(uUser, @sInfo) and AgvGetVehicleInfo(uAgv, @vInfo))

  bool loadOnAgv, loadOnUser
  ; read the bit corresponding to lpad present on agv
  loadOnAgv = (vInfo.uStatus & VST_CARICO_PRESENTE)

  loadOnUser = sInfo.bPresenza

  ; if both agv and user point have a load
  if (loadOnAgv && loadOnUser)
    MessageBox("Cannot move agv " + (uAgv + 1) + " to " + GetSiteName(uUser) + " : both have a trolley")
    return
  endif

  ; if only agv have a load, the mission unload to user is registered
  if (loadOnAgv && not loadOnUser)
    ; call to use defined function
    RegisterMission(uAgv, MIS_UNLOAD_ONLY, uUser)
    return
  endif

  ; if only user point have load, the mission load to agv is registerd.
  if (loadOnUser && not loadOnAgv)
    RegisterMission(uAgv, MIS_LOAD_ONLY, uUser)
    return
  endif

  endif

  ; register movement to point,
  ; if there is no lad neither on agv neither on user point,
  ; or if the point is a generic point
  RegisterMission(uAgv, MIS_TO_POINT, uUser)

When the user drag and drop the vehicle onto a point, the callback function ``OnAgvDroppedToPoint()`` is called, then the function ``RegisterMission()`` is called inside it as we can in the listing :ref:`lstDrag`.

RegisterMission()
------------------
The function ``RegisterMission()`` is a user defined function, with the goal to assign missions, can be found in ``common.xs``.

The keyword ``forward`` is used to define a prototype function, it tell the program that somewhere the function is implemented. if ``forward`` is not used, and we implement for example a ``functionA`` before a ``fucntionB``, and ``functionA`` call ``fucntionB``, the program will give error, because he expect that ``fucntionB`` is implemented before ``functionA``.

The function have 4 input parameters: ``uAgv`` (agv code), ``uCode`` (mission id), ``iPar1`` and ``iPar2``. Where in this case in ``iPar1`` is passed the point id.

Constants to identify missions and macros are defined as follow:

.. code-block:: none
  :caption: Mission defition
  :name:

  // Mission defition. Missions can begin from 0,
  //because there are no missions already defined in AgvManger

  // No mission in progress
  $define MIS_NULL							0

  $define MIS_LOAD_ONLY						10
  $define MIS_UNLOAD_ONLY						11
  $define MIS_TO_POINT						14

  // MACRO definition, begin always from 100
  // Movement to waypoint
  $define MAC_MOVE_TO_WP					100
  // Load from the point defined by par1
  $define MAC_LOAD_TROLLEY				102
  // Unload on the point defined by par1
  $define MAC_UNLOAD_TROLLEY				103

In ``RegisterMission()`` we will start a new mission and fill the **macro list** with MACROs. We will use respectively ``agvStartMission()`` and ``agvAddMacro()``.

As you can notice, a mission is started by calling ``agvStartMission(uint agvId, uint missionId, string missionDescription)``. This function return ``true`` if a mission is in progress. We can define a new function that return a string value, to get the description of missions.
After that we write a select case statement in order to fill the macro list depending on the mission code and to give movement instructions by calling the user defined function ``RegisterMovement()``.

For example, if our mission is ``MIS_LOAD_ONLY``  we register a movement to the user point by calling ``RegisterMovement(agvId,userPointId)``, where we will add the macro ``MAC_MOVE_TO_WP``, then we add the 2 macros : ``MAC_LOAD_TROLLEY`` and ``MAC_END``. So the macro list have 3 macros, table tabmacrolist_. This should be clear, the vehicle first move to the user point, once arrived, load the agv then finish executing the mission.

Macro list of the load mission, ``MIS_LOAD_ONLY``. As you can see the paramters can assume different value types depending on the macro or micro:

.. _tabmacrolist:

====  ==================  ===============  ===============  =====  ======
uAgv  MAC code            iPar1            iPar2            iPar3  iPar4
====  ==================  ===============  ===============  =====  ======
1     MAC\_MOVE\_TO\_WP   Waypoint id      concatenateNext
1     MAC\_LOAD\_TROLLEY  User point code  bVasiPieni
1     MAC\_END            MIS\_LOAD\_ONLY
====  ==================  ===============  ===============  =====  ======

The same reasoning can be applied for other missions. Following the a part of the code:

.. code-block:: none
  :caption: RegisterMission() code fragment
  :name: lstRegisterMission

  // starting mission "uCode", with descrition "text"
  if (not AgvStartMission(uAgv, uCode, text))
    return MIS_NULL
  end
  // user point info strutcture
  XSiteInfo sInfo

  //Fill the macro list with the macro for the selected mission
  // when we call registerMission(), we pass as iPar1 the user point index
  select (uCode)
    // Loading agv mission
    case MIS_LOAD_ONLY
      if (not AgvGetSiteInfo(iPar1, @sInfo))
        // Strange error. Should not happen!!!
        AgvStopMission(uAgv)
        return MIS_NULL
      endif
      // iPar1 = point in store where toilet must be taken
      RegisterMovement(uAgv, iPar1)
      // Take the trolley with the toilet
      // Trolley with toilet
      AgvAddMacro(uAgv, MAC_LOAD_TROLLEY, iPar1, sInfo.bVasiPieni)
      // END of this mission
      AgvAddMacro(uAgv, MAC_END, uCode)
      break

    // unloading agv mission
    case MIS_UNLOAD_ONLY
      if (not AgvGetSiteInfo(iPar1, @sInfo))
        // Strange error. Should not happen!!!
        AgvStopMission(uAgv)
        return MIS_NULL
      endif
      // iPar1 = point in store where toilet must be taken
      RegisterMovement(uAgv, iPar1)

      // Leave the trolley with the toilet
      // Trolley with toilet
      AgvAddMacro(uAgv, MAC_UNLOAD_TROLLEY, iPar1, sInfo.bVasiPieni)
      // END of this mission
      AgvAddMacro(uAgv, MAC_END, uCode)
      break

    // movement to a point mission
    case MIS_TO_POINT
      //Move to selected point
      RegisterMovement(uAgv, iPar1)
      //END of this mission
      AgvAddMacro(uAgv, MAC_END, uCode)
      break

    // mission not defined
    default
      MessageBox("Mission not implemented: " + uCode)
      return MIS_NULL
  end

The function ``RegisterMovement()`` is self-explanatory.

.. code-block:: none
  :caption: RegisterMovement()
  :name: lstRegisterMovment

  code RegisterMovement(uint uAgv, uint userId, uchar destOrientation = 'X',
  			bool concatenateNext = true
  			)
  	uint wpidx
  	//add waypoint, return an unique id of the added point.
  	wpidx = AgvAddWaypoint(uAgv, userId, destOrientation)
  	// add movement macro related to the point we get previously
  	AgvAddMacro(uAgv, MAC_MOVE_TO_WP, wpidx, concatenateNext)
  end

In this case mission are registered by calling the user defined function ``RegisterMission()``. This function was called by the function ``OnAgvDroppedToPoint()``. If we want to assign missions in another way, we can call the function ``RegisterMission()`` inside the callback function ``onNextMission()`` that is called when the agv is enabled.

Independently on how a mission is registered, when a mission is started the callback function ``onExpandMacro()`` is called in order to begin the execution of macros and micors.

onExpandMacro()
----------------
``onExpandMacro()`` is called when there are MACROs in the macro list, check the flowchart in the official documentation and in the previous chapter, in the section mission execution.

To this callback function are passed the agv index, mission index, MACRO index, and 4 parameters. The agv index and mission index are passed from AgvManager to the function, that are related the the list to be expanded. Every mission have its own macro list. The parameters are read from the macro list.

.. code-block:: none
  :caption: onExpandMacro()
  :name: lstonExpandMacro

  code OnExpandMacro(uint uAgv, uint uMission, uint iMacroCode,
  		 int iPar1, int iPar2, int iPar3, int
  		 ) : bool

  	select (iMacroCode)
  		case MAC_MOVE_TO_WP
  			// iPar1 = Waypoint id
  			// iPar2 = (bool) do concatenate next macro
  			select (AgvMoveToWayPoint(uAgv, uMission, WpFl_RicalcolaPercorsi | WpFl_EliminaCompletato))
  				case MoveResult_CompletedMovement	; Completed movement
  				case MoveResult_WaypointReached		; Waypoint reached
  					if (iPar2)
  						AgvComputeNextMacro(uAgv)
  					endif
  					return true
  				default
  					return false
  			endselect
  			return true

  		case MAC_LOAD_TROLLEY
  			// par1 is the point
  			// par2 is true if there is a toilet on the trolley
  			// par3 is true it the trolley is ready to be taken out of store
  			AgvRegisterOperation(uAgv, uMission, O_LOAD, iPar2, iPar3, 0, 0, iPar1)
  			break

  		case MAC_UNLOAD_TROLLEY
  			// par1 is the point
  			AgvRegisterOperation(uAgv, uMission, O_UNLOAD, 0, 0, 0, 0, iPar1)
  			break

  		case MAC_END
  			SetAgvMessage(uAgv, "")
  			AgvRegisterSystemBloccante(uAgv, uMission, S_END)
  			break

  		default
  			qt_warning("Unknown macro: " + iMacroCode)
  			break
  	end
  	return TRUE
  end

Simply the macro load register on operation of type ``O_LOAD``, the unload macro register the ``O_UNLOAD`` operation and the end macro register the system macro ``S_END``. These three micros are already defined by AgvManager.
Search in the official documentation the prefixes ``O_`` ad ``S_`` to find a complete list Operations and System micro type.

The macro ``MAC_MOVE_TO_WP`` execute the movement command by calling ``AgvMoveToWayPoint(,,)`` and register a ``MIC_MOVE`` micro type. When the movement is completed or the waypoint is reached the function return true, that mean the expansion of the macro has finished.

After the expansion of macros, the micro are executed by calling the callback fucntion ``onExecuteMicro()``.

OnExecuteMicro()
-----------------

We see how micros are registered when macros are expanded. Now we see how micros are executed.
The ``MAC_LOAD_TROLLEY`` had registered an ``O_LOAD`` micro of type ``MIC_OPERATION``.
The ``MAC_UNLOAD_TROLLEY`` had registered an ``O_UNLOAD`` micro of type ``MIC_OPERATION`` and the macro ``MAC_END`` had registered an ``S_END`` of type ``MIC_SYSTEM``.

.. code-block:: none

  case MIC_OPERATION
  	select (iPar0)
  		case O_LOAD
  			if (bLastCall)
  				MultiMessageState(uAgv, "Agv " + (uAgv + 1) + " : loaded from " + userId)
  				SetAgvMessage(uAgv, "")
  				// Agv has finished the load:
  				// AgvExecLoad() puts the logical content of the user point identified by userId
  				// on the agv, and removes from the user point.
  				// NOTE: the operation was sent to the agv in OnExpandMacro()
  				// expanding the macro MAC_LOAD_TROLLEY
  				AgvExecLoad(uAgv, userId)
  				return true
  			else
  				MultiMessageState(uAgv, "Agv " + (uAgv + 1) + " : loading from " + userId)
  				SetAgvMessage(uAgv, "Loading")
  				return false
  			endif
  			break

  		case O_UNLOAD
  			if (bLastCall)
  				MultiMessageState(uAgv, "Agv " + (uAgv + 1) + " : unloaded to " + userId)
  				SetAgvMessage(uAgv, "")
  				AgvExecUnload(uAgv, userId)
  				return true
  			else
  				MultiMessageState(uAgv, "Agv " + (uAgv + 1) + " : unloading to " + userId)
  				SetAgvMessage(uAgv, "Unloading")
  				return false
  			endif
  			break
  		default
  			qt_warning("Unknown MIC_OPERATION : " + iPar0 + " (mission = " + iMission + ", par1 = " + iPar1 + ")")
  			break
  	end
  case MIC_SYSTEM
  	select (iPar0)
  		case S_NULL
  			// Micro of that type are generated by AgvManager, I am not intereseted on it.
  			break
  		case S_END
  			// End of mission
  			if (vInfo.uStatus & VST_EXEC_COMANDO)
  				MultiMessageState(uAgv, "Agv " + (uAgv + 1) + ": wait for agv commands finished")
  				return false
  			endif
  			MultiMessageState(uAgv, "Agv " + (uAgv + 1) + ": finished executing commands")
  			AgvStopMission(uAgv)
  			SetAgvMessage(uAgv, "")
  			break
  end

In the case of ``O_LOAD`` and ``O_UNLOAD``, AgvManager send associated commands to the vehicle. When the vehicle terminate the execution of the command associated to the micro, AgvManger call the callback function ``onExecuteMciro()`` with the parameter ``bLastCall`` is set to true. During the last call, when ``bLastCall=true`` we can perform also a logical load or unload by calling respectively ``AgvExecLoad()`` or ``AgvExecUnload()``.

The case of ``S_END``, the function ``AgvStopMission(uAgv)`` is called in order to stop terminate the mission execution.

The ``MIC_MOVE`` are handled by AgvManager not by the script. So is not necessary to write the case of micro movements.
