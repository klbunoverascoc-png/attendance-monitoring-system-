from datetime import datetime

students = {}
attendance_records = []


def valid_id(student_id):
    parts = student_id.split("-")
    return (len(parts) == 3 and len(parts[0]) == 2 and
            len(parts[1]) == 4 and len(parts[2]) == 6 and
            all(part.isdigit() for part in parts))


def get_student_id():
    while True:
        student_id = input("Enter Student ID (XX-XXXX-XXXXXX): ").strip()

        if valid_id(student_id):
            return student_id

        print("Invalid ID. Use the format XX-XXXX-XXXXXX (12 digits).")


def valid_teacher_id(teacher_id):
    parts = teacher_id.split("-")
    return (len(parts) == 3 and len(parts[0]) == 2 and
            len(parts[1]) == 3 and len(parts[2]) == 1 and
            parts[0].isdigit() and parts[1].isdigit() and
            parts[2].isalpha())


def get_teacher_id():
    while True:
        teacher_id = input("Enter Teacher ID (XX-XXX-A): ").strip()

        if valid_teacher_id(teacher_id):
            return teacher_id

        print("Invalid Teacher ID. Use the format XX-XXX-A (5 digits and 1 letter).")


def get_student_name():
    while True:
        name = input("Enter Student Name: ").strip()

        if name == "":
            print("Name cannot be empty.")
            continue

        valid = True

        for character in name:
            if not character.isalpha() and character != " ":
                valid = False
                break

        if valid:
            return name

        print("Invalid name. Use letters and spaces only.")


def get_existing_student():
    while True:
        student_id = get_student_id()

        if student_id in students:
            return student_id

        print("Student not found. Enter an existing Student ID.")


def get_section():
    while True:
        section = input("Enter Section: ").strip().upper()
        if section != "":
            return section
        print("Section cannot be empty. Try again.")


def get_student_by_name():
    while True:
        name = input("Enter Student Name: ").strip()
        matches = []

        for student_id in students:
            if students[student_id]["name"].lower() == name.lower():
                matches.append(student_id)

        if len(matches) == 1:
            return matches[0]
        elif len(matches) > 1:
            print("Multiple students have this name:")
            for number in range(len(matches)):
                student_id = matches[number]
                print(number + 1, "-", students[student_id]["name"],
                      "(ID:", student_id + ", Section:", students[student_id]["section"] + ")")
            while True:
                choice = input("Choose student number: ").strip()
                if choice.isdigit() and 1 <= int(choice) <= len(matches):
                    return matches[int(choice) - 1]
                print("Invalid choice. Try again.")

        print("Student not found. Enter an existing Student Name.")


def get_date():
    while True:
        date = input("Enter Date (YYYY-MM-DD): ").strip()
        parts = date.split("-")

        if len(parts) == 3:
            if len(parts[0]) == 4 and len(parts[1]) == 2 and len(parts[2]) == 2:
                if parts[0].isdigit() and parts[1].isdigit() and parts[2].isdigit():
                    year = int(parts[0])
                    month = int(parts[1])
                    day = int(parts[2])

                    if month >= 1 and month <= 12:
                        if month == 2:
                            max_day = 28

                            if year % 400 == 0 or (year % 4 == 0 and year % 100 != 0):
                                max_day = 29
                        elif month == 4 or month == 6 or month == 9 or month == 11:
                            max_day = 30
                        else:
                            max_day = 31

                        if day >= 1 and day <= max_day:
                            return date

        print("Invalid date. Enter a valid date in YYYY-MM-DD format.")


def get_choice(prompt, choices):
    while True:
        choice = input(prompt).strip()

        if choice in choices:
            return choice

        print("Invalid choice. Please try again.")


def add_student():
    print("\n===== ADD STUDENT PROFILE =====")

    while True:
        student_id = get_student_id()

        if student_id not in students:
            break

        print("Student ID already exists. Enter another ID.")

    name = get_student_name()
    section = get_section()

    students[student_id] = {
        "name": name,
        "section": section,
        "status": "Active"
    }

    print("Student profile added successfully!")


def record_attendance():
    print("\n===== RECORD ATTENDANCE =====")

    if not students:
        print("No student profiles available. Add a student first.")
        return

    student_id = get_student_by_name()

    if students[student_id]["status"] == "Dropped":
        print("Cannot record attendance for a dropped student.")
        return

    while True:
        date = get_date()
        duplicate = False

        for record in attendance_records:
            if record["student_id"] == student_id and record["date"] == date:
                duplicate = True
                break

        if not duplicate:
            break

        print("Attendance already recorded for this student on that date.")
        print("Please enter another date.")

    print("\n1. Present")
    print("2. Absent")

    choice = get_choice("Choose attendance status: ", ("1", "2"))

    if choice == "1":
        status = "Present"
    else:
        status = "Absent"

    attendance_records.append({
        "student_id": student_id,
        "date": date,
        "name": students[student_id]["name"],
        "status": status
    })

    print("Attendance recorded successfully!")


