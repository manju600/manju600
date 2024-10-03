import tkinter as tk
from tkinter import messagebox, ttk
import sqlite3

class Hospital:
    def __init__(self):
        self.db_name = 'hospital_management.db'
        self.create_db()

    def create_db(self):
        conn = sqlite3.connect(self.db_name)
        c = conn.cursor()

        # Create tables
        c.execute('''
        CREATE TABLE IF NOT EXISTS doctors (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            specialization TEXT NOT NULL
        )
        ''')

        c.execute('''
        CREATE TABLE IF NOT EXISTS patients (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            name TEXT NOT NULL,
            age INTEGER NOT NULL,
            contact_number TEXT NOT NULL,
            address TEXT NOT NULL,
            discharged BOOLEAN NOT NULL DEFAULT 0
        )
        ''')

        c.execute('''
        CREATE TABLE IF NOT EXISTS appointments (
            id INTEGER PRIMARY KEY AUTOINCREMENT,
            patient_id INTEGER,
            doctor_id INTEGER,
            date TEXT NOT NULL,
            time TEXT NOT NULL,
            status TEXT NOT NULL DEFAULT 'pending',
            FOREIGN KEY (patient_id) REFERENCES patients(id),
            FOREIGN KEY (doctor_id) REFERENCES doctors(id)
        )
        ''')

        conn.commit()
        conn.close()

    def execute_query(self, query, parameters=()):
        conn = sqlite3.connect(self.db_name)
        c = conn.cursor()
        c.execute(query, parameters)
        conn.commit()
        conn.close()

    def fetch_query(self, query, parameters=()):
        conn = sqlite3.connect(self.db_name)
        c = conn.cursor()
        c.execute(query, parameters)
        results = c.fetchall()
        conn.close()
        return results

