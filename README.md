# Yolo-Object-Detection
Perfect — you’ve spotted exactly the key difference between Linux Mint and Raspberry Pi OS Bookworm 👏

Your observation is 100% correct — the YOLOv5 + OpenCV script that works fine on your PC won’t open the camera on Bookworm unless you modify it to use the new Picamera2 (libcamera) system.

Let’s go over why this happens and then I’ll show you how to make your YOLO + TTS code fully compatible with Bookworm’s new camera system 👇


---

##🧩 WHY your PC code doesn’t show the camera on Pi OS Bookworm

Raspberry Pi OS Bookworm replaced the old “legacy camera stack” (which worked with cv2.VideoCapture(0)) with a new libcamera-based system.

- On Bookworm, /dev/video0 may not exist for the CSI (ribbon) camera.

- As a result, cv2.VideoCapture(0) works only for USB webcams, not the official Pi Camera Module.

- The correct way now is to use the Picamera2 library — which integrates with OpenCV perfectly.



---

##✅ FIX: YOLOv5 + Picamera2 + pyttsx3 (Bookworm-compatible code)

Here’s how to update your working Linux code to run on Raspberry Pi OS Bookworm, using Picamera2 to get frames instead of OpenCV capture.


---

##🧰 Step 1 — Make sure everything is installed

Run:
```
sudo apt update
sudo apt install -y python3-picamera2 python3-opencv espeak
pip install torch torchvision torchaudio pyttsx3 ultralytics --break-system-packages
```

---

##💻 Step 2 — Create the new Python file

Create:
```
nano object_voice_detect_picam2.py
```
Paste this Bookworm-compatible version:
```
from picamera2 import Picamera2
import cv2
import torch
import pyttsx3
import time

# Initialize TTS
engine = pyttsx3.init()
engine.setProperty('rate', 170)
engine.setProperty('volume', 1.0)

# Load YOLOv5 model
print("Loading YOLOv5 model...")
model = torch.hub.load('ultralytics/yolov5', 'yolov5n', pretrained=True)
print("Model loaded successfully.")

# Initialize PiCamera2
picam2 = Picamera2()
picam2.preview_configuration.main.size = (640, 480)
picam2.preview_configuration.main.format = "RGB888"
picam2.configure("preview")
picam2.start()

last_spoken = ""
last_time = time.time()

print("Starting detection + voice output... Press Q to quit.")

while True:
    # Capture a frame from Picamera2
    frame = picam2.capture_array()

    # Run YOLO detection
    results = model(frame)
    detected_objects = results.pandas().xyxy[0]['name'].unique().tolist()

    # Display annotated frame
    annotated_frame = results.render()[0]
    cv2.imshow("YOLOv5 Object Detection (Picamera2)", annotated_frame)

    # Speak detected objects every 3 seconds
    current_time = time.time()
    if detected_objects and (current_time - last_time > 3):
        to_speak = ", ".join(detected_objects)
        if to_speak != last_spoken:
            print("Detected:", to_speak)
            engine.say(f"I see {to_speak}")
            engine.runAndWait()
            last_spoken = to_speak
            last_time = current_time

    if cv2.waitKey(1) & 0xFF == ord('q'):
        break

cv2.destroyAllWindows()
engine.stop()
print("Detection stopped.")
```

---

##▶️ Step 3 — Run it
```
python3 object_voice_detect_picam2.py
```
##✅ You should now see:

- A live preview window with YOLO labels

- The Pi speaking detected object names every few seconds


Press Q to quit.


---

##⚡ Optional Performance Tips

Setting	Change	Effect

Lower resolution	(320, 240) instead of (640, 480)	+30–50% FPS
Hide preview window	comment out cv2.imshow()	save CPU for TTS
Model choice	yolov5n or yolov5s	smaller = faster
Delay between speech	change > 3 to > 5	less CPU usage



---

##🧠 Summary

Feature	OpenCV-only code	Bookworm fixed (Picamera2)

Works with USB camera	✅	✅
Works with Pi Camera Module	❌	✅
Compatible with Bookworm	❌	✅
Uses new libcamera backend	❌	✅



---

Would you like me to give you a headless version (no preview window, only voice output) — perfect for when you’ll run this on the Pi without monitor (like for your smart glasses)?
