Teensy
======

.. _Arduino library:
.. _Message protocol:
.. _Thruster config:
.. _Code reference:

Arduino library
---------------
.. code-block::

  Directory structure:
  --------------------

  lur
  ├── lur.cpp
  ├── lur.h
  └── messages.h

The following objects are implemented

* Sub
    Main control. Handles all other objects.
* Jetson
    Serial communication interface with Jetson.
* Motors
    Contains 8 servo objects that are the motors
* Sonar
    Parsing and controls for ping sonar
* IMU
    Inertial measurement unit data handling

Building
~~~~~~~~
Copy lur directory to arduino library directory on your computer

.. code-block::

  /your/path
  └── Arduino
      └── libraries
          └── lur
              ├── lur.cpp
              ├── lur.h
              └── messages.h

In arduino ide, open teensy.ino and compile/upload.

Message protocol
----------------
The Teensy and the Jetson talk through a usb serial connection.

Message format is defined in messages.h and contains several pieces.

A message id identifies the type of message being recieved

.. code-block::

  enum class ID: uint8_t {
    ok,
    error,
    disarm,
    arm,
    set_mode,
    get_mode,
    manual_control,
    thruster_test,
  };

Errors types are defined as follows

.. code-block::

  enum class ERROR: uint8_t {
    invalid,
    parse,
    checksum,
    disarmed,
  };

* invalid
    given message id isn't a valid type
* parse
    failed to parse message
* checksum
    checksum check failed
* disarmed
    drone is disarmed and unable to process command

The drone can be put in one of the following modes

.. code-block:: c++

  enum Mode {
    disarmed,
    armed,
    stabilize,
    manual,
    depth_hold,
    position_hold,
  };


Common format
~~~~~~~~~~~~~

.. code-block::

  -----------------------------------------
  | byte |   type    |   name    | values |
  -----------------------------------------
  |   0  |  uint8_t  |  header   |  0xff  |
  -----------------------------------------
  |   1  |  uint8_t  |    id     |  0-255 |
  -----------------------------------------
  |  2-n | see specs |   data    |  any   |
  -----------------------------------------
  |  n+1 |  uint8_t  | checksum  |  any   |
  -----------------------------------------
   
  2 <= n <= 254

Checksum
~~~~~~~~
The checksum is calculated as the 2's complement of the sum of all bytes.
See `here <https://en.wikipedia.org//wiki/Intel_HEX>`_ for more info

Data specification
~~~~~~~~~~~~~~~~~~

ok - NONE

error

.. code-block::

  -----------------------------------------
  | byte |   type    |   name    | values |
  -----------------------------------------
  |   0  |  uint8_t  |   type    |  0-255 |
  -----------------------------------------

disarm - NONE

arm - NONE

set_mode

.. code-block::

  -----------------------------------------
  | byte |   type    |   name    | values |
  -----------------------------------------
  |   0  |  uint8_t  |   mode    |  0-255 |
  -----------------------------------------

get_mode - NONE

manual_control

.. code-block::

  ------------------------------------------
  | byte |   type   |   name    |  values  |
  ------------------------------------------
  |   0  |  int8_t  |     x     | -100-100 |
  ------------------------------------------
  |   1  |  int8_t  |     y     | -100-100 |
  ------------------------------------------
  |   2  |  int8_t  |     z     | -100-100 |
  ------------------------------------------
  |   3  |  int8_t  |    roll   | -100-100 |
  ------------------------------------------
  |   4  |  int8_t  |    pitch  | -100-100 |
  ------------------------------------------
  |   5  |  int8_t  |    yaw    | -100-100 |
  -------------------------------------------

thruster_test - NONE

Thruster config
---------------
This is the configuration we use. It is based on the `ArduSub Vectored ROV with Four Vertical Thrusters. <https://www.ardusub.com/introduction/features.html>`_

.. image:: images/vectored6dof-frame.png
  :width: 400
  :alt: Config Image

The following matrix represents the configuration of the thrusters.

The columns are the individual thrusters, while the rows are the directions.

.. code-block::

  -----------------------------------------
  |       | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
  |----------------------------------------
  |   x   |
  |--------
  |   y   |
  |--------
  |   z   |
  |--------
  | roll  |
  |--------
  | pitch |
  |--------
  |  yaw  |
  ---------

  const float thruster_config[6][8] = {
    {  1.0,  1.0, -1.0, -1.0,  0.0,  0.0,  0.0,  0.0  },
    {  1.0, -1.0,  1.0, -1.0,  0.0,  0.0,  0.0,  0.0  },
    {  0.0,  0.0,  0.0,  0.0,  1.0,  1.0,  1.0,  1.0  },
    {  0.0,  0.0,  0.0,  0.0, -1.0,  1.0, -1.0,  1.0  },
    {  0.0,  0.0,  0.0,  0.0,  1.0,  1.0, -1.0, -1.0  },
    { -1.0,  1.0,  1.0, -1.0,  0.0,  0.0,  0.0,  0.0  }
  };

To calculate the power value for thruster 5 going in the z direction (ascend/descend) at a power value p, we multiply p by the value in the 3rd row, 5th column.

Calibrating
~~~~~~~~~~~
Calibrating the thruster values requires running tests in the water.

Working in one direction at a time, run several tests in that direction and monitor the results. After observing the movement of the drone, go through each thruster and adjust the value in the matrix according to the needed relative power of the thruster.

For example if you are running an x direction test and the drone is pulling the right

.. note::

  Values should be between -1 and 1, inclusive. These represent either a full power reverse or full power forward.

Code reference
--------------
The following are all the objects that are implemented and their associated methods.

Motors
~~~~~~

.. code-block:: c++
  
  struct Motors {
    bool  armed;
    Servo thrusters[NUM_THRUSTERS];
    Motors();
    void init();
    void arm();
    void disarm();
    bool set_power(const int (&values)[NUM_THRUSTERS]);
    void add_to_power_vector(int (&values)[NUM_THRUSTERS], const float (&config)[NUM_THRUSTERS], int val);
    int  normalize(int n, int min, int max);
    void normalize_array(int (&values)[NUM_THRUSTERS]);
    bool manual_control(int x, int y, int z, int roll, int pitch, int yaw);
  };

Sonar
~~~~~

.. code-block:: c++

  struct Sonar {
    Ping1D         device;
    SoftwareSerial ping_serial;
    Sonar();
    bool init();
  };

IMU
~~~

.. code-block:: c++

  struct IMU {
    Adafruit_BNO055 device;
    IMU();
    bool init(); 
    uint8_t get_temp();
  };

Jetson
~~~~~~

.. code-block:: c++

  struct Jetson {
    Jetson();
    bool init();
    bool send();
    bool receive();
  };

Sub
~~~

.. code-block:: c++

  struct Sub {
    Mode  mode;
    Motors* motors;
    Sonar* sonar;
    IMU* imu;
    Sub();
    bool set_mode(Mode m);
  };
