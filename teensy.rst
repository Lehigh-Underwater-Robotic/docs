Teensy
======

.. _Arduino library:
.. _Serial communication:
.. _Thruster config:

Arduino library
---------------
.. code-block::

  Directory structure:
  --------------------

  lur
  ├── lur.cpp
  ├── lur.h
  └── messages.h

Building
--------
Copy lur directory to arduino library directory on your computer
.. code-block::

  /your/path
  └── Arduino
      └── libraries
          └── lur
              ├── lur.cpp
              ├── lur.h
              └── messages.h

In arduino ide, open teensy.ico and compile/upload.

Serial communication
--------------------

Thruster config
---------------
.. image:: images/vectored6dof-frame.png
  :width: 400
  :alt: https://www.ardusub.com/introduction/features.html

The following matrix represents the configuration of the thrusters.

The columns are the individual thrusters, while the rows are the directions
.. code-block::
       1  2  3  4  5  6  7  8
  x
  y
  z
  roll
  pitch
  yaw

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
-----------
Calibrating the thruster values requires running tests in the water.

Working in one direction at a time, run several tests in that direction and monitor the results. After observing the movement of the drone, go through each thruster and adjust the value in the matrix according to the needed relative power of the thruster.

For example if you are running an x direction test and the drone is pulling the right

.. note::

  Values should be between -1 and 1, inclusive. These represent either a full power reverse or full power forward.
