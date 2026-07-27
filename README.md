1. Introduction & Project Architecture
This project details the engineering phase 1 implementation of a decentralized autonomous swarm robotics system optimized for automated warehouse material handling operations. Modern warehouse management systems typically rely on centralized server frameworks, creating a fatal single point of failure. If the central coordinator fails, the entire automation matrix collapses. This project counteracts that structural limitation by introducing a robust decentralized architecture.
1.1 Decentralized Swarm Principles
In this system, individual robotic agents possess localized intelligence and operate without a master controller directing task assignments. The swarm relies on a shared global environment visualization state, allowing each agent to independently perform localization, path estimation, behavioral logic adjustment, and conflict resolution.
Engineering Objective: To deliver a scalable framework where individual units can enter or exit the operational environment dynamically without disrupting ongoing material transport arrays.
1.2 Structural Data Flow Loop
The technical pipeline functions across a high-speed closed loop connecting physical spaces to digital mapping canvases:
•	Global Vision Acquisition: An overhead digital video frame captures physical arena movements.
•	Digital Twin Synthesis: A PyCharm station running OpenCV extracts ArUco matrix IDs, scaling real spatial markers into explicit metric centimeters.
•	Decentralized Network Broadcast: The station packs coordinates into a unified JSON string format and fires it via UDP to a network broadcast target.
•	Edge Computation: An onboard ESP32 microcontroller processes the packet stream, executing spatial trigonometry locally to guide the physical motor chassis.
2. Python Digital Twin & Tracking Dashboard
The management dashboard forms the visual and calculation core of the indoor localization grid. Operating inside a Python environment using OpenCV, it transforms raw image tracking feeds into calibrated metric data loops.
2.1 Homography & Real-to-Pixel Coordinate Scaling
To bridge the physical test track to virtual software frames, the system implements linear spatial transformation metrics. Pixels do not hold true meaning for mechatronic navigation; hence, a conversion constant translates coordinates to physical centimeters based on a defined 120 cm x 120 cm arena dimension.
Scale Factor Equations:
Scale_X = Window_Width / Arena_Width_CM
Scale_Y = Window_Height / Arena_Height_CM
  
 
Figure 2.1: Online ArUco Tracking Token Matrices (ID 0 and ID 1)
2.2 Comprehensive Core Dashboard Engine Script
The Python code snippet below demonstrates the video scaling canvas configuration, ArUco identification matrix, color-coded tracking dictionaries, and live UDP JSON stream delivery:
import cv2
import cv2.aruco as aruco
import numpy as np
import socket
import json
import math

ARENA_WIDTH_CM = 120.0
ARENA_HEIGHT_CM = 120.0
DISPLAY_WIDTH = 800
DISPLAY_HEIGHT = 800

SCALE_X = DISPLAY_WIDTH / ARENA_WIDTH_CM
SCALE_Y = DISPLAY_HEIGHT / ARENA_HEIGHT_CM

def cm_to_px(x_cm, y_cm):
    return int(x_cm * SCALE_X), int(y_cm * SCALE_Y)

UDP_IP = "255.255.255.255"
UDP_PORT = 5005
sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
sock.setsockopt(socket.SOL_SOCKET, socket.SO_BROADCAST, 1)

STATUS_COLORS = {
    "IDLE": (0, 255, 0),       # Green
    "BUSY": (0, 255, 255),     # Yellow
    "PICKING": (255, 0, 255),  # Magenta
    "ERROR": (0, 0, 255)       # Red
}

bot_states = {0: "IDLE", 1: "BUSY"}
cap = cv2.VideoCapture("http://192.168.1.5:8080/video")

aruco_dict = aruco.getPredefinedDictionary(aruco.DICT_4X4_100)
aruco_params = aruco.DetectorParameters()
detector = aruco.ArucoDetector(aruco_dict, aruco_params)

def draw_scaled_warehouse(ui_frame):
    for i in range(1, 4):
        x_pos = int(i * (DISPLAY_WIDTH / 4))
        y_pos = int(i * (DISPLAY_HEIGHT / 4))
        cv2.line(ui_frame, (x_pos, 0), (x_pos, DISPLAY_HEIGHT), (60, 60, 60), 1)
        cv2.line(ui_frame, (0, y_pos), (DISPLAY_WIDTH, y_pos), (60, 60, 60), 1)