def update_student_status():
    print("\n===== UPDATE STUDENT STATUS =====")

    if not students:
        print("No student profiles available. Add a student first.")
        return

    student_id = get_student_by_name()

    print("\n1. Active")
    print("2. Inactive")
    print("3. Dropped")

    choice = get_choice("Choose student status: ", ("1", "2", "3"))

    if choice == "1":
        status = "Active"
    elif choice == "2":
        status = "Inactive"
    else:
        status = "Dropped"

    students[student_id]["status"] = status

    print("Student status updated to", status)


def view_attendance_records():
    print("\n===== VIEW ATTENDANCE RECORDS =====")

    if not attendance_records:
        print("No attendance records available.")
        return

    section = get_section()
    matching_records = []

    for record in attendance_records:
        if students[record["student_id"]]["section"] == section:
            matching_records.append(record)

    if not matching_records:
        print("No attendance records found for Section", section)
        return

    print("\nSection:", section)
    print(f"{'Student ID':<18}{'Date':<14}{'Student Name':<25}{'Status'}")
    print("-" * 70)

    for record in matching_records:
        print(
            f"{record['student_id']:<18}"
            f"{record['date']:<14}"
            f"{record['name']:<25}"
            f"{record['status']}"
        )


def get_counts(student_id):
    present = 0
    absent = 0

    for record in attendance_records:
        if record["student_id"] == student_id:
            if record["status"] == "Present":
                present += 1
            elif record["status"] == "Absent":
                absent += 1

    return present, absent


def view_class_summary():
    print("\n===== CLASS SUMMARY =====")

    if not students:
        print("No student profiles available.")
        return

    section = get_section()
    matching_students = []

    for student_id in sorted(students):
        if students[student_id]["section"] == section:
            matching_students.append(student_id)

    if not matching_students:
        print("No students found for Section", section)
        return

    print("\nSection:", section)
    print(
        f"{'Student ID':<18}"
        f"{'Student Name':<24}"
        f"{'Status':<12}"
        f"{'Present':<9}"
        f"{'Absent'}"
    )
    print("-" * 80)

    for student_id in matching_students:
        name = students[student_id]["name"]
        status = students[student_id]["status"]

        present, absent = get_counts(student_id)

        print(
            f"{student_id:<18}"
            f"{name:<24}"
            f"{status:<12}"
            f"{present:<9}"
            f"{absent}"
        )


def student_menu():
    print("\n===== STUDENT ATTENDANCE =====")

    if not students:
        print("No student profiles available.")
        return

    student_id = get_existing_student()

    print("\nStudent Name:", students[student_id]["name"])
    print("Student ID:", student_id)
    print("Enrollment Status:", students[student_id]["status"])

    print("\n===== MY ATTENDANCE RECORDS =====")

    found = False

    print(f"{'Date':<16}{'Status'}")
    print("-" * 30)

    for record in attendance_records:
        if record["student_id"] == student_id:
            print(f"{record['date']:<16}{record['status']}")
            found = True

    if not found:
        print("No attendance records available.")

    present, absent = get_counts(student_id)

    print("\n===== MY ATTENDANCE SUMMARY =====")
    print("Total Records:", present + absent)
    print("Present:", present)
    print("Absent:", absent)


def teacher_menu(teacher_id):
    while True:
        now = datetime.now()

        print("\n===== TEACHER MENU =====")
        print("Date:", now.strftime("%B %d, %Y"))
        print("Teacher ID:", teacher_id)
        print("1. Add Student Profile")
        print("2. Record Attendance")
        print("3. Update Student Status")
        print("4. View Attendance Records")
        print("5. View Class Summary")
        print("6. Back to Main Menu")

        choice = get_choice(
            "Enter your choice: ",
            ("1", "2", "3", "4", "5", "6")
        )

        if choice == "1":
            add_student()
        elif choice == "2":
            record_attendance()
        elif choice == "3":
            update_student_status()
        elif choice == "4":
            view_attendance_records()
        elif choice == "5":
            view_class_summary()
        elif choice == "6":
            break


def main():
    while True:
        now = datetime.now()

        print("\n===== CLASSCHECK =====")
        print("Digital Attendance Monitoring System")
        print("Date:", now.strftime("%B %d, %Y"))
        print("1. Teacher")
        print("2. Student")
        print("3. Exit")

        choice = get_choice("Enter your choice: ", ("1", "2", "3"))

        if choice == "1":
            print("\n===== TEACHER IDENTIFICATION =====")

            teacher_id = get_teacher_id()

            teacher_menu(teacher_id)

        elif choice == "2":
            student_menu()

        elif choice == "3":
            print("Thank you for using ClassCheck!")
            break


main()