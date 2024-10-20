# medication-manager
Collecting data about the medicine a patient take, the quantity of medicines left, remind him/her to take the medicine on time and reminds him/her to order more if quantity low
import tkinter as tk
from tkinter import messagebox
import sqlite3
from plyer import notification
import time
import threading


# Database setup
def setup_database():
    conn = sqlite3.connect('medications.db')
    c = conn.cursor()
    c.execute('''
        CREATE TABLE IF NOT EXISTS medications (
            id INTEGER PRIMARY KEY,
            name TEXT NOT NULL,
            dosage TEXT NOT NULL,
            quantity INTEGER NOT NULL,
            reminder_time TEXT NOT NULL
        )
    ''')
    conn.commit()
    conn.close()


# Function to add medication
def add_medication(name, dosage, quantity, reminder_time):
    conn = sqlite3.connect('medications.db')
    c = conn.cursor()
    c.execute("INSERT INTO medications (name, dosage, quantity, reminder_time) VALUES (?, ?, ?, ?)",
              (name, dosage, quantity, reminder_time))
    conn.commit()
    conn.close()
    check_low_quantities()
    messagebox.showinfo("Success", "Medication added successfully!")


# Function to check for low quantities and notify
def check_low_quantities():
    conn = sqlite3.connect('medications.db')
    c = conn.cursor()
    c.execute("SELECT name, quantity FROM medications")
    medications = c.fetchall()
    for med in medications:
        if med[1] < 5:  # Threshold for low quantity
            notify_user(med[0])
    conn.close()


# Function to send notification
def notify_user(medicine_name):
    notification.notify(
        title='Medication Reminder',
        message=f'Time to reorder: {medicine_name}',
        app_name='Medication Manager',
        timeout=10,
    )


# Function to run periodic checks
def run_periodic_checks():
    while True:
        time.sleep(3600)  # Check every hour
        check_low_quantities()


# Function to handle the button click
def on_add_button_click():
    name = entry_name.get()
    dosage = entry_dosage.get()
    quantity = entry_quantity.get()
    reminder_time = entry_reminder_time.get()

    if name and dosage and quantity.isdigit() and reminder_time:
        add_medication(name, dosage, int(quantity), reminder_time)
        entry_name.delete(0, tk.END)
        entry_dosage.delete(0, tk.END)
        entry_quantity.delete(0, tk.END)
        entry_reminder_time.delete(0, tk.END)
    else:
        messagebox.showerror("Input Error", "Please fill all fields correctly!")


# Initialize the main window
app = tk.Tk()
app.title('Medication Manager')

# Create and place the UI elements
tk.Label(app, text='Medication Name:').grid(row=0, column=0)
entry_name = tk.Entry(app)
entry_name.grid(row=0, column=1)

tk.Label(app, text='Dosage:').grid(row=1, column=0)
entry_dosage = tk.Entry(app)
entry_dosage.grid(row=1, column=1)

tk.Label(app, text='Quantity Left:').grid(row=2, column=0)
entry_quantity = tk.Entry(app)
entry_quantity.grid(row=2, column=1)

tk.Label(app, text='Reminder Time (HH:MM):').grid(row=3, column=0)
entry_reminder_time = tk.Entry(app)
entry_reminder_time.grid(row=3, column=1)

tk.Button(app, text='Add Medication', command=on_add_button_click).grid(row=4, columnspan=2)

# Setup database
setup_database()

# Start the periodic check in a separate thread
threading.Thread(target=run_periodic_checks, daemon=True).start()

# Start the main loop
app.mainloop()

