#**Computer-Vision Assisted Swarm Mobile Robot Platform with Distributed UDP Messaging and Articulated Manipulator**
This repository contains the FYP engineering implementation and design dossier for a computer-vision assisted, decentralized autonomous swarm robotics system optimized for automated warehouse material handling.

Developed by the four-member team Swarmobotics and presented to the Department of Mechatronics Engineering, this project aims to counteract the fatal single point of failure common in centralized warehouse management systems by distributing intelligence across localized agents.

System Architecture
The swarm relies on a shared global environment visualization state, enabling independent localization, path estimation, and conflict resolution without a master controller. The data flow loop consists of:

Global Vision Acquisition: An overhead digital video frame captures all physical movements within the arena using an iPhone broadcasting via an HTTP IP Camera server.

Digital Twin Synthesis: A PyCharm station running OpenCV extracts ArUco matrix IDs (DICT_4X4_250), calculating real-time spatial coordinates and directional headings for both robots and dynamic targets.

Decentralized Network Broadcast: The station packs the coordinate data into a lightweight, comma-separated telemetry string (ID,X,Y,Angle) and transmits it via a UDP broadcast over a 2.4GHz hotspot bridge.

Edge Computation: Onboard ESP32 microcontrollers process the incoming packet stream, update dynamic target coordinates (e.g., Rack 4), and execute local spatial trigonometry to drive the 4WD motor chassis.

Python Digital Twin & Tracking Dashboard
The system's visual and calculation core operates inside a Python environment to transform raw image feeds into calibrated metric data loops.

Spatial Tracking & Targeting: Bridges the physical track to virtual frames, actively monitoring both the mobile agents (Bot 0) and dynamic destinations (Rack 4) to calculate relative distances.

Dashboard Features: The script utilizes ArUco identification matrices, overlays live heading vectors and bounding boxes, draws a real-time targeting line between the bot and its destination, and delivers live UDP telemetry streams to the robotic agents.

Mechanical Design & Structure
The mobile agents feature a multi-tiered platform architecture manufactured from high-density lightweight acrylic polymer sheeting, which provides structural rigidity, electrical insulation, and vibration damping.

Lower Deck (LD): The structural power platform housing the 7.4V Lithium-Ion battery, four high-current 12V 100RPM DC metal gear motors, and heavy-duty wheel assemblies for 4WD operation.

Upper Deck (UD): The digital acquisition deck containing the ESP32 microcontroller, localized sensor arrays, and overhead ArUco tokens.

End-Effector Gripper: A custom gear-driven scissor linkage claw mechanism at the front of the bot, powered by an SG90 servo motor designed to sweep from a 180-degree initialization state to a 100-degree mechanical lock to safely grip packages.

Electrical Architecture
The electrical control distribution isolates high-frequency data from internal circuit sags using a general-purpose copper perfboard panel.

Core Hardware: Driven by a 30-pin ESP32 DevKit V1 module routed to a dual H-bridge TB6612FNG motor driver configuration to handle all four wheels synchronously.

Power Split Rule:

VM (Motor Power): Tied directly to the unregulated 7.4V battery to handle high inductive current surges from the DC motors.

VCC (Logic Power): Connected to the stable 3.3V/5V logic output to protect processing gates from motor interference, with the TB6612FNG STBY pins pulled high for continuous operation.

Firmware & Operational Benchmarks
The edge processing firmware is written in C++ and runs locally on each ESP32 unit. The firmware establishes Wi-Fi/UDP connections, deserializes coordinate packets, and operates absolute trigonometry functions.

During physical evaluations, the system has achieved the following performance benchmarks:

Communication Failsafe Loop: Implemented a strict 500ms safety timeout that instantly halts all motor actuation if the agent's ArUco marker is occluded or the camera stream drops.

Dynamic Target Locking: The system successfully transitioned from hardcoded coordinate navigation to dynamic tracking, allowing the bot to adjust its path in real-time as the target rack is moved across the arena.

Proportional Trajectory Error Realignment: The agent computes deviation vectors using inverted Y-axis kinematics (accounting for OpenCV's top-left origin). It spins on its center axis to correct its alignment if the angle error exceeds 20 degrees, then drives synchronously toward the target.

Next Steps (In Development)
Tactile Sensor Fusion Handoff: Integrating the onboard HC-SR04 ultrasonic sensor to actively monitor local proximity. When an echo registers at 4 cm or less from the targeted rack, the system will interrupt the visual trajectory path and activate the servo-driven claw to secure the payload.
