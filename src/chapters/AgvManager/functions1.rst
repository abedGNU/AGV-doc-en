
Fundamental functions and data structures
******************************************

In this chapter we will present some data structure of AgvManager and some examples on agv scripting. In this chapter we will show only piece of code necessary to explain the concepts.

In AgvManager's documentation we can find functions and data structures divided by argument, it means functions to manage vehicles, maps, databases, etc. Refer to the official documentation in order to get a complete list of functions and data structures.

An Agv usually transport a loading unit (UDC, LU) e.g. pallet, trolley, etc. , or a loading unit with a load on it e.g. a pallet with some mechanical parts on it.

For the presence Loading unit we find the variable bPresenza or bPres.
For the presense of a load on board of the Loading Unit (UDC) we find the variable bVasiPieni.

Data structures and constants definition can be found in **Script editor - All functions**.

Xscript Agv Data structures
============================

In the documentation under the voice **Estensione x-script per AgvManager --> Funzioni per la gestione degli agv**, we can find some functions and data structures to manage AGVs. Here we will present some data structures and functions that can operate on them.

Note that AgvManager have internal data structures where to save informations about vehicles, maps, points, etc. When we need, for example to get information about agv number 4, we create a strucutre similar to the one AgvManager have and by calling a dedicated function we can get information on Agv 4.

XMapParams
-------------

This structures contains some fields to define the dimensions of an AGV and movement behavior. There are two functions that operate on this structure to get information from AgvManger and set information to it.
For example, if we define a variable ``mPar`` as: ``XMapParams mPar``, we can read parameter from AgvManager and store them into this structure by calling the function ``AgvGetMapParams(@mpar)``.

If we need to modify some parameter we can we use the dot operator of the structure.
To apply modification onto AgvManager we have to call the function ``AgvSetMapParams(@mpar)``, that transfer the data from the structure ``mPar`` to AgvManager.

Note the use of ``@`` when passing the variable ``mPar`` to these 2 functions. The variable is passed by reference not by value.

Meaning of the structure fields??????????

.. code-block:: none
  :caption: XMapParams
  :name: lstXMapParams

  ;	Parametri definizione comportamento movimentazione veicoli
  ;
  object XMapParams
    setSymmetricalVehicleDimension(int length, int width, int diagonal=0)
    setVehicleDimension(int length_front, int length_rear, int width_left, int width_right, int radius = 0)

    int iLunghVeicolo                  // Larghezza veicolo (per anticollisione)
    int iLarghVeicolo                  // Lunghezza veicolo (per anticollisione)
    int iDiagVeicolo                   // Diagonale veicolo (per anticollisione, autocalcolata se == 0)
    int iVehicleLengthFront            // Lunghezza veicolo dal centro in avanti (per anticollisione)
    int iVehicleLengthRear             // Lunghezza veicolo dal centro all'indietro (per anticollisione)
    int iVehicleWidthLeft              // Larghezza veicolo dal centro verso sinistra (per anticollisione)
    int iVehicleWidthRight             // Larghezza veicolo dal centro verso destra (per anticollisione)
    int iVehicleRadius                 // Raggio di massimo ingombro durante rotazione (per anticollisione)
    real dHandicapIncrocio             // Distanza aggiunta per uso incrocio (DEPRECATO)
    real dHandicapForRotation          // Distanza aggiunta per ogni rotazione
    real dHandicapForCurve             // Distanza aggiunta per ogni curva
    real dUseHandicap                  // Handicap per segmenti prenotati in senso contrario
    real dFattoreCurva                 // (Non usato, deprecato) Fattore moltiplicativo dell'ingombro in caso di curva
    real dAngMinCurvaOk                // Angolo per cui il cambio di corridoio e' possibile con una rotazione anche se c'e' divieto di ingombro dei quadranti (cambio verso)
    real dHandicapIncontraDestMove     // Distanza aggiunta se si incontra un veicolo fermo. Se negativa non si passa proprio (ricerca di un percorso alternativo)
    real dHandicapMarciaIndietro       // Handicap moltiplicativo per tratti a marcia indietro
    real dDistanzaDaIncrocioOkFermata  // Distanza minima per fermata prima o dopo un incrocio in prenotazione movimento
    bool bOkInversioneSuIncrocio       // Ok inversione su incrocio
    bool bNoFermataSuIncrocio          // Vietato fermare su incrocio
    bool bPalletBloccaPercorso         // Non si passa su user di tipo 'C' occupato
    bool bNoMovSuTrattiPrenotati      // Non muovere gli agv su tratti prenotati da altri agv
    bool bNoPercorsoAgvNoMis          // Escludi dal percorso tratti su cui si trovano agv non in missione
    bool bNoPercorsoAgvDisab          // Escludi dal percorso tratti su cui si trovano agv disabilitati (e non in missione)
    bool bDontMoveOnDestMove          // Non muovere un agv se andrebbe a finire sulla destinazione di un altro agv
    int iAgvPosThreshold              // Soglia di posizione agv
  endobject

