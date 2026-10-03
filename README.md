# Real Time Security

AI-powered real-time security dashboard with face recognition, object detection, and automatic email alerts.

![Real Time Security](static/img/logo.png)

## Features

- Live camera monitoring in a modern web dashboard
- Face recognition (authorized vs unauthorized)
- YOLOv8 object detection
- Emotion analysis
- Automatic email alerts with unauthorized person snapshots
- Dual recipient email support
- Local alert image saving (`alerts/`)
- Auto video recording on alerts (`recordings/`)
- Left navigation UI with Real Time Security branding

## Tech Stack

- Python + Flask
- OpenCV
- face_recognition
- DeepFace
- YOLOv8 (Ultralytics)
- Gmail SMTP (App Password)

## Setup

1. Clone the repo:

```bash
git clone https://github.com/k-v-jaswanth/Real-Time-AI-Security-System.git
cd Real-Time-AI-Security-System
```

2. Install dependencies:

```bash
pip install -r requirements.txt
```

3. Create `.env` from the example:

```bash
copy .env.example .env
```

Set:

```env
SECURITY_EMAIL=realtimesecuritysystem@gmail.com
SECURITY_EMAIL_PASSWORD=your-16-char-app-password
SECURITY_ALERT_RECIPIENT=first@gmail.com
SECURITY_ALERT_RECIPIENT_2=second@gmail.com
```

4. Add authorized faces:

```text
data/authorized_faces/
  person_name/
    photo1.jpg
    photo2.jpg
```

5. Run:

```bash
python main.py
```

