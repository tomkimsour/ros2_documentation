.. redirect-from::

    Tutorials/Ros2bag/Recording-And-Playing-Back-Data

.. _ROS2Bag:

Recording and playing back data
===============================

**Goal:** Record data published on a topic and a service so you can replay and examine it any time.

**Tutorial level:** Beginner

**Time:** 15 minutes

.. contents:: Contents
   :depth: 2
   :local:

Background
----------

``ros2 bag`` is a command line tool for recording data published on topics and services in your ROS 2 system.
It accumulates the data passed on any number of topics and services, then saves it in a database.
You can then replay the data to reproduce the results of your tests and experiments.
Recording topics and services is also a great way to share your work and allow others to recreate it.


Prerequisites
-------------

You should have ``ros2 bag`` installed as a part of your regular ROS 2 setup.

If you need to install ROS 2, see the :doc:`Installation instructions <../../../Installation>`.

This tutorial talks about concepts covered in previous tutorials, like :doc:`nodes <../Understanding-ROS2-Nodes/Understanding-ROS2-Nodes>`, :doc:`topics <../Understanding-ROS2-Topics/Understanding-ROS2-Topics>` and :doc:`services <../Understanding-ROS2-Services/Understanding-ROS2-Services>`.
It also uses the :doc:`turtlesim package <../Introducing-Turtlesim/Introducing-Turtlesim>` and  :doc:`Service Introspection Demo <../../Demos/Service-Introspection>`.

As always, don't forget to source ROS 2 in :doc:`every new terminal you open <../Configuring-ROS2-Environment>`.


Managing Topic Data
-------------------

1 Setup
^^^^^^^

You'll be recording your keyboard input in the ``turtlesim`` system to save and replay later on, so begin by starting up the ``/turtlesim`` and ``/teleop_turtle`` nodes.

Open a new terminal and run:

.. code-block:: console

    ros2 run turtlesim turtlesim_node

Open another terminal and run:

.. code-block:: console

    ros2 run turtlesim turtle_teleop_key

Let's also make a new directory to store our saved recordings, just as good practice:

.. tabs::

    .. group-tab:: Linux

        .. code-block:: console

            mkdir bag_files
            cd bag_files

    .. group-tab:: macOS

        .. code-block:: console

            mkdir bag_files
            cd bag_files

    .. group-tab:: Windows

        .. code-block:: console

            md bag_files
            cd bag_files


2 Choose a topic
^^^^^^^^^^^^^^^^

``ros2 bag`` can record data from messages published to topics.
To see the list of your system's topics, open a new terminal and run the command:

.. code-block:: console

  ros2 topic list

Which will return:

.. code-block:: console

  /parameter_events
  /rosout
  /turtle1/cmd_vel
  /turtle1/color_sensor
  /turtle1/pose

In the topics tutorial, you learned that the ``/turtle_teleop`` node publishes commands on the ``/turtle1/cmd_vel`` topic to make the turtle move in turtlesim.

To see the data that ``/turtle1/cmd_vel`` is publishing, run the command:

.. code-block:: console

  ros2 topic echo /turtle1/cmd_vel

Nothing will show up at first because no data is being published by the teleop.
Return to the terminal where you ran the teleop and select it so it's active.
Use the arrow keys to move the turtle around, and you will see data being published on the terminal running ``ros2 topic echo``.

.. code-block:: console

  linear:
    x: 2.0
    y: 0.0
    z: 0.0
  angular:
    x: 0.0
    y: 0.0
    z: 0.0
    ---


3 Record topics
^^^^^^^^^^^^^^^

3.1 Record a single topic
~~~~~~~~~~~~~~~~~~~~~~~~~

To record the data published to a topic use the command syntax:

.. code-block:: console

    ros2 bag record <topic_name>

Before running this command on your chosen topic, open a new terminal and move into the ``bag_files`` directory you created earlier, because the rosbag file will save in the directory where you run it.

Run the command:

.. code-block:: console

    ros2 bag record /turtle1/cmd_vel