XVehicleInfo
-------------
This data structure contain information about the vehicle, for example alarm status, mission in progress, operating mode, capacity of battery, etc.
To get information from AgvManager about the vehicle we can call the function ``AgvGetVehicleInfo(uint agvId, xvehicleinfo & info)``, we pass to the function the index of the agv and an XVehicleInfo variable.

For example if we need information about Agv number 4, we create the structure ``XVehicleInfo vInfo``, then we call ``AgvGetVehicleInfo(4, @vInfo)``.
In this way, we can for example read the battery status ``vInfo.uBatteryCapacity``. Note that after a while the Agv is working, this value will be different from the value AgvManager have, we have to call again the function ``AgvGetVehicleInfo(,)`` in order to update information.

The field ``uint uStatus``, **Vehicle Status Flag**,  is an 32 bit unsigned integer where information are saved in a binary way. There are defined some constants (flags) in order to decode information:

.. code-block:: none

  // Vehicle status flags, bit mask 2^n.
  // 4 least significant bits

  $define VST_POTENZA_ATTIVA   1 // Power active, mask bit 0
  $define VST_EXEC_COMANDO     2 // executing command, mask bit 1
  $define VST_CARICO_PRESENTE  8 // load present, mask bit 3
  $define VST_CARICA_INCORSO   4 // charge in progress, mask bit 2

For example we need to know if the vehicle have a load on board, we can write:
``bLoadOnBoard = vInfo.uStatus & VST_CARICO_PRESENTE``.

The first 4 bits (from 0 to 3), are reserved to system vehicle status (status communicated by the vehicle to AgvManager).
The user can define, in the AGV controller software, its own flag status beginning from bit 4, depending on the state of the vehicle and the plant requirements.

.. code-block:: none

  //
  //	Informazioni stato attuale veicolo
  //
  object XVehicleInfo
    uint uLineID            // Id. linea attuale agv
    int iPosition           // Posizione [mm] dell'agv sulla linea attuale
    int iAngle              // Angolo in gradi attuale dell'agv
    uint uMode              // Modalita' attuale veicolo (vedi etichette VM_)
    uint uStatus            // Flag di stato veicolo (vedi etichette VST_)
    uint uAlarmStatus       // Flag di allarme veicolo
    uint uBatteryCapacity   // Capacita' batteria:
    // 0		= Batteria completamente carica
    // 1000	= Batteria completamente scarica
    float dBatteryPerc      // Percentuale carica batteria : 0.0	= completamente scarica, 100.0 = completamente carica
    uint uMission           // Id. missione attuale agv
    uint uCommand           // Id. comando attuale agv
    uint uDirV              // Direzione sul corridoio: uno tra [FRSD]
    uint uExtendedPalletId  // Identificativo informazioni estese pallet (se presenti)
    XLastMoveInfo lastMove  // Ultimo movimento registrato (.isValid indica validita', in piu' se non valido, .uLine == 0)
    XLastMoveInfo destMove  // Destinazione finale (.isValid indica validita', in piu' se non valido, .uLine == 0)
  endobject

XSiteInfo
----------
A data structure where information about site can be stored. A site can be a user point or a battery point.
By calling ``agvGetSiteInfo(int userPointId, XSiteInfo& sInfo)`` we can get the user point informations from AgvManager. The function have as parameter the **id or code** of the user point and a reference to a ``XSiteInfo`` variable.
The function return true if the user point exist.
With the function ``agvSetSiteInfo(int userPointId, XSiteInfo & sInfo)`` we can set the parameter of a user point in AgvManager.

For example the field ``bPresenza`` is a boolean variable that indicate if the user point contain a loading unit or not.

When executing a loading operation into the vehicle **from station to vehicle**, by calling the function ``AgvExecLoad(agv,userPoint)`` the value of ``bPresenza`` is set to false and the vehicle status flag, ``uStatus``, corresponding to ``VST_CARICO_PRESENTE`` is set to true.

