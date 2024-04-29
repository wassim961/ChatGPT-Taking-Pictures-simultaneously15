# ChatGPT-Taking-Pictures-simultaneously15
# Created by Wassim Ch using free Chat GPT 3.5.
# Anyone can contribute to improve this program.
# This Python program is for taking Pictures from multiple cameras approximately 120 cams at the same time to create a 3D plastic sample of people, A small plastic Colored Figurine.

import cv2
import os
import tkinter as tk
from tkinter import filedialog, messagebox, simpledialog
import winsound
from PIL import Image, ImageTk

def show_about():
    messagebox.showinfo("About", "A program to take pictures from all Connected USB Cameras. This program was built with the help of ChatGPT. Rev 1 (2023)")

def exit_app():
    window.destroy()

def cameras_resolution():
    label.config(text="This is Page 1")
    home_button.grid(row=1, column=0)
    cameras_resolution()
    #display_cameras_resolution()

def show_live_cams():
    label.config(text="This is Page 2")
    home_button.grid(row=1, column=0)
    show_live_cams()

def page3():
    label.config(text="This is Page 3")
    home_button.grid(row=1, column=0)

def go_home():
    label.config(text="Hello, World!")
    home_button.grid_forget()

window = tk.Tk()
window.title("Cameras")
window.geometry("500x375+400+150")
window.configure(bg='white')

menu_bar = tk.Menu(window)

file_menu = tk.Menu(menu_bar, tearoff=0)
file_menu.add_command(label="Exit", command=exit_app)
menu_bar.add_cascade(label="File", menu=file_menu)

pages_menu = tk.Menu(menu_bar, tearoff=0)
pages_menu.add_command(label="Cameras Resolution", command=cameras_resolution)
pages_menu.add_command(label="Show Live Cams", command=show_live_cams)
pages_menu.add_command(label="Page 3", command=page3)
menu_bar.add_cascade(label="Pages", menu=pages_menu)

help_menu = tk.Menu(menu_bar, tearoff=0)
help_menu.add_command(label="About", command=show_about)
menu_bar.add_cascade(label="Help", menu=help_menu)

window.config(menu=menu_bar)

label = tk.Label(window, text="Hello, World!")
label.grid(row=0, column=0, pady=0)

home_button = tk.Button(window, text="Home", command=go_home)

window.grid_rowconfigure(0, weight=0)
window.grid_columnconfigure(0, weight=1)

# Declare global variables
cameras = []
camera_resolutions = []
save_directory = ""
camera_info_labels = []
camera_count_label = None

def main():
    global camera_count_label

    # Initialize cameras
    num_cameras = 0
    while True:
        camera = cv2.VideoCapture(num_cameras, cv2.CAP_DSHOW)
        if not camera.read()[0]:
            break

        # Set the width and height of the frame
        camera.set(cv2.CAP_PROP_FRAME_WIDTH, 3264)
        camera.set(cv2.CAP_PROP_FRAME_HEIGHT, 2448)

        cameras.append(camera)

        # Get camera resolution
        width = int(camera.get(cv2.CAP_PROP_FRAME_WIDTH))
        height = int(camera.get(cv2.CAP_PROP_FRAME_HEIGHT))
        resolution = f"{width}x{height}"
        camera_resolutions.append(resolution)

        num_cameras += 1

    # Connected cameras label
    camera_count_label = tk.Label(window, text=f"Connected Cameras: {len(cameras)}", bg="white")
    camera_count_label.grid(row=0, column=0, pady=10)

    # Frame to hold buttons and labels
    button_frame = tk.Frame(window, bg="white")
    button_frame.grid(row=1, column=0, padx=10)

    # Select directory button
    select_directory_btn = tk.Button(button_frame, text="Select Directory", height=7, width=25, command=select_directory, bg="#cdd1f3")
    select_directory_btn.grid(row=0, column=0, pady=10)


    # Take pictures button
    take_pictures_btn = tk.Button(button_frame, text="Take Pictures", height=7, width=25, command=take_pictures, bg="#cdd1f3")
    take_pictures_btn.grid(row=0, column=1, pady=10)

    # Show live cams button
    show_live_cams_btn = tk.Button(button_frame, text="Show Live Cams", height=7, width=25, command=show_live_cams, bg="#cdd1f3")
    show_live_cams_btn.grid(row=2, column=0, pady=10)

    # Cameras resolution button
    cameras_resolution_btn = tk.Button(button_frame, text="Cameras Resolution", height=7, width=25, command=cameras_resolution, bg="#cdd1f3")
    cameras_resolution_btn.grid(row=2, column=1, pady=10)

    # Start the Tkinter event loop
    window.mainloop()


def select_directory():
    global save_directory
    save_directory = filedialog.askdirectory()

