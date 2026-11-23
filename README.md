"""
Hospital Appointment Scheduler
Project No.: 80
Language: Python 3
Concepts: Dictionaries, Datetime, Functions, CRUD Operations

Run:
    python hospital_appointment_scheduler.py

Note:
    Appointments are stored in a Python dictionary while the program is running.
    They reset when the program is closed, as required by the basic assignment.
"""

from datetime import datetime, time

DOCTORS = ("Dr. Mehta", "Dr. Rao")
OPENING_TIME = time(9, 0)
LAST_SLOT = time(16, 30)
SLOT_MINUTES = 30

# Appointment ID is the dictionary key.
appointments = {}
next_id_number = 101


def valid_date(date_text):
    """Return a date object only when input exactly matches DD-MM-YYYY."""
    try:
        return datetime.strptime(date_text, "%d-%m-%Y").date()
    except ValueError:
        return None


def valid_time(time_text):
    """Return a time object only when input exactly matches HH:MM."""
    try:
        parsed = datetime.strptime(time_text, "%H:%M").time()
    except ValueError:
        return None

    minutes_from_open = (
        parsed.hour * 60 + parsed.minute
        - (OPENING_TIME.hour * 60 + OPENING_TIME.minute)
    )
    total_slot_minutes = (
        LAST_SLOT.hour * 60 + LAST_SLOT.minute
        - (OPENING_TIME.hour * 60 + OPENING_TIME.minute)
    )

    if (minutes_from_open < 0 or
            parsed > LAST_SLOT or
            minutes_from_open % SLOT_MINUTES != 0 or
            minutes_from_open > total_slot_minutes):
        return None
    return parsed


def slot_is_taken(doctor, appointment_date, appointment_time, exclude_id=None):
    """Check whether this doctor already has an appointment in this slot."""
    for appointment_id, details in appointments.items():
        if appointment_id == exclude_id:
            continue
        if (details["doctor"] == doctor
                and details["date"] == appointment_date
                and details["time"] == appointment_time):
            return True
    return False


def choose_doctor():
    print("\nAvailable doctors:")
    for number, doctor in enumerate(DOCTORS, start=1):
        print(f"{number}. {doctor}")

    choice = input("Choose doctor (1/2): ").strip()
    if choice == "1":
        return DOCTORS[0]
    if choice == "2":
        return DOCTORS[1]
    print("Invalid doctor choice. Please choose 1 or 2.")
    return None


def read_valid_date():
    date_text = input("Enter date (DD-MM-YYYY): ").strip()
    parsed_date = valid_date(date_text)
    if parsed_date is None:
        print("Invalid date. Please enter a real date in DD-MM-YYYY format.")
    return parsed_date


def read_valid_time():
    time_text = input("Enter time (HH:MM, 24-hour format): ").strip()
    parsed_time = valid_time(time_text)
    if parsed_time is None:
        print("Invalid time. Use 30-minute slots from 09:00 to 16:30.")
    return parsed_time


def book_appointment():
    global next_id_number

    print("\n--- BOOK APPOINTMENT ---")
    patient = input("Enter patient name: ").strip()
    if not patient:
        print("Patient name cannot be empty.")
        return

    doctor = choose_doctor()
    if doctor is None:
        return

    appointment_date = read_valid_date()
    if appointment_date is None:
        return

    appointment_time = read_valid_time()
    if appointment_time is None:
        return

    if slot_is_taken(doctor, appointment_date, appointment_time):
        print(
            f"Slot unavailable: {doctor} already has an appointment "
            f"on {appointment_date.strftime('%d-%m-%Y')} at "
            f"{appointment_time.strftime('%H:%M')}."
        )
        return

    appointment_id = f"A{next_id_number}"
    next_id_number += 1

    appointments[appointment_id] = {
        "patient": patient,
        "doctor": doctor,
        "date": appointment_date,
        "time": appointment_time,
    }

    print(
        f"Appointment booked. ID: {appointment_id} | {patient} | {doctor} | "
        f"{appointment_date.strftime('%d-%m-%Y')} "
        f"{appointment_time.strftime('%H:%M')}"
    )


def view_appointments():
    print("\n--- ALL APPOINTMENTS ---")
    if not appointments:
        print("No appointments found.")
        return

    print(f"{'ID':<8} {'PATIENT':<22} {'DOCTOR':<14} {'DATE':<13} {'TIME':<8}")
    print("-" * 70)
    for appointment_id, details in appointments.items():
        print(
            f"{appointment_id:<8} "
            f"{details['patient']:<22} "
            f"{details['doctor']:<14} "
            f"{details['date'].strftime('%d-%m-%Y'):<13} "
            f"{details['time'].strftime('%H:%M'):<8}"
        )
    print(f"Total appointments: {len(appointments)}")


def reschedule_appointment():
    print("\n--- RESCHEDULE APPOINTMENT ---")
    appointment_id = input("Enter appointment ID: ").strip().upper()

    if appointment_id not in appointments:
        print("Appointment ID not found.")
        return

    details = appointments[appointment_id]
    new_date = read_valid_date()
    if new_date is None:
        return

    new_time = read_valid_time()
    if new_time is None:
        return

    if slot_is_taken(details["doctor"], new_date, new_time,
                     exclude_id=appointment_id):
        print(
            f"Slot unavailable: {details['doctor']} already has an "
            f"appointment on {new_date.strftime('%d-%m-%Y')} at "
            f"{new_time.strftime('%H:%M')}."
        )
        return

    details["date"] = new_date
    details["time"] = new_time
    print(
        f"Appointment {appointment_id} rescheduled to "
        f"{new_date.strftime('%d-%m-%Y')} {new_time.strftime('%H:%M')}."
    )


def cancel_appointment():
    print("\n--- CANCEL APPOINTMENT ---")
    appointment_id = input("Enter appointment ID: ").strip().upper()

    if appointment_id not in appointments:
        print("Appointment ID not found.")
        return

    del appointments[appointment_id]
    print(f"Appointment {appointment_id} cancelled.")


def show_menu():
    print("\n========== HOSPITAL APPOINTMENT SCHEDULER ==========")
    print("1. Book Appointment")
    print("2. View All Appointments")
    print("3. Reschedule Appointment")
    print("4. Cancel Appointment")
    print("5. Exit")
    print("====================================================")


def main():
    print("Welcome to Hospital Appointment Scheduler!")
    while True:
        show_menu()
        choice = input("Enter your choice (1-5): ").strip()

        if choice == "1":
            book_appointment()
        elif choice == "2":
            view_appointments()
        elif choice == "3":
            reschedule_appointment()
        elif choice == "4":
            cancel_appointment()
        elif choice == "5":
            print("Thank you for using Hospital Appointment Scheduler. Goodbye!")
            break
        else:
            print("Invalid menu choice. Please enter a number from 1 to 5.")


if __name__ == "__main__":
    main()