When executing an unload operation **from vehicle to the station**, by calling ``AgvExecUnload(agv, userPoint)`` the value of ``bPresenza`` to true. Note that these functions execute a logical load and unload. No commands are sent to agv. To let the agv execute a load or unload operations, ``agvRegisterOperation()`` must be called.

Some fields can be read and write ``(rw)`` from the script others are read only ``(ro)``.

``bVasiPieni`` is a variable that indicate the presence of a load on the UDC (Loading Unit).

.. code-block:: none

  ;	Informazioni associate a punto USER

  object XSiteInfo
  	bool bAttiva           // (rw)
  	bool bPriorita         // (rw)
  	bool bPresenza         // (rw) // UDT on board
  	bool bVasiPieni        // (rw) // product on borad of UDT
  	bool bInAllarme        // (rw)
  	bool bVisibile         // (rw)
  	uint uTipo             // (rw)
  	uint uFlags            // (ro) // Vehicle flags. see UF_***
  	real dStoreTime        // (ro) in giorni
  	uint uLato             // (ro) [L (sinistra) | R (destra) | C (centro)]
  	uint uExtendedPalletId // (ro) Identificativo informazioni estese pallet (se presenti)
  endobject

By calling the function ``agvGetUserFlags(uint user)``, we get the user point flags ``XSiteInfo.uFlags``.

.. code-block:: none

  //
  // Definizione codici User Flags
  //
  // Flags riservati ad AgvManager :
  $define UF_MODIFIED            1
  $define UF_NO_FREQ             2
  $define UF_INUSE               4
  $define UF_PRE_INUSE           8
  $define UF_PALLET_SU_PERCORSO  32
  $define UF_MASK_FLAGS_AGVM     4095

  // Flags impostabili da script ed usati da AgvManager
  $define UF_ACCESSIBLE          0x00001000	// Passaggio attraverso user possibile (default vero)
  $define UF_FORCE_STOP          0x00002000	// Obbligo di spezzare movimento
  $define UF_NO_STOP             0x00004000	// Punto di sosta vietata
  $define UF_NO_STOP_CROSS       0x00008000	// E' vietato fermare l'agv su questo incrocio
  $define UF_FORCE_BREAK_CROSS   0x00010000	// Obbligo di spezzare movimento su incrocio
  $define UF_BLINK_ICON          0x00020000	// Se sito in attesa di missione o riservato, icona blinka
  $define UF_NO_INVERSIONE       0x00040000	// Vietato fare inversione su questo punto

  // Flags impostabili da script e non usati da AgvManager
  $define UF_RESERVED            0x00080000	// Prenotato da agv per missione (viene disegnato bollo rosso)
  $define UF_MISS_OK             0x00100000	// Sito in attesa di missione (viene disegnato bollo verde)
  $define UF_FLAG_XSCRIPT        0x01000000	// Primo flag utilizzabile liberamente da script

  //  Esempio d'uso :
  //  $define UF_MY_FLAG_1			shl(UF_FLAG_XSCRIPT,1)
  //  $define UF_MY_FLAG_2			shl(UF_FLAG_XSCRIPT,2)

XListaSiti
-----------

The xListaSiti store a reference to a site (user o battery point). The operations done on elements of list are applied to sites in map. Imagine the list as a pointer to the sites in map.

.. code-block:: none

  //~~~~~~~~~~~~~~~~~~~~~~~~~~~
  //    Gestione lista siti
  //~~~~~~~~~~~~~~~~~~~~~~~~~~~

  object XListaSiti
  	uint pObj									// INTERNAL POINTER - DO NOT TOUCH
  	internal 0x02000600 Constructor()
  	internal 0x02000601 Destructor()
  	internal 0x02000602 IsEmpty() : bool		// Test se lista vuota
  	internal 0x02000603 Count() : uint			// Ritorna numero siti in lista
  	internal 0x02000604 Prepend(uint)			// Aggiunge sito in testa alla lista
  	internal 0x02000604 AddHead(uint)			// DEPRECATED
  	internal 0x02000605 Append(uint)			// Aggiunge sito in coda alla lista
  	internal 0x02000605 AddTail(uint)			// DEPRECATED
  	internal 0x02000606 Find(uint) : uint		// Torna posizione sito in lista (-1 se non trovato)
  	internal 0x02000607 RemoveFirst() : uint	// Rimuove il primo sito dalla lista, e ne torna il valore
  	internal 0x02000607 RemoveHead() : uint		// DEPRECATED
  	internal 0x02000608 RemoveLast() : uint		// Rimuove l'ultimo sito dalla lista, e ne torna il valore
  	internal 0x02000608 RemoveTail() : uint		// DEPRECATED
  	internal 0x02000609 RemoveAt(uint) : uint	// Rimuove il sito alla posizione specificata (ritorna l'indice del sito rimosso)
  	internal 0x0200060A RemoveAll()				// Svuota la lista
  	internal 0x0200060B At(uint) : uint			// Torna l'indice del sito alla posizione specificata
  	internal 0x0200060C SetFlag(uint)			// Impostazione flag da settare per tutti i siti in lista
  	internal 0x0200060D AllFlags() : uint		// Torna l'or binario dei flags di tutti i siti in lista
  	internal 0x0200060E contains(uint) : bool	// Test user presente in lista
  endobject