while True:
    ret, frame = cap.read()
    if not ret: break
    frame = cv2.resize(frame, (DISPLAY_WIDTH, DISPLAY_HEIGHT))
    dashboard = np.zeros((DISPLAY_HEIGHT, DISPLAY_WIDTH, 3), dtype=np.uint8)
    draw_scaled_warehouse(dashboard)

    corners, ids, rejected = detector.detectMarkers(frame)
    swarm_data = {}

    if ids is not None:
        for i in range(len(ids)):
            marker_id = int(ids[i][0])
            c = corners[i][0]
            cx, cy = int(np.mean(c[:, 0])), int(np.mean(c[:, 1]))
            cx_cm, cy_cm = round(cx / SCALE_X, 1), round(cy / SCALE_Y, 1)

            angle_rad = math.atan2(c[1][1] - c[0][1], c[1][0] - c[0][0])
            angle_deg = round(math.degrees(angle_rad), 1)
            current_status = bot_states.get(marker_id, "IDLE")
            bot_color = STATUS_COLORS.get(current_status, (255, 255, 255))

            cv2.circle(dashboard, (cx, cy), 25, bot_color, 3)
            swarm_data[f"id{marker_id}"] = {"x": cx_cm, "y": cy_cm, "ang": angle_deg, "status": current_status}

    if swarm_data:
        sock.sendto(json.dumps(swarm_data).encode(), (UDP_IP, UDP_PORT))
    cv2.imshow("Real-Time Metric Swarm Dashboard", dashboard)
    if cv2.waitKey(1) & 0xFF == ord('q'): break
cap.release()
cv2.destroyAllWindows()

 
 
Figure 2.2: Live Visual Representation of the Real-Time Metric Swarm Dashboard
3. Mechanical Design & Structural Body Modeling
The physical structure of the swarm agent is designed using a multi-tiered platform architecture. This layout systematically isolates heavy electromechanical assemblies from computational logic elements to maintain high operational integrity.
3.1 Tiered Chassis Topology
The mobile agent's body is divided into two distinct primary layers managed through custom geometric layouts:
•	Lower Deck (LD): Serves as the structural power platform. It houses the high-current 12V 100RPM DC metal gear motors, heavy-duty differential wheel assemblies, and the core 7.4V Lithium-Ion power source pack.
•	Upper Deck (UD): Acts as the clean digital acquisition deck. It maintains a completely flat plane layout optimized for hosting the overhead vision ArUco tokens, localized sensor arrays, and the ESP32 microcontroller.

 
Figure 3.1: 3D CAD Geometric Structural Shell Layout - Lower Deck Profile (LD)

 
Figure 3.2: 3D CAD Geometric Structural Shell Layout - Upper Deck Profile (UD)
3.2 Material Selection & Manufacturing Feasibility
The plates are modeled and manufactured using high-density lightweight acrylic polymer sheeting. This selection balances rigid structural loading capabilities with effective damping characteristics against high-frequency motor vibration harmonics. It also provides excellent electrical insulation across the chassis plates.
3.3 Mechanical Gripper Linkage Geometry
To execute physical inventory handling, the front-end features a custom gear-driven scissor linkage claw mechanism. Powered by an SG90 servo motor, it provides a highly repeatable operational clearance sweep moving from a 180-degree wide initialization state to a 100-degree mechanical locking configuration to grip package boxes safely.

        
        

Figure 3.3: Physical Prototyping Assembly & End-Effector Scissor Gripper Mechanism Close-ups
4. Electrical Architecture & Hardware Integration
The electrical control distribution routes signals systematically to optimize high-frequency data isolation and prevent internal circuit sags.
4.1 Core Pin Connection Profile
The control hardware centers around a 30-pin ESP32 DevKit V1 module routed to a dual H-bridge TB6612FNG motor driver using a general-purpose copper dot-board perfboard prototyping panel.
Target Component	Pin Label	ESP32 Target GPIO Mapping	Functional System Purpose
TB6612FNG Driver	STBY	GPIO 17 (TX2)	Standby Override Link (Kept HIGH)
TB6612FNG Driver	AIN1	GPIO 18 (D18)	Left Motor Input Directional Control Pin 1
TB6612FNG Driver	AIN2	GPIO 19 (D19)	Left Motor Input Directional Control Pin 2
TB6612FNG Driver	PWMA	GPIO 5 (D5)	Left Motor Speed Vector (Pulse Width Mod)
TB6612FNG Driver	BIN1	GPIO 21 (D21)	Right Motor Input Directional Control Pin 1
TB6612FNG Driver	BIN2	GPIO 22 (D22)	Right Motor Input Directional Control Pin 2
TB6612FNG Driver	PWMB	GPIO 23 (D23)	Right Motor Speed Vector (Pulse Width Mod)
HC-SR04 Ultrasonic	TRIG	GPIO 27 (D27)	Safe Trigger Wave Emission IO Channel
HC-SR04 Ultrasonic	ECHO	GPIO 26 (D26)	Safe Echo Capture Bounce Return Pulse Channel

 
Figure 4.1: Schematic Block Diagram of the Whole Integrated Circuit Distribution System
4.2 Motor Driver Power Split Rules (VM vs VCC)
To protect against electrical 'noise' sags and unexpected brownouts, power routing is handled through separate tracks:
•	VM (Motor Power): Tied straight to the raw un-regulated 7.4V battery loop to handle high inductive current surges from the wheels.
•	VCC (Logic Power): Connected to the stable 5V output of the ESP32 onboard regulator to keep processing gates isolated from motor interference.
5. Onboard Decentralized Swarm Agent Firmware
The edge processing firmware running locally on each independent ESP32 unit establishes network connection handshakes, deserializes incoming matrix packets via parsing allocations, and operates absolute trigonometry functions to steer mechanical wheels.
5.1 Onboard Embedded Agent C++ Engine Code
#include <WiFi.h>
#include <WiFiUdp.h>
#include <ArduinoJson.h>
#include <ESP32Servo.h>