def take_pictures():
    global cameras, camera_resolutions, camera_count_label, save_directory

    # Check if a directory is selected, otherwise create a default directory
    if not save_directory:
        save_directory = os.path.join(os.path.expanduser("~"), "Desktop", "3D Picture Work")
        os.makedirs(save_directory, exist_ok=True)

    # Play 1 beep sound when software starts taking pictures
    for _ in range(1):
        winsound.Beep(1000, 1000)

    for i, camera in enumerate(cameras):
        # Reopen the camera
        camera.open(i, cv2.CAP_DSHOW)

        # Set the width and height of the frame
        camera.set(cv2.CAP_PROP_FRAME_WIDTH, 3264)
        camera.set(cv2.CAP_PROP_FRAME_HEIGHT, 2448)

        # Capture frames from cameras
        frames = []
        for camera in cameras:
            ret, frame = camera.read()
            if not ret:
                print("Failed to capture frame from one of the cameras.")
                break
            frames.append(frame)

        # Check if all frames were successfully captured
        if len(frames) == len(cameras):
            # Save the images
            for i, frame in enumerate(frames):
                filename = f"camera{i + 1}.jpg"
                filepath = os.path.join(save_directory, filename)
                cv2.imwrite(filepath, frame)
                print(f"Picture {i + 1} saved as {filename}")

            # Release the camera
            # TODO comment out release cameras to fix cameras not showing after taking pics
            #camera.release()

    # Play shutter release sound
    # Make sure to replace "shutter.wav" with the actual path to your shutter release sound file.
    winsound.PlaySound("C:\\Users\\Desktop-Core-i5\\Desktop\\take still pictures of all USB cameras connected to the compute\\Shutter.wav", winsound.SND_FILENAME)

    # Update the label with the new camera count
    camera_count_label.configure(text=f"Connected Cameras: {len(cameras)}")

def show_live_cams():
    def update_frames():
        for i, camera in enumerate(cameras):
            ret, frame = camera.read()
            if ret:
                frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
                img = Image.fromarray(frame)
                img = img.resize((204, 153))
                img_tk = ImageTk.PhotoImage(image=img)
                label = camera_labels[i]
                label.configure(image=img_tk)
                label.image = img_tk

        # TODO change the number to change refresh rate
        # (slower fps, better ram usage)
        window.after(10, update_frames)

    def save_images():
        for i, camera in enumerate(cameras):
            ret, frame = camera.read()
            if ret:
                frame = cv2.cvtColor(frame, cv2.COLOR_BGR2RGB)
                img = Image.fromarray(frame)
                img.save(os.path.join(save_directory, f"camera_{i+1}.jpg"))  # Save image to the specified directory

    # Create Tkinter window
    window = tk.Toplevel()
    window.title("USB Cameras Viewer")
    window.geometry("500x375+400+150")
    window.configure(bg="white")  # Set background color

    # Create a frame with a scrollbar
    canvas = tk.Canvas(window, bg="white")
    canvas.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)
    scrollbar = tk.Scrollbar(window, command=canvas.yview)
    scrollbar.pack(side=tk.RIGHT, fill=tk.Y)
    canvas.configure(yscrollcommand=scrollbar.set)

    # Create a frame inside the canvas
    frame = tk.Frame(canvas, bg="white")
    canvas.create_window((0, 0), window=frame, anchor=tk.NW)

    # Calculate grid dimensions
    num_cols = min(len(cameras), 3)
    num_rows = (len(cameras) + num_cols - 1) // num_cols

    # Add camera labels to the frame in a grid layout
    camera_labels = []
    for i, camera in enumerate(cameras):
        row = i // num_cols
        col = i % num_cols
        label = tk.Label(frame, bg="white")
        label.grid(row=row, column=col, pady=10)
        camera_labels.append(label)

    # Configure the scrollbar to work with the canvas
    frame.bind("<Configure>", lambda e: canvas.configure(scrollregion=canvas.bbox("all")))

    # Create a button to take pictures for all cameras
    button_frame = tk.Frame(window, bg="white")
    button_frame.pack(pady=10)
    take_picture_button = tk.Button(button_frame, text="Take Pictures", height=2, width=10, command=save_images)
    take_picture_button.pack()

    # Set the save directory on the desktop
    save_directory = os.path.join(os.path.expanduser("~"), "Desktop", "camera_images")
    os.makedirs(save_directory, exist_ok=True)

    # Start capturing frames and updating the GUI
    update_frames()


def cameras_resolution():
    def close_resolution_window():
        resolution_window.destroy()

    # Create a new window for camera dimensions
    resolution_window = tk.Toplevel()
    resolution_window.title("Cameras Resolution")
    resolution_window.geometry("500x375+400+150")
    resolution_window.configure(bg="white")  # Set background color



    # Camera info labels
    for i in range(len(cameras)):
        camera_info_label = tk.Label(
            resolution_window,
            text=f"Camera {i + 1} - Resolution: {camera_resolutions[i]}",
            font=("Arial", 9),
            bg="white",
            fg="black"
        )
        camera_info_label.pack(pady=5)
        camera_info_labels.append(camera_info_label)

    # Close button
    close_button = tk.Button(resolution_window, text="Close", command=close_resolution_window)
    close_button.pack(pady=10)

if __name__ == "__main__":
    main()