class HospitalGUI:
    def __init__(self, root):
        self.hospital = Hospital()
        self.root = root
        self.root.title("Hospital Management System")

        # Create a notebook (tabbed interface)
        self.notebook = ttk.Notebook(root)
        self.notebook.pack(fill='both', expand=True)

        # Create frames for each tab
        self.create_add_patient_frame()
        self.create_add_doctor_frame()
        self.create_book_appointment_frame()
        self.create_view_appointments_frame()
        self.create_cancel_appointment_frame()
        self.create_search_doctor_frame()
        self.create_view_patients_frame()
        self.create_view_discharged_patients_frame()
        self.create_discharge_patient_frame()
        self.create_view_patient_by_id_frame()

    def create_add_patient_frame(self):
        frame = ttk.Frame(self.notebook)
        self.notebook.add(frame, text="Add Patient")

        # Patient form
        ttk.Label(frame, text="Name").grid(row=0, column=0)
        self.name_entry = ttk.Entry(frame)
        self.name_entry.grid(row=0, column=1)

        ttk.Label(frame, text="Age").grid(row=1, column=0)
        self.age_entry = ttk.Entry(frame)
        self.age_entry.grid(row=1, column=1)

        ttk.Label(frame, text="Contact Number").grid(row=2, column=0)
        self.contact_number_entry = ttk.Entry(frame)
        self.contact_number_entry.grid(row=2, column=1)

        ttk.Label(frame, text="Address").grid(row=3, column=0)
        self.address_entry = ttk.Entry(frame)
        self.address_entry.grid(row=3, column=1)

        ttk.Button(frame, text="Add Patient", command=self.add_patient).grid(row=4, column=0, columnspan=2)

    def create_add_doctor_frame(self):
        frame = ttk.Frame(self.notebook)
        self.notebook.add(frame, text="Add Doctor")

        # Doctor form
        ttk.Label(frame, text="Name").grid(row=0, column=0)
        self.doctor_name_entry = ttk.Entry(frame)
        self.doctor_name_entry.grid(row=0, column=1)

        ttk.Label(frame, text="Specialization").grid(row=1, column=0)
        self.specialization_entry = ttk.Entry(frame)
        self.specialization_entry.grid(row=1, column=1)

        ttk.Button(frame, text="Add Doctor", command=self.add_doctor).grid(row=2, column=0, columnspan=2)

    def create_book_appointment_frame(self):
        frame = ttk.Frame(self.notebook)
        self.notebook.add(frame, text="Book Appointment")

        # Appointment form
        ttk.Label(frame, text="Patient ID").grid(row=0, column=0)
        self.app_patient_id_entry = ttk.Entry(frame)
        self.app_patient_id_entry.grid(row=0, column=1)

        ttk.Label(frame, text="Doctor ID").grid(row=1, column=0)
        self.app_doctor_id_entry = ttk.Entry(frame)
        self.app_doctor_id_entry.grid(row=1, column=1)

        ttk.Label(frame, text="Date").grid(row=2, column=0)
        self.date_entry = ttk.Entry(frame)
        self.date_entry.grid(row=2, column=1)

        ttk.Label(frame, text="Time").grid(row=3, column=0)
        self.time_entry = ttk.Entry(frame)
        self.time_entry.grid(row=3, column=1)

        ttk.Button(frame, text="Book Appointment", command=self.book_appointment).grid(row=4, column=0, columnspan=2)

    def create_view_appointments_frame(self):
        frame = ttk.Frame(self.notebook)
        self.notebook.add(frame, text="View Appointments")

        # View appointments form
        ttk.Label(frame, text="Patient ID").grid(row=0, column=0)
        self.view_patient_id_entry = ttk.Entry(frame)
        self.view_patient_id_entry.grid(row=0, column=1)

        ttk.Button(frame, text="View Appointments", command=self.view_appointments).grid(row=1, column=0, columnspan=2)

        self.appointments_text = tk.Text(frame, height=10, width=50)
        self.appointments_text.grid(row=2, column=0, columnspan=2)

    def create_cancel_appointment_frame(self):
        frame = ttk.Frame(self.notebook)
        self.notebook.add(frame, text="Cancel Appointment")

        # Cancel appointment form
        ttk.Label(frame, text="Appointment ID").grid(row=0, column=0)
        self.cancel_appointment_id_entry = ttk.Entry(frame)
        self.cancel_appointment_id_entry.grid(row=0, column=1)

        ttk.Button(frame, text="Cancel Appointment", command=self.cancel_appointment).grid(row=1, column=0, columnspan=2)

    def create_search_doctor_frame(self):
        frame = ttk.Frame(self.notebook)
        self.notebook.add(frame, text="Search Doctor")

        # Search doctor form
        ttk.Label(frame, text="Specialization").grid(row=0, column=0)
        self.search_specialization_entry = ttk.Entry(frame)
        self.search_specialization_entry.grid(row=0, column=1)

        ttk.Button(frame, text="Search Doctor", command=self.search_doctor).grid(row=1, column=0, columnspan=2)

        self.doctors_text = tk.Text(frame, height=10, width=50)
        self.doctors_text.grid(row=2, column=0, columnspan=2)

    def create_view_patients_frame(self):
        frame = ttk.Frame(self.notebook)
        self.notebook.add(frame, text="View Patients")

        # View patients form
        ttk.Button(frame, text="View All Patients", command=self.view_patients).grid(row=0, column=0, columnspan=2)

        self.patients_text = tk.Text(frame, height=10, width=50)
        self.patients_text.grid(row=1, column=0, columnspan=2)

    def create_view_discharged_patients_frame(self):
        frame = ttk.Frame(self.notebook)
        self.notebook.add(frame, text="View Discharged Patients")

        # View discharged patients form
        ttk.Button(frame, text="View Discharged Patients", command=self.view_discharged_patients).grid(row=0, column=0, columnspan=2)

        self.discharged_patients_text = tk.Text(frame, height=10, width=50)
        self.discharged_patients_text.grid(row=1, column=0, columnspan=2)

    def create_discharge_patient_frame(self):
        frame = ttk.Frame(self.notebook)
        self.notebook.add(frame, text="Discharge Patient")

        # Discharge patient form
        ttk.Label(frame, text="Patient ID").grid(row=0, column=0)
        self.discharge_patient_id_entry = ttk.Entry(frame)
        self.discharge_patient_id_entry.grid(row=0, column=1)

        ttk.Button(frame, text="Discharge Patient", command=self.discharge_patient).grid(row=1, column=0, columnspan=2)

    def create_view_patient_by_id_frame(self):
        frame = ttk.Frame(self.notebook)
        self.notebook.add(frame, text="View Patient by ID")

        # View patient by ID form
        ttk.Label(frame, text="Patient ID").grid(row=0, column=0)
        self.view_patient_by_id_entry = ttk.Entry(frame)
        self.view_patient_by_id_entry.grid(row=0, column=1)

        ttk.Button(frame, text="View Patient", command=self.view_patient_by_id).grid(row=1, column=0, columnspan=2)

        self.patient_info_text = tk.Text(frame, height=10, width=50)
        self.patient_info_text.grid(row=2, column=0, columnspan=2)

    # Methods for interacting with the database
    def add_patient(self):
        name = self.name_entry.get()
        age = self.age_entry.get()
        contact_number = self.contact_number_entry.get()
        address = self.address_entry.get()

        if not (name and age and contact_number and address):
            messagebox.showerror("Input Error", "All fields must be filled out.")
            return

        query = 'INSERT INTO patients (name, age, contact_number, address) VALUES (?, ?, ?, ?)'
        self.hospital.execute_query(query, (name, age, contact_number, address))
        messagebox.showinfo("Success", "Patient added successfully!")

    def add_doctor(self):
        name = self.doctor_name_entry.get()
        specialization = self.specialization_entry.get()

        if not (name and specialization):
            messagebox.showerror("Input Error", "All fields must be filled out.")
            return

        query = 'INSERT INTO doctors (name, specialization) VALUES (?, ?)'
        self.hospital.execute_query(query, (name, specialization))
        messagebox.showinfo("Success", "Doctor added successfully!")

    def book_appointment(self):
        patient_id = self.app_patient_id_entry.get()
        doctor_id = self.app_doctor_id_entry.get()
        date = self.date_entry.get()
        time = self.time_entry.get()

        if not (patient_id and doctor_id and date and time):
            messagebox.showerror("Input Error", "All fields must be filled out.")
            return

        query = 'INSERT INTO appointments (patient_id, doctor_id, date, time) VALUES (?, ?, ?, ?)'
        self.hospital.execute_query(query, (patient_id, doctor_id, date, time))
        messagebox.showinfo("Success", "Appointment booked successfully!")

    def view_appointments(self):
        patient_id = self.view_patient_id_entry.get()

        if not patient_id:
            messagebox.showerror("Input Error", "Patient ID must be provided.")
            return

        query = '''
        SELECT a.id, d.name, a.date, a.time, a.status 
        FROM appointments a 
        JOIN doctors d ON a.doctor_id = d.id 
        WHERE a.patient_id = ?
        '''
        appointments = self.hospital.fetch_query(query, (patient_id,))
        self.appointments_text.delete(1.0, tk.END)

        for appointment in appointments:
            self.appointments_text.insert(tk.END, f"Appointment ID: {appointment[0]}, Doctor: {appointment[1]}, Date: {appointment[2]}, Time: {appointment[3]}, Status: {appointment[4]}\n")

    def cancel_appointment(self):
        appointment_id = self.cancel_appointment_id_entry.get()

        if not appointment_id:
            messagebox.showerror("Input Error", "Appointment ID must be provided.")
            return

        query = 'UPDATE appointments SET status = "canceled" WHERE id = ?'
        self.hospital.execute_query(query, (appointment_id,))
        messagebox.showinfo("Success", "Appointment canceled successfully!")

    def search_doctor(self):
        specialization = self.search_specialization_entry.get()

        if not specialization:
            messagebox.showerror("Input Error", "Specialization must be provided.")
            return

        query = 'SELECT id, name FROM doctors WHERE specialization = ?'
        doctors = self.hospital.fetch_query(query, (specialization,))
        self.doctors_text.delete(1.0, tk.END)

        for doctor in doctors:
            self.doctors_text.insert(tk.END, f"Doctor ID: {doctor[0]}, Name: {doctor[1]}\n")

    def view_patients(self):
        query = 'SELECT id, name, age, contact_number, address FROM patients'
        patients = self.hospital.fetch_query(query)
        self.patients_text.delete(1.0, tk.END)

        for patient in patients:
            self.patients_text.insert(tk.END, f"Patient ID: {patient[0]}, Name: {patient[1]}, Age: {patient[2]}, Contact Number: {patient[3]}, Address: {patient[4]}\n")

    def view_discharged_patients(self):
        query = 'SELECT id, name, age, contact_number, address FROM patients WHERE discharged = 1'
        discharged_patients = self.hospital.fetch_query(query)
        self.discharged_patients_text.delete(1.0, tk.END)

        for patient in discharged_patients:
            self.discharged_patients_text.insert(tk.END, f"Patient ID: {patient[0]}, Name: {patient[1]}, Age: {patient[2]}, Contact Number: {patient[3]}, Address: {patient[4]}\n")

    def discharge_patient(self):
        patient_id = self.discharge_patient_id_entry.get()

        if not patient_id:
            messagebox.showerror("Input Error", "Patient ID must be provided.")
            return

        query = 'UPDATE patients SET discharged = 1 WHERE id = ?'
        self.hospital.execute_query(query, (patient_id,))
        messagebox.showinfo("Success", "Patient discharged successfully!")

    def view_patient_by_id(self):
        patient_id = self.view_patient_by_id_entry.get()

        if not patient_id:
            messagebox.showerror("Input Error", "Patient ID must be provided.")
            return

        query = 'SELECT * FROM patients WHERE id = ?'
        patient = self.hospital.fetch_query(query, (patient_id,))

        self.patient_info_text.delete(1.0, tk.END)
        if patient:
            patient = patient[0]
            self.patient_info_text.insert(tk.END, f"Patient ID: {patient[0]}\nName: {patient[1]}\nAge: {patient[2]}\nContact Number: {patient[3]}\nAddress: {patient[4]}\nDischarged: {patient[5]}\n")
        else:
            self.patient_info_text.insert(tk.END, "No patient found with the given ID.")

if __name__ == "__main__":
    root = tk.Tk()
    app = HospitalGUI(root)
    root.mainloop()
