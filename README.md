#Computer-Vision Assisted Swarm Mobile Robot Platform with Distributed UDP Messaging and Articulated Manipulator
  This repository contains the Phase 1 engineering implementation and design dossier for a computer-vision assisted, decentralized autonomous swarm robotics system optimized for automated warehouse material handling.

  Developed by the four-member team Swarmobotics and presented to the Department of Mechatronics Engineering (Continuous Internal Evaluation - Phase 1, May 2026), this project aims to counteract the fatal single point of failure common in centralized warehouse management systems by distributing intelligence across localized agents.

##**System Architecture**
    The swarm relies on a shared global environment visualization state, enabling independent localization, path estimation, and conflict resolution without a master controller. The data flow loop consists of:

###**Global Vision Acquisition**: An overhead digital video frame captures all physical movements within the arena.

###**Digital Twin Synthesis**: A PyCharm station running OpenCV extracts ArUco matrix IDs, scaling real spatial markers into metric centimeters.

###**Decentralized Network Broadcast**: The station packs the coordinate data into a unified JSON string and transmits it via a UDP broadcast.

###**Edge Computation**: Onboard ESP32 microcontrollers process the incoming packet stream and execute local spatial trigonometry to drive the motor chassis.

##**Python Digital Twin & Tracking Dashboard**
    The system's visual and calculation core operates inside a Python environment to transform raw image feeds into calibrated metric data loops.

###**Spatial Transformation**: Bridges the physical track to virtual frames using linear spatial transformation metrics based on a defined 120 cm x 120 cm arena.

###**Dashboard Features**: The script utilizes color-coded tracking dictionaries (e.g., Green for IDLE, Yellow for BUSY), ArUco identification matrices (DICT_4X4_100), and live UDP JSON stream delivery to the robotic agents.

##**Mechanical Design & Structure**
    The mobile agents feature a multi-tiered platform architecture manufactured from high-density lightweight acrylic polymer sheeting, which provides structural rigidity, electrical insulation, and vibration damping.

###**Lower Deck (LD):** The structural power platform housing the 7.4V Lithium-Ion battery, high-current 12V 100RPM DC metal gear motors, and heavy-duty differential wheel assemblies.

###**Upper Deck (UD):** The digital acquisition deck containing the ESP32 microcontroller, localized sensor arrays, and overhead ArUco tokens.

###**End-Effector Gripper**: A custom gear-driven scissor linkage claw mechanism at the front of the bot. It is powered by an SG90 servo motor that sweeps from a 180-degree initialization state to a 100-degree mechanical lock to safely grip packages.

##**Electrical Architecture**
    The electrical control distribution isolates high-frequency data from internal circuit sags using a general-purpose copper perfboard panel.

###**Core Hardware**: Driven by a 30-pin ESP32 DevKit V1 module routed to a dual H-bridge TB6612FNG motor driver.

###**Power Split Rule:**

###**VM (Motor Power):** Tied directly to the unregulated 7.4V battery to handle high inductive current surges from the DC motors.

###**VCC (Logic Power):** Connected to the stable 5V output of the ESP32 onboard regulator to protect processing gates from motor interference.

##**Firmware & Operational Benchmarks**
    The edge processing firmware is written in C++ and runs locally on each ESP32 unit. The firmware establishes WiFi/UDP connections, deserializes matrix packets, and operates absolute trigonometry functions.

###**During physical evaluations, the system achieved the following performance benchmarks:**

###**Proportional Trajectory Error Realignment:** The agent computes deviation vectors from the global map matrix and spins on its center axis to correct its alignment if the angle error exceeds 25 degrees.

###**Tactile Sensor Fusion Handoff:** An onboard HC-SR04 ultrasonic sensor actively monitors local proximity. When an echo registers at 4 cm or less, the system interrupts the trajectory path and activates the servo-driven claw to secure the payload firmly.