You will see the following messages in the terminal (the date and time will be different):

.. code-block:: console

    [INFO] [rosbag2_storage]: Opened database 'rosbag2_2019_10_11-05_18_45'.
    [INFO] [rosbag2_transport]: Listening for topics...
    [INFO] [rosbag2_transport]: Subscribed to topic '/turtle1/cmd_vel'
    [INFO] [rosbag2_transport]: All requested topics are subscribed. Stopping discovery...

Now ``ros2 bag`` is recording the data published on the ``/turtle1/cmd_vel`` topic.
Return to the teleop terminal and move the turtle around again.
The movements don't matter, but try to make a recognizable pattern to see when you replay the data later.

.. image:: images/record.png

Press ``Ctrl+C`` to stop recording.

The data will be accumulated in a new bag directory with a name in the pattern of ``rosbag2_year_month_day-hour_minute_second``.
This directory will contain a ``metadata.yaml`` along with the bag file in the recorded format.

3.2 Record multiple topics
~~~~~~~~~~~~~~~~~~~~~~~~~~

You can also record multiple topics, as well as change the name of the file ``ros2 bag`` saves to.

Run the following command:

.. code-block:: console

  ros2 bag record -o subset /turtle1/cmd_vel /turtle1/pose

The ``-o`` option allows you to choose a unique name for your bag file.
The following string, in this case ``subset``, is the file name.

To record more than one topic at a time, simply list each topic separated by a space.

You will see the following message, confirming that both topics are being recorded.

.. code-block:: console

  [INFO] [rosbag2_storage]: Opened database 'subset'.
  [INFO] [rosbag2_transport]: Listening for topics...
  [INFO] [rosbag2_transport]: Subscribed to topic '/turtle1/cmd_vel'
  [INFO] [rosbag2_transport]: Subscribed to topic '/turtle1/pose'
  [INFO] [rosbag2_transport]: All requested topics are subscribed. Stopping discovery...

You can move the turtle around and press ``Ctrl+C`` when you're finished.

.. note::

    There is another option you can add to the command, ``-a``, which records all the topics on your system.

4 Inspect topic data
^^^^^^^^^^^^^^^^^^^^

You can see details about your recording by running:

.. code-block:: console

    ros2 bag info <bag_file_name>

Running this command on the ``subset`` bag file will return a list of information on the file:

.. code-block:: console

    ros2 bag info subset

.. code-block:: console

  Files:             subset.mcap
  Bag size:          228.5 KiB
  Storage id:        mcap
  Duration:          48.47s
  Start:             Oct 11 2019 06:09:09.12 (1570799349.12)
  End                Oct 11 2019 06:09:57.60 (1570799397.60)
  Messages:          3013
  Topic information: Topic: /turtle1/cmd_vel | Type: geometry_msgs/msg/Twist | Count: 9 | Serialization Format: cdr
                   Topic: /turtle1/pose | Type: turtlesim_msgs/msg/Pose | Count: 3004 | Serialization Format: cdr

5 Play topic data
^^^^^^^^^^^^^^^^^

Before replaying the bag file, enter ``Ctrl+C`` in the terminal where the teleop is running.
Then make sure your turtlesim window is visible so you can see the bag file in action.

Enter the command:

.. code-block:: console

    ros2 bag play subset

The terminal will return the message:

.. code-block:: console

    [INFO] [rosbag2_storage]: Opened database 'subset'.