Some useful functions
======================

We already see some callback funtions like ``onApplicationStart()``, ``onNextMission()``, ``onExpandMacro()``, ``onExecuteMicro()`` and some utility functions like ``agvAddMacro()``, ``agvAddWayPoint()``, ``agvRegisterPassante()``, ``agvRegisterBloccante()``, ``agvRegisterOperation()``.

There are a lot of functions provided by AgvManager. We will some of them in the examples. We will see also how we can create our own functions and objects.

Movement functions
-------------------
In the documentation we can find some functions, e.g. ``agvMoveToWayPoint()``, ``AgvRegisterMoveTo()``, etc. to define the movement destination as well as the path ``agvAddWayPoint()``, and some constants.

Constants related to this category of functions begin with ``MoveResult_`` or ``EsitoMov_``, some of these constants are self-explanatory, e.g. ``MoveResult_WaypointReached``, ``MoveResult_CompletedMovement``.

For example if we want to give the final destination without caring about the path we can call ``AgvRegisterMoveTo()`` in the callback function ``onExpandMacro()`` giving to it as input the destination point and agvManager build the path automatically. The path may be recalculated every time ``onExpandMacro()`` is called, depending on the state of the plant and other Agvs.

If we want to build the path we can use ``agvAddWayPoint()`` and ``agvMoveToWayPoint()``. We register different macros as the way point. We can build the path by waypoints when we compile the macro list, in ``registerMission`` and ``registerMovement``. Then the motion is executed in ``onExpandMacro()`` by calling ``agvMoveToWayPoint()``.

MICRO registration functions
-----------------------------
The following functions register a micro operation or instruction:
	- agvRegisterSystemPassante(,,,,) MIC\_SYSTEM, system micro instruction.
	- agvRegisterSystemBloccante(,,,,) MIC\_SYSTEM, system micro instruction.

	- agvRegisterPassante(,,,,) [P] command, Pass-through operation.
	- agvReigsterOperation(,,,,) MIC\_OPERATION [O] Operation to send to the vehicle. The syntax of the command is: [Occccmmmm,type,p1,p2,p3,p4].
	- agvRegisterMovingOperation(,,,,) MIC\_MOVE [Q] Operation with movement.

	- agvRegisterWait(,,,,) [W] Wait condition operation.


To get a list of all micro type search in the documentation the prefix "{MIC\_}". Other types of Micros are registerd directly by AgvManager like micro of type MIC\_MOVE.

Points
-------

- agvUserExists(uint uCode) : return true if a generic point, user point or cross exist.
- siteExists(uint uCode) : return true if the site (USER or CBat point) exists.
- agvGetSiteInfo(uint userId, xSiteInfo \&sInfo): get information about USER point with id userId.
- SetSiteText(uint userId, string text) : set a text to shown on the user point on the map. e.g. SetSiteText(userId, "(" + row + ", " + col + ")").
- SetSiteName(uint userId, string text) : set the name of the site, visible in the tooltip

We can associate two different kind of properties to a point: int or string. Properties can be used by the script as we wish.

- SetIntProperty(uint, string, int)
- IntProperty(uint, string)
- addInProperty(,,,,)
- ``AddIntProperty(i, PROP_ASSIGNED_AGV, "Assigned agv", ACCESS_INST, XSitePropertyFlg_volatile)``

Creating functions and objects
===============================

Functions are useful to divided our logic and simplify the program. The keyword code is used to create functions.
Use the keyword Forward, if you want to use the fucntion in another function that you implemented before it. It is like the prototype of the function in C language.

Objects are like classes in object oriented programming. Objects can be created by using object and endobjects.
Classes have a constructor function, that is called when the class is instanciated, when the the object is created from the class.
