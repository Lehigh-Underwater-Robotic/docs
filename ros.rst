ROS
===

.. _Required packages:
.. _Creating ROS packages:
.. _Connecting to the Jetson:
.. _Building:
.. _Running:

Required packages
-----------------

OpenCV (refer to `OpenCV section <https://lehigh-underwater-robotics.readthedocs.io/en/latest/computer_vision.html#yolo>`_ of docs)
ROS Foxy `(sudo apt install ros-foxy-ros-base python3-argcomplete) <https://docs.ros.org/en/foxy/Installation/Ubuntu-Install-Debians.html#>`_
Colcon (sudo apt install python3-colcon-common-extensions)
rosdep (sudo apt install python3-rosdep)

If unable to communicate with devices

.. code::

    sudo chmod 666 /dev/ttyACM0
    sudo chmod 666 /dev/ttyACM1
    sudo gpasswd -a $USER dialout

Creating ROS packages
---------------------

.. code::

    $ ros2 pkg create --build-type ament_cmake --node-name {NODE_NAME} {PKG_NAME} --dependencies rclcpp std_msgs
    
      // install all required dependencies
    $ rosdep install -i --from-path src --rosdistro foxy -y
      // install required dependencies for specific package
    $ rosdep install -i --from-path src/{PKG_DIR} --rosdistro foxy -y


Connecting to the Jetson
------------------------

.. note::

    You must be connected to the same network as the Jetson.

.. code::

   $ ssh lur@192.168.1.122

Building
--------

.. code::

    $ ./scripts/build_lur.sh
    $ . install/local_setup.bash

    $ ./scripts/build.sh
    $ . install/local_setup.bash

    $ ./scripts/build_test
    $ . install/local_setup.bash

Running
-------

.. code:: 

    // start mavros
    ros2 launch launch_files/mavros.yaml
    // change mode to stabilize
    ros2 run mavros mav sys mode -c STABILIZE
    // arm drone
    ros2 run mavros mav safety arm
    // starts brain and camera
    ros2 launch launch_files/main.yaml
    // reads in 'man_test.txt' and publishes to brain
    // which then publishes to manual control
    ros2 launch launch_files/test.yaml
