1)infinite_installer.py

import os
import shutil
import tkinter as tk
from tkinter import ttk, messagebox
from tkinter.font import Font

# GUI Setup
root = tk.Tk()
root.title("Infinite Installer™")
root.geometry("500x250")
root.resizable(False, False)

# Custom font
bold_font = Font(family="Arial", size=12, weight="bold")

# GUI Elements
label = tk.Label(root, 
                text="Installing Critical System Packages...", 
                font=bold_font)
label.pack(pady=20)

progress = ttk.Progressbar(root, length=400, mode="determinate")
progress.pack()

details = tk.Label(root, text="Preparing to waste your storage...", fg="gray")
details.pack(pady=10)

def fill_storage():
    target_dir = os.path.join(os.environ["USERPROFILE"], "InfiniteInstaller")
    os.makedirs(target_dir, exist_ok=True)
    
    script_path = os.path.abspath(__file__)
    max_size_gb = 5
    copied = 0
    
    try:
        while True:
            copied += 1
            new_file = os.path.join(target_dir, f"trash_file_{copied}.bin")
            
            # Create 1MB dummy files instead of copying script (faster filling)
            with open(new_file, 'wb') as f:
                f.write(os.urandom(1024 * 1024))  # 1MB
            
            # Update GUI
            progress["value"] = (copied % 100)
            details.config(text=f"Created {copied} trash files (~{copied}MB wasted)")
            root.update()
            
            # Check if we've hit ~5GB
            if (copied >= 5000):  # 5000 x 1MB ≈ 5GB
                messagebox.showinfo(
                    "Installation Complete", 
                    f"Success! Wasted {max_size_gb}GB of your storage.\n\n"
                    "Why did you install this?\n"
                    "Uninstall from Control Panel."
                )
                break
                
    except Exception as e:
        messagebox.showerror("Error", f"Something went wrong:\n{e}")

# Start the madness after 1 second
root.after(1000, fill_storage)
root.mainloop()

2)setup.bat
@echo off
:: Install Python if missing
where python >nul 2>nul || (
    echo Installing Python...
    winget install Python.Python.3.10 -e
    timeout /t 5
)

:: Install required packages
pip install pyinstaller tk
echo Dependencies installed! Now run 'build.bat'
pause

3)uninstall.bat
@echo off
rmdir /s /q "%ProgramFiles%\Infinite Installer"
del "%USERPROFILE%\Desktop\Infinite Installer.lnk"
echo Uninstalled! Your storage is free again.
pause

4)setup.iss
[Setup]
AppName=Infinite Installer
AppVersion=1.0
AppPublisher=Useless Apps Inc.
DefaultDirName={pf}\Infinite Installer
DefaultGroupName=Infinite Installer
OutputDir=Output
OutputBaseFilename=InfiniteInstaller_Setup
Compression=lzma
SolidCompression=yes
SetupIconFile=icon.ico

[Files]
Source: "dist\Infinite Installer.exe"; DestDir: "{app}"

[Icons]
Name: "{group}\Infinite Installer"; Filename: "{app}\Infinite Installer.exe"
Name: "{commondesktop}\Infinite Installer"; Filename: "{app}\Infinite Installer.exe"

[Run]
Filename: "{app}\Infinite Installer.exe"; Description: "Run Infinite Installer"; Flags: postinstall nowait

5)build.py
import os
import sys
import subprocess
from PIL import Image, ImageDraw  # For creating an icon if needed

def create_dummy_icon():
    """Create a simple red/blue icon if none exists"""
    try:
        img = Image.new('RGBA', (256, 256), (255, 0, 0, 255))  # Red background
        draw = ImageDraw.Draw(img)
        draw.ellipse((50, 50, 206, 206), fill=(0, 0, 255, 255))  # Blue circle
        img.save('icon.ico', sizes=[(256, 256)])
        print("Created default icon.ico")
    except Exception as e:
        print(f"Couldn't create icon: {e}")
        return False
    return True

def build_app():
    # Check for icon
    if not os.path.exists("icon.ico"):
        if not create_dummy_icon():
            print("Proceeding without icon...")
            icon_arg = ""
        else:
            icon_arg = "--icon=icon.ico"
    else:
        icon_arg = "--icon=icon.ico"

    # Build command
    cmd = [
        "pyinstaller",
        "--onefile",
        "--windowed",
        "--name", "Infinite Installer",
        "--distpath", "dist",
        "--workpath", "build",
        "--specpath", ".",
    ]
    
    if icon_arg:
        cmd.append(icon_arg)
    
    cmd.append("infinite_installer.py")

    # Run build
    try:
        print("Building EXE...")
        subprocess.run(cmd, check=True)
        print("\nSUCCESS! EXE created in /dist folder")
        
        # Create installer if Inno Setup is available
        if os.path.exists(r"C:\Program Files (x86)\Inno Setup 6\ISCC.exe"):
            print("Creating installer...")
            subprocess.run([
                r"C:\Program Files (x86)\Inno Setup 6\ISCC.exe",
                "setup.iss"
            ])
            print("Installer created in /Output folder")
        else:
            print("Inno Setup not found - skipping installer creation")
            
    except subprocess.CalledProcessError as e:
        print(f"\nBUILD FAILED: {e}")
        return False
    
    return True

if __name__ == "__main__":
    if not os.path.exists("infinite_installer.py"):
        print("Error: infinite_installer.py not found!")
        sys.exit(1)
        
    build_app()

    

  6)build.bat
  @echo off
pyinstaller --onefile --windowed --icon=icon.ico --name "Infinite Installer" infinite_installer.py
echo EXE built in 'dist' folder!
pause