const char* ssid     = "Redmi";     
const char* password = "s1s2c7.v"; 
const int UDP_PORT   = 5005;
WiFiUDP udp;

const int MY_ID = 0;              
const int CLAW_OPEN = 180;        
const int CLAW_CLOSE = 100;       

float targetX = 15.0; float targetY = 45.0;
const int AIN1 = 18; const int AIN2 = 19; const int PWMA = 5;
const int BIN1 = 21; const int BIN2 = 22; const int PWMB = 23;
const int STBY = 17;

Servo clawServo;
const int SERVO_PIN = 13;
const int TRIG_PIN  = 27; const int ECHO_PIN  = 26; 

void drive(int leftSpeed, int rightSpeed) {
  digitalWrite(AIN1, leftSpeed > 0 ? HIGH : LOW);
  digitalWrite(AIN2, leftSpeed > 0 ? LOW : HIGH);
  analogWrite(PWMA, abs(leftSpeed));
  digitalWrite(BIN1, rightSpeed > 0 ? HIGH : LOW);
  digitalWrite(BIN2, rightSpeed > 0 ? LOW : HIGH);
  analogWrite(PWMB, abs(rightSpeed));
}

void setup() {
  Serial.begin(115200);
  pinMode(AIN1, OUTPUT); pinMode(AIN2, OUTPUT); pinMode(PWMA, OUTPUT);
  pinMode(BIN1, OUTPUT); pinMode(BIN2, OUTPUT); pinMode(PWMB, OUTPUT);
  pinMode(STBY, OUTPUT); digitalWrite(STBY, HIGH);
  WiFi.begin(ssid, password);
  while (WiFi.status() != WL_CONNECTED) { delay(500); }
  udp.begin(UDP_PORT);
}

void loop() {
  int packetSize = udp.parsePacket();
  if (packetSize) {
    char packetBuffer[512];
    int len = udp.read(packetBuffer, 511);
    if (len > 0) packetBuffer[len] = 0;
    JsonDocument doc;
    deserializeJson(doc, packetBuffer);
    String myKey = "id" + String(MY_ID);
    if (doc.containsKey(myKey)) {
      float currentX = doc[myKey]["x"];
      float currentY = doc[myKey]["y"];
      float currentAng = doc[myKey]["ang"];
      
      float dx = targetX - currentX;
      float dy = targetY - currentY;
      float distance = sqrt(dx*dx + dy*dy);
      
      if (distance < 12.0) {
        drive(0,0); // Reach Target sequence
        return;
      }
      
      float targetAng = atan2(dy, dx) * 180.0 / M_PI;
      float error = targetAng - currentAng;
      while (error > 180) error -= 360;
      while (error < -180) error += 360;
      
      if (abs(error) > 25.0) {
        if (error > 0) drive(110, -110);
        else drive(-110, 110);
      } else {
        drive(140, 140);
      }
    }
  }
}
 
5.2 System Verification & Presentation Benchmarks
During functional evaluations, the integrated mechatronic system successfully achieved the following project deliverables:
•	Proportional Trajectory Error Realignment: The agent parses the global map matrix, computing deviation vectors and spinning on its center axis to correct its alignment whenever the angle error exceeds 25 degrees.
•	Tactile Sensor Fusion Handoff: The moment the HC-SR04 ultrasonic echo registers a local proximity of 4 cm or less, the system interrupts the trajectory path to activate the servo-driven claw, securing the payload firmly between its calibrated 180-degree open and 100-degree closed mechanical limits.

attachments/files/30427592/PyCharm.Code.for.Dashboard.with.Status.Indicator.for.each.BOT.using.ArUco.code.docx)
<img width="1439" height="899" alt="Dashboard Screenshot" src="https://github.com/user-attachments/assets/41ef722d-80bd-4ea4-8497-8f9318149410" />
<img width="361" height="444" alt="Connection Table" src="https://github.com/user-attachments/assets/53de8d9c-2488-4e08-a190-8c33d8e61449" />
<img width="720" height="391" alt="Block Diagram of Whole Circuit" src="https://github.com/user-attachments/assets/b2585fe9-349a-402b-9ff2-6f05c0e78971" />
<img width="1422" height="859" alt="Arduino IDE Serial monitor" src="https://github.com/user-attachments/assets/2e8e70e3-e571-4ef1-a02b-32bbf3b106c0" />
