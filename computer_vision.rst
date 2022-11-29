Computer Vision
===============

.. _OpenCV:
.. _YOLO:

OpenCV
------
`OpenCV <https://opencv.org/>`_

OpenCV is preinstalled on the Jetson but if you have trouble with the installation find the documentation here:
`Basic installation <https://docs.opencv.org/4.x/d7/d9f/tutorial_linux_install.html>`_

.. note::

   It is necessary to run the image processing work on the GPU of the jetson. OpenCV was compiled from source with special flags to enable this. See our OpenCV compile script `here <https://github.com/ntoddlong/lur/blob/main/scripts/opencv_jetson_build.sh>`_

YOLO
----
`YOLO <https://pjreddie.com/darknet/yolo>`_
