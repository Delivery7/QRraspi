import tkinter as tk
import random
import qrcode
from PIL import Image, ImageTk
import firebase_admin
from firebase_admin import credentials, db
import serial
import time
import RPi.GPIO as GPIO  # Library untuk GPIO Raspberry Pi

# GPIO setup for buzzer
BUZZER_PIN = 18  # Ganti dengan pin GPIO yang digunakan untuk buzzer
GPIO.setmode(GPIO.BCM)
GPIO.setup(BUZZER_PIN, GPIO.OUT)

# Firebase setup
try:
    cred = credentials.Certificate('qr-baru-firebase-adminsdk-ofxtt-0a2f2c31b8.json')
    firebase_admin.initialize_app(cred, {
        'databaseURL': 'https://qr-baru-default-rtdb.asia-southeast1.firebasedatabase.app/'
    })
    firebase_connected = True
except Exception as e:
    print("Firebase initialization error:", e)
    firebase_connected = False

# Serial setup (pyserial) - Replace with the correct port
try:
    esp32 = serial.Serial('/dev/ttyUSB0', 115200, timeout=1)  # Port di Raspberry Pi
    serial_connected = True
except Exception as e:
    print("Serial connection error:", e)
    serial_connected = False

# Global variables
random_number = None
new_window = None

# Function to buzz for a specific duration
def buzz(duration):
    GPIO.output(BUZZER_PIN, GPIO.HIGH)
    time.sleep(duration)
    GPIO.output(BUZZER_PIN, GPIO.LOW)

# Generate QR code and send data to Firebase
def generate_qr():
    global random_number
    random_number = random.randint(100000, 999999)
    qr = qrcode.QRCode(box_size=10, border=4)
    qr.add_data(str(random_number))
    qr.make(fit=True)
    qr_image = ImageTk.PhotoImage(qr.make_image(fill_color="black", back_color="white"))
    qr_label.config(image=qr_image)
    qr_label.image = qr_image

    # Reset status label to "Waiting for Scan" when generating new QR
    status_label.config(text="Waiting For Scan", fg="#FF7D2D")

    if firebase_connected:
        db.reference('/Data').update({'QR Generator': random_number})
    else:
        print("Firebase not connected.")

    # Set the next QR generation after 10 seconds
    new_window.after(10000, generate_qr)

# Real-time Firebase listener
def setup_realtime_listener():
    def callback(event):
        try:
            data = event.data
            print("QR Scanner:", data)

            if isinstance(data, str):
                # Check if the received data matches
                if data == str(random_number):  # Ensure data matches
                    status_label.config(text="Verified! ✅", fg="green")
                    
                    # Buzz and send 'true' to ESP32
                    buzz(0.5)  # Buzzer aktif selama 0.5 detik
                    if serial_connected:
                        esp32.write(b'true\n')  # Send signal "true" to ESP32
                else:
                    status_label.config(text="Waiting For Scan", fg="#FF7D2D")
            else:
                status_label.config(text="Invalid Data Format", fg="red")
        except Exception as e:
            print("Error in listener:", e)
            status_label.config(text="Error in Listener!", fg="red")

    if firebase_connected:
        db.reference('/Data/QR_Scanner').listen(callback)
    else:
        status_label.config(text="Firebase not connected!", fg="red")

# Open new window for QR Generator
def open_new_window():
    global new_window
    
    # If window already exists, close it
    if new_window is not None and new_window.winfo_exists():
        new_window.destroy()

    # Hide the main window (Delivery Robot)
    window.withdraw()

    # Create new window
    new_window = tk.Toplevel(window)
    new_window.title("QR Generator")
    
    # Set the QR Generator window to fullscreen
    new_window.attributes('-fullscreen', True)
    new_window.configure(bg="#292929")
    
    # Header
    tk.Label(
        new_window, 
        text="QR Generator", 
        font=("Roboto", 30, "bold"), 
        fg="#FF7D2D", 
        bg="#292929"
    ).pack(pady=40)

    # QR Code Label
    global qr_label
    qr_label = tk.Label(new_window, bg="#292929")
    qr_label.pack(pady=20)

    # Random Number Display
    global random_number_label
    random_number_label = tk.Label(new_window, font=("Roboto", 18), fg="#FFFFFF", bg="#292929")
    random_number_label.pack(pady=10)

    # Status Label
    global status_label
    status_label = tk.Label(
        new_window, 
        text="Waiting for Data...", 
        font=("Roboto", 18), 
        fg="#FF7D2D", 
        bg="#292929"
    )
    status_label.pack(pady=10)

    # Setup Firebase Listener and QR Code Generation
    setup_realtime_listener()
    generate_qr()

    # Button to Return to Main Window
    tk.Button(
        new_window, 
        text="Back to Main Menu", 
        command=lambda: [new_window.destroy(), window.deiconify()], 
        font=("Roboto", 14, "bold"), 
        fg="#FFFFFF", 
        bg="#FF7D2D", 
        activebackground="#FF7D2D", 
        activeforeground="#FFFFFF", 
        width=20, 
        height=2
    ).pack(pady=20)

# Main Tkinter window
window = tk.Tk()
window.title("Delivery Robot")

# Set main window to fullscreen
window.attributes('-fullscreen', True)

window.configure(bg="#292929")

# Main Title (centered)
tk.Label(
    window, 
    text="Delivery Robot", 
    font=("Roboto", 40, "bold"), 
    fg="#FF7D2D", 
    bg="#292929"
).place(relx=0.5, rely=0.3, anchor="center")

# QR Generator Button (centered)
tk.Button(
    window, 
    text="QR Generator", 
    command=open_new_window, 
    font=("Roboto", 16, "bold"), 
    fg="#FFFFFF", 
    bg="#FF7D2D", 
    activebackground="#292929", 
    activeforeground="#FFFFFF", 
    width=20, 
    height=3
).place(relx=0.5, rely=0.45, anchor="center")

# Footer Label
tk.Label(
    window, 
    text="© 2024 Delivery Robot", 
    font=("Roboto", 12), 
    fg="#FFFFFF", 
    bg="#292929"
).pack(side="bottom", pady=10)

# Graceful shutdown for GPIO
def on_close():
    GPIO.cleanup()  # Release GPIO resources
    window.destroy()

window.protocol("WM_DELETE_WINDOW", on_close)

window.mainloop()