Open: [http://127.0.0.1:5000](http://127.0.0.1:5000)

## Email Alerts

When an unauthorized person is detected:

1. Snapshot is saved in `alerts/`
2. Same image is emailed to both configured recipients
3. Optional short video is saved in `recordings/`

Use a Gmail **App Password** (not your normal password):  
https://myaccount.google.com/apppasswords

## Live Public Link (optional, laptop must stay on)

Keep the app running, then in another terminal:

```powershell
powershell -ExecutionPolicy Bypass -File .\start_live.ps1
```

This creates a temporary public URL. It **stops** if the laptop sleeps or shuts down.

## Always Online Website (no laptop needed)

See [DEPLOY_WEBSITE.md](DEPLOY_WEBSITE.md) to host the dashboard on Render.
This keeps the website online even when your laptop is off (camera still needs local PC).

## Always Online with Camera (cloud + IP camera)

See [DEPLOY.md](DEPLOY.md) for VPS + IP/RTSP camera hosting.

## Project Structure

```text
main.py                 # App entry
app.py                  # Flask routes
security_engine.py      # Camera + AI + email alerts
templates/index.html    # Dashboard UI
static/                 # CSS, JS, logo
alerts/                 # Unauthorized snapshots
recordings/             # Alert videos
.env.example            # Email config template
```

## Notes

- Do not commit `.env` (contains secrets)
- Model weights (`.pt`) and runtime DB/logs are ignored

















Based on the repository structure and README you shared, the working architecture is essentially:

```
                         ┌─────────────────────────────┐
                         │       USER / ADMIN           │
                         │  Web Browser / Dashboard    │
                         └──────────────┬──────────────┘
                                        │
                                        │ HTTP
                                        ▼
                         ┌─────────────────────────────┐
                         │        Flask Web App        │
                         │          app.py             │
                         │                             │
                         │  • Dashboard routes         │
                         │  • Live status              │
                         │  • Camera stream            │
                         │  • Alert information        │
                         └──────────────┬──────────────┘
                                        │
                                        ▼
                  ┌─────────────────────────────────────────┐
                  │          SECURITY ENGINE                │
                  │        security_engine.py               │
                  │                                         │
                  │  Camera → Frame Processing → AI         │
                  └──────────────┬──────────────────────────┘
                                 │
                                 ▼
                    ┌────────────────────────┐
                    │      OpenCV Camera      │
                    │                         │
                    │ Webcam / IP Camera      │
                    └───────────┬────────────┘
                                │
                                │ Video Frames
                                ▼
              ┌──────────────────────────────────────────────┐
              │              AI PROCESSING PIPELINE           │
              │                                              │
              │  ┌──────────────┐                            │
              │  │ Face Detect  │                            │
              │  └──────┬───────┘                            │
              │         ▼                                    │
              │  ┌─────────────────┐                         │
              │  │ Face Recognition│                         │
              │  │ face_recognition│                         │
              │  └──────┬──────────┘                         │
              │         │                                    │
              │         ▼                                    │
              │  Authorized? ───────────────┐                │
              │         │                   │                │
              │       YES                  NO                │
              │         │                   │                │
              │         ▼                   ▼                │
              │    Normal State       SECURITY ALERT        │
              │                             │                │
              │                             ▼                │
              │                    ┌────────────────┐        │
              │                    │ Save Snapshot  │        │
              │                    │   alerts/      │        │
              │                    └───────┬────────┘        │
              │                            │                 │
              │                            ▼                 │
              │                    ┌────────────────┐        │
              │                    │ Record Video   │        │
              │                    │ recordings/    │        │
              │                    └───────┬────────┘        │
              │                            │                 │
              │                            ▼                 │
              │                    ┌────────────────┐        │
              │                    │ Email Alert    │        │
              │                    └───────┬────────┘        │
              └────────────────────────────┼─────────────────┘
                                           │
                         ┌─────────────────┴─────────────────┐
                         ▼                                   ▼
                ┌──────────────────┐                ┌──────────────────┐
                │ Recipient 1      │                │ Recipient 2      │
                │ SECURITY_ALERT_  │                │ SECURITY_ALERT_  │
                │ RECIPIENT        │                │ RECIPIENT_2      │
                └──────────────────┘                └──────────────────┘
```

 ## Complete working flow

 ### 1\. Camera input

 The system starts from:

```
Webcam / IP Camera
       ↓
OpenCV
       ↓
Continuous video frames
```

 `security_engine.py` is the main component responsible for camera and security processing.

---

 ### 2\. Frame processing

 Each camera frame is passed through the AI pipeline.

```
Camera Frame
     │
     ├── Face Detection
     │
     ├── Face Recognition
     │
     ├── YOLOv8 Object Detection
     │
     └── Emotion Analysis
```

 The repository uses:

 - **OpenCV** — camera/video processing
- **face\_recognition** — identifying authorized faces
- **YOLOv8 / Ultralytics** — object detection
- **DeepFace** — emotion analysis

---

 ### 3\. Authorized-face recognition

 The system has an authorized-face database:

```
data/
└── authorized_faces/
    ├── person_1/
    │   ├── photo1.jpg
    │   └── photo2.jpg
    │
    └── person_2/
        ├── photo1.jpg
        └── photo2.jpg
```

 At runtime:

```
Detected Face
      ↓
Generate Face Encoding
      ↓
Compare with Authorized Faces
      ↓
   ┌──┴───┐
   │      │
Match   No Match
   │      │
   ▼      ▼
Known   Unknown
Person  Person
```

---

 ## 4\. Security decision

 The important control point is:

```
                Camera Frame
                     │
                     ▼
              Face Detection
                     │
                     ▼
             Face Recognition
                     │
              ┌──────┴──────┐
              │             │
        Authorized      Unauthorized
              │             │
              ▼             ▼
        Continue       SECURITY EVENT
```

 An unauthorized person causes the alert workflow.

---

 ## 5\. Alert workflow

 When an unauthorized person is detected:

```
Unauthorized Person
        │
        ├───────────────┐
        │               │
        ▼               ▼
Save Snapshot      Start/Save Video
  alerts/             recordings/
        │               │
        └───────┬───────┘
                ▼
          Email Service
                │
          ┌─────┴─────┐
          ▼           ▼
     Recipient 1  Recipient 2
```

 The same unauthorized-person snapshot can therefore reach both configured recipients.

---

 ## 6\. Dashboard flow

 The frontend is roughly:

```
Browser
   │
   │ HTTP
   ▼
Flask
app.py
   │
   ├── templates/index.html
   │
   ├── static/
   │    ├── CSS
   │    └── JavaScript
   │
   └── Security Engine
             │
             ▼
          Camera
```

 So the browser is **not directly running the AI models**.

 The architecture is:

```
Browser
   ↓
Flask Backend
   ↓
Security Engine
   ↓
OpenCV + AI Models
   ↓
Camera
```

 The processed information/video is then exposed back to the dashboard.

---

 ## 7\. Main software architecture

 A useful way to explain the project in a presentation is:

```
┌────────────────────────────────────────────────────┐
│                    PRESENTATION                    │
│                                                    │
│              HTML + CSS + JavaScript               │
│                    Dashboard                        │
└────────────────────────┬───────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────┐
│                    WEB LAYER                       │
│                                                    │
│                    Flask                           │
│                    app.py                          │
└────────────────────────┬───────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────┐
│                  AI / SECURITY                     │
│                                                    │
│              security_engine.py                    │
│                                                    │
│  OpenCV │ Face Recognition │ YOLOv8 │ DeepFace     │
└────────────────────────┬───────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────┐
│                    INPUT                           │
│                                                    │
│               Webcam / IP Camera                   │
└────────────────────────┬───────────────────────────┘
                         │
                         ▼
┌────────────────────────────────────────────────────┐
│                   OUTPUT                           │
│                                                    │
│  Dashboard │ Snapshot │ Video │ Email Alerts       │
└────────────────────────────────────────────────────┘
```

 ## 8\. Deployment architecture

 The repository also has **two deployment modes**.

 ### Local mode

```
Laptop
 │
 ├── Camera
 │
 ├── Python
 │    └── Flask
 │         └── AI Engine
 │
 └── Browser
      └── localhost:5000
```

 The laptop needs to remain running.

 ### Cloud website mode

```
                 Internet
                    │
                    ▼
             ┌─────────────┐
             │   Render    │
             │             │
             │ Flask Web   │
             │ Dashboard   │
             └──────┬──────┘
                    │
                    │
              Website only
                    │
                    ▼
                 Browser
```

 But there's an important distinction in your README:

 **Putting the Flask dashboard on Render does not magically give the cloud server access to your laptop webcam.**

 For actual always-online camera monitoring, the repository describes a VPS + IP/RTSP camera architecture:

```
              IP / RTSP CAMERA
                      │
                      │ Video Stream
                      ▼
              ┌───────────────┐
              │ VPS / Cloud   │
              │               │
              │ OpenCV        │
              │ Face AI       │
              │ YOLOv8        │
              │ DeepFace      │
              └───────┬───────┘
                      │
             ┌────────┴─────────┐
             ▼                  ▼
        Web Dashboard       Email Alerts
             │                  │
             ▼             ┌────┴────┐
          Admin            ▼         ▼
                         Email 1   Email 2
```

 ### In one sentence

 The project's core working flow is:

 **Camera → OpenCV → Face Recognition / YOLOv8 / Emotion Analysis → Security Decision → Unauthorized Snapshot/Video → Dual Email Alert → Flask Live Dashboard.**

 For a **college/project presentation**, this can be condensed into a clean **7-block architecture diagram**: **Camera → Preprocessing → AI Detection → Face Authentication → Threat Decision → Alert System → Web Dashboard**.
