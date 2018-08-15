
Overview on some functions
*****************************

Access level
------------

Sometime in a plant to a worker is permitted to do some job, to maintainer other jobs and so. In AgvManager are defined 5 different levels of users, that correspond to 5 different constants:

.. code-block:: none

	$define ACCESS_USER1 0
	$define ACCESS_USER2 1
	$define ACCESS_USER3 2
	$define ACCESS_INST  3   ; Installer
	$define ACCESS_NO_OP 4

The actual level can be read by calling the function ``ActualAccessLevel()``.

.. code-block:: none

	SetAccessLevelForOperation(DefQual_OpTrascinaAgvSuLinea, ACCESS_INST)

onUpdateIO()
--------------

Drag and drop is useful to test vehicle. Normally a vehicle have to respond to some commands and react under some conditions that come from the plant. AgvManager can read input and output from a plc or a database. The callback function ``onUpdateIO(,,)`` is used to read input from a plc and write outputs to a plc.

IO can be defined in AgvConfigurator, in the tab PLC we define the communication protocol and the number of DWord (uint 32bit) to be exchanged in input and output. In the tab I/O description we can assign names to digital inputs and outputs.
In AgvManager, in the tab **Input/Output [F5]** we can see the list of IO, read the value and force inputs and outputs.

The callback function ``onUpdateIO`` is called at the beginning of the main loop cycle. In this function we can read inputs by calling the fucntion ``agvGetInputXXXX`` and write outputs by calling ``agvSetOutputXXXX``.

There are different get and set functions to read inputs and write outputs, it depend on what and how we read or write. For example, ``bool agvGetInput(uint offset)`` read the bit that have index ``offset`` and return a boolean value depending on the value of that bit. ``agvGetInputDWord(uint offset, uint & val)`` read the DWord at the index ``offset`` and write the value in ``val``, note that ``val`` is passed to the function by reference.

Note that the first bit have ``offset = 1`` not ``0``. It is convenient to define some constants that represent IO signals. For example it the signal ``Unloading done``, e.g. a push button connected to input 7, i.e. byte 0 bit 7, we can define a constant like ``$define INP_UNLOADING_DONE 7``, then call the function ``bool iUnloadDone= AgvGetInput(INP_UNLOADING_DONE)``. The same can be done for Outputs.


OnAgvStatusChange(): Agv status flag change
--------------------------------------------

When the flag status of the vehicle ``xVehicleInfo.uStatus`` change value, the callback function ``OnAgvStatusChange()`` is called by AgvManager.

The fucntion ``SetAgvStatusDescription(uAgv, int stId, string desc)``. We can set the description of the bit with index stId.
If we want e.g. to change the descrition of the status bit ``VST_CARICO_PRESENTE`` we have to write:

.. code-block:: none

	loadType = TYPE_EMPTY_TROLLEY
	SetAgvStatusDescription(uAgv, -3, "<font color=" + colorName(AgvGetTYpeColor(TYPE_EMPTY_TROLLEY)) + ">Trolley on agv</font>")}
	AgvSetAgvLoadInfo(uAgv, trolleyOnAgv, loadType, toiletOnTrolley)

In this example we change the description into **Trolley on agv**, the string is HTML formatted string. This information is shown in the windows **Vehicle information**.

The function ``AgvSetAgvLoadInfo()`` set information about the Loading Unit (UDC, Unita Di Carico) on the vehicle, e.g. ``AgvSetAgvLoadInfo(uAgv, bpresenza, loadType, bVasiPieni)``, ``bPresenza`` will be ``bPalletOnAgv``, ``bVasiPieni`` will be ``bPalletFull``.


OnAgvModeChange(): Agv Operating mode change
----------------------------------------------

When the vehicle operating mode ``xVehicleInfo.uMode`` changed, AgvManager call the callback function ``OnAgvModeChange(uint uAgv, uint oldMode, uint newMode)``.

By calling the function ``bool AgvInAutomatico(uAgv)`` we get true if the Agv is in automatic mode, i.e. ``VM_AUTOMATICO``, or in manual emergency mode, i.e. ``VM_MANU_EMERG``.

Look for the prefix ``VM_`` or ``MOD_`` to get a list of operating modes, depending on the Agv navigation type.

Settings: ``XSettings``
-------------------------

settings.ini file strucuture

Interaction with user: ``XForm``
---------------------------------

Qt creator

Semaphores
-----------

A semaphore can be only set to green by the script. A semaphore is set to red when all Agvs leave the area of the semaphore.

per quanto riguarda i semafori, come si fa ad associare l'agv alla richiesta? in AgvSetGreenSemaphore(uint , bool) il primo parametro e' il codice di startSemeforo, non c'e' il numero dell'agv.

Marco,
8:40 PM Una volta che il semaforo è verde lo è per tutti gli agv

8:52 PMAgvGetSemaphoreRequestMask(uint semaphoreStartId) : uint	// Ritorna stato richiesta semaforo. Torna la maschera degli agv che stanno chiedendo il semaforocome fai a settare i bit della maschera?

Marco,
9:06 PMLi setta AgvManager quando degli agv stanno aspettando di passare per l'area del semaforoServe che il percorso di un agv incroci l'area del semaforo, e che l'agv sia nelle vicinanze del semaforo stesso: in pratica la maschera viene impostata quando il semaforo sta bloccando il movimento dell'agvOvviamente serve anche che venga chiamata una funzione di esecuzione movimentoViene anche impostata se ad esempio un operatore porta l'agv in manuale di emergenza dentro l'area di pertinenza del semaforo, o se all'avvio del software un agv si trova già dentro l'area.

.. code-block:: none
  :caption: Semaphores functions and constants

  ;	Gestione semafori

  $define XSemaforo_nonesiste -1
  $define XSemaforo_rosso 0
  $define XSemaforo_verde 1
  AgvGetSemaphoreRequestMask(uint semaphoreStartId) : uint    ; Ritorna stato richiesta semaforo. Torna la maschera degli agv che stanno chiedendo il semaforo
  AgvSetGreenSemaphore(uint semaphoreStartId, bool)           ; Imposta conferma via libera semaforo con id semaphoreStartId
  AgvGetSemaphoreColour(uint semaphoreStartId) : int          ; Torna colore semaforo con id semaphoreStartId