Your turtle will follow the same path you entered while recording (though not 100% exactly; turtlesim is sensitive to small changes in the system's timing).

.. image:: images/playback.png

Because the ``subset`` file recorded the ``/turtle1/pose`` topic, the ``ros2 bag play`` command won't quit for as long as you had turtlesim running, even if you weren't moving.

This is because as long as the ``/turtlesim`` node is active, it publishes data on the  ``/turtle1/pose`` topic at regular intervals.
You may have noticed in the ``ros2 bag info`` example result above that the  ``/turtle1/cmd_vel`` topic's ``Count`` information was only 9; that's how many times we pressed the arrow keys while recording.

Notice that ``/turtle1/pose`` has a ``Count`` value of over 3000; while we were recording, data was published on that topic 3000 times.

To get an idea of how often position data is published, you can run the command:

.. code-block:: console

    ros2 topic hz /turtle1/pose

Managing Service Data
---------------------

1 Setup
^^^^^^^

You'll be recording service data between ``introspection_client`` and ``introspection_service``, then display and replay that same data later on.
To record service data between service client and server, ``Service Introspection`` must be enabled on the node.

Let's start ``introspection_client`` and ``introspection_service`` nodes and enable ``Service Introspection``.
You can see more details for :doc:`Service Introspection Demo <../../Demos/Service-Introspection>`.

Open a new terminal and run ``introspection_service``, enabling ``Service Introspection``:

.. code-block:: console

  ros2 run demo_nodes_cpp introspection_service --ros-args -p service_configure_introspection:=contents

Open another terminal and run ``introspection_client``, enabling ``Service Introspection``:

.. code-block:: console

  ros2 run demo_nodes_cpp introspection_client --ros-args -p client_configure_introspection:=contents

2 Check service availability
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``ros2 bag`` can only record data from available services.
To see the list of your system's services, open a new terminal and run the command:

.. code-block:: console

  ros2 service list

Which will return:

.. code-block:: console

  /add_two_ints
  /introspection_client/describe_parameters
  /introspection_client/get_parameter_types
  /introspection_client/get_parameters
  /introspection_client/get_type_description
  /introspection_client/list_parameters
  /introspection_client/set_parameters
  /introspection_client/set_parameters_atomically
  /introspection_service/describe_parameters
  /introspection_service/get_parameter_types
  /introspection_service/get_parameters
  /introspection_service/get_type_description
  /introspection_service/list_parameters
  /introspection_service/set_parameters
  /introspection_service/set_parameters_atomically

To check if ``Service Introspection`` is enabled on the client and service, run the command:

.. code-block:: console

  ros2 service echo --flow-style /add_two_ints

You should see service communication like below:

.. code-block:: console

  info:
    event_type: REQUEST_SENT
    stamp:
      sec: 1713995389
      nanosec: 386809259
    client_gid: [1, 15, 96, 219, 162, 1, 108, 201, 0, 0, 0, 0, 0, 0, 21, 3]
    sequence_number: 133
  request: [{a: 2, b: 3}]
  response: []
  ---

3 Record services
^^^^^^^^^^^^^^^^^

To record service data, the following options are supported.
Service data can be recorded with topics at the same time.

To record specific services:

.. code-block:: console

  ros2 bag record --service <service_names>

To record all services:

.. code-block:: console

  ros2 bag record --all-services

Run the command:

.. code-block:: console

  ros2 bag record --service /add_two_ints

You will see the following messages in the terminal (the date and time will be different):

.. code-block:: console

  [INFO] [1713995957.643573503] [rosbag2_recorder]: Press SPACE for pausing/resuming
  [INFO] [1713995957.662067587] [rosbag2_recorder]: Event publisher thread: Starting
  [INFO] [1713995957.662067614] [rosbag2_recorder]: Listening for topics...
  [INFO] [1713995957.666048323] [rosbag2_recorder]: Subscribed to topic '/add_two_ints/_service_event'
  [INFO] [1713995957.666092458] [rosbag2_recorder]: Recording...

Now ``ros2 bag`` is recording the service data published on the ``/add_two_ints`` service.
To stop the recording, enter ``Ctrl+C`` in the terminal.

The data will be accumulated in a new bag directory with a name in the pattern of ``rosbag2_year_month_day-hour_minute_second``.
This directory will contain a ``metadata.yaml`` along with the bag file in the recorded format.

4 Inspect service data
^^^^^^^^^^^^^^^^^^^^^^

You can see details about your recording by running:

.. code-block:: console

  ros2 bag info <bag_file_name>

Running this command will return a list of information on the file:

.. code-block:: console

  Files:             rosbag2_2024_04_24-14_59_17_0.mcap
  Bag size:          15.1 KiB
  Storage id:        mcap
  ROS Distro:        rolling
  Duration:          9.211s
  Start:             Apr 24 2024 14:59:17.676 (1713995957.676)
  End:               Apr 24 2024 14:59:26.888 (1713995966.888)
  Messages:          0
  Topic information:
  Service:           1
  Service information: Service: /add_two_ints | Type: example_interfaces/srv/AddTwoInts | Event Count: 78 | Serialization Format: cdr

5 Play service data
^^^^^^^^^^^^^^^^^^^

Before replaying the bag file, enter ``Ctrl+C`` in the terminal where ``introspection_client`` is running.
When ``introspection_client`` stops running, ``introspection_service`` also stops printing the result because there are no incoming requests.

Replaying the service data from the bag file will start sending the requests to ``introspection_service``.

Enter the command:

.. code-block:: console

    ros2 bag play --publish-service-requests <bag_file_name>

The terminal will return the message:

.. code-block:: console

  [INFO] [1713997477.870856190] [rosbag2_player]: Set rate to 1
  [INFO] [1713997477.877417477] [rosbag2_player]: Adding keyboard callbacks.
  [INFO] [1713997477.877442404] [rosbag2_player]: Press SPACE for Pause/Resume
  [INFO] [1713997477.877447855] [rosbag2_player]: Press CURSOR_RIGHT for Play Next Message
  [INFO] [1713997477.877452655] [rosbag2_player]: Press CURSOR_UP for Increase Rate 10%
  [INFO] [1713997477.877456954] [rosbag2_player]: Press CURSOR_DOWN for Decrease Rate 10%
  [INFO] [1713997477.877573647] [rosbag2_player]: Playback until timestamp: -1

Your ``introspection_service`` terminal will once again start printing the following service messages:

.. code-block:: console

  [INFO] [1713997478.090466075] [introspection_service]: Incoming request
  a: 2 b: 3

This is because ``ros2 bag play`` sends the service request data from the bag file to the ``/add_two_ints`` service.

We can also introspect service communication as ``ros2 bag play`` is playing it back to verify the ``introspection_service``.

Run this command before ``ros2 bag play`` to see the ``introspection_service``:

.. code-block:: console

  ros2 service echo --flow-style /add_two_ints

You can see the service request from the bag file and the service response from  ``introspection_service``.

.. code-block:: console

  info:
    event_type: REQUEST_RECEIVED
    stamp:
      sec: 1713998176
      nanosec: 372700698
    client_gid: [1, 15, 96, 219, 80, 2, 158, 123, 0, 0, 0, 0, 0, 0, 20, 4]
    sequence_number: 1
  request: [{a: 2, b: 3}]
  response: []
  ---
  info:
    event_type: RESPONSE_SENT
    stamp:
      sec: 1713998176
      nanosec: 373016882
    client_gid: [1, 15, 96, 219, 80, 2, 158, 123, 0, 0, 0, 0, 0, 0, 20, 4]
    sequence_number: 1
  request: []
  response: [{sum: 5}]

Summary
-------

You can record data passed on topics and services in your ROS 2 system using the ``ros2 bag`` command.
Whether you're sharing your work with others or introspecting your own experiments, it's a great tool to know about.

Next steps
----------

You've completed the "Beginner: CLI Tools" tutorials!
The next step is tackling the "Beginner: Client Libraries" tutorials, starting with :doc:`../../Beginner-Client-Libraries/Creating-A-Workspace/Creating-A-Workspace`.

Related content
---------------

A more thorough explanation of ``ros2 bag`` can be found in the README `here <https://github.com/ros2/rosbag2>`__.
For more information on service recording and playback can be found in the design document `here <https://github.com/ros2/rosbag2/blob/{DISTRO}/docs/design/rosbag2_record_replay_service.md>`__.
For more information on QoS compatibility and ``ros2 bag``, see :doc:`../../../How-To-Guides/Overriding-QoS-Policies-For-Recording-And-Playback`.
