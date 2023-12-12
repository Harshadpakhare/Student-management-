# Student-management-
#include <iostream>
#include <fstream>
#include <string>
#include <vector>
#include <algorithm>
#include <limits>

using namespace std;

const int MAX_STUDENTS = 100;

class Student {
public:
    string name;
    int rollNumber;
    double cgpa;
    string phnNo;
    string studentClass;
    string division;
};

vector<Student> students;

bool isRollNumberUnique(int rollNumber) {
    return find_if(students.begin(), students.end(),
                   [rollNumber](const Student& student) {
                       return student.rollNumber == rollNumber;
                   }) == students.end();
}

bool isValidClassName(const string& className) {
    return all_of(className.begin(), className.end(), ::isalpha);
}

bool isValidDivision(const string& division) {
    return all_of(division.begin(), division.end(), ::isalpha);
}

void addStudent() {
    if (students.size() < MAX_STUDENTS) {
        Student newStudent;
        cout << "Enter student name, roll number, CGPA, phone number, class, and division: ";
        cin.ignore();
        getline(cin, newStudent.name);

        while (!all_of(newStudent.name.begin(), newStudent.name.end(), ::isalpha)) {
            cout << "Invalid name. Name must contain only alphabetical characters.\n";
            cout << "Enter student name, roll number, CGPA, phone number, class, and division: ";
            getline(cin, newStudent.name);
        }

        while (!(cin >> newStudent.rollNumber) ||
               newStudent.rollNumber < 1 || newStudent.rollNumber > 100 ||
               !isRollNumberUnique(newStudent.rollNumber) ||
               !(cin >> newStudent.cgpa) ||
               newStudent.cgpa < 0 || newStudent.cgpa > 10) {
            cin.clear();
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
            cout << "Invalid input. Please enter a valid roll number (between 1 and 100) or unique roll number, and CGPA (between 0 and 10).\n";
            cout << "Enter student name, roll number, CGPA, phone number, class, and division: ";
            getline(cin, newStudent.name);
        }

        cout << "Enter phone number (10 digits): ";
        cin >> newStudent.phnNo;

        while (any_of(students.begin(), students.end(),
                      [&newStudent](const Student& s) {
                          return s.phnNo == newStudent.phnNo;
                      }) || newStudent.phnNo.length() != 10) {
            if (newStudent.phnNo.length() != 10) {
                cout << "Phone number must have exactly 10 digits. ";
            } else {
                cout << "Phone number already exists. ";
            }
            cout << "Enter a unique phone number (10 digits): ";
            cin >> newStudent.phnNo;
        }

        cout << "Enter class: ";
        cin >> newStudent.studentClass;

        while (!isValidClassName(newStudent.studentClass)) {
            cout << "Invalid class name. Class must contain only alphabetical characters.\n";
            cout << "Enter class: ";
            cin >> newStudent.studentClass;
        }

        cout << "Enter division: ";
        cin >> newStudent.division;

        while (!isValidDivision(newStudent.division)) {
            cout << "Invalid division. Division must contain only alphabetical characters.\n";
            cout << "Enter division: ";
            cin >> newStudent.division;
        }

        students.push_back(newStudent);

        ofstream outfile("data 2.txt", ios::app);
        if (outfile.is_open()) {
            outfile << newStudent.name << " " << newStudent.rollNumber << " " << newStudent.cgpa << " "
                    << newStudent.phnNo << " " << newStudent.studentClass << " " << newStudent.division << endl;
            cout << "Student added successfully.\n";
        } else {
            cout << "Error: Unable to open the file for writing.\n";
        }
    } else {
        cout << "Student database is full. Cannot add more students.\n";
    }
}

void displayStudentInfo(int rollNumber) {
    auto it = find_if(students.begin(), students.end(),
                      [rollNumber](const Student& student) {
                          return student.rollNumber == rollNumber;
                      });
    if (it != students.end()) {
        cout << "Name: " << it->name << "\n";
        cout << "Roll Number: " << it->rollNumber << "\n";
        cout << "CGPA: " << it->cgpa << "\n";
        cout << "Phone Number: " << it->phnNo << "\n";
        cout << "Class: " << it->studentClass << "\n";
        cout << "Division: " << it->division << "\n";
    } else {
        cout << "Student not found.\n";
    }
}

void removeStudent(int rollNumber) {
    auto it = remove_if(students.begin(), students.end(),
                        [rollNumber](const Student& student) {
                            return student.rollNumber == rollNumber;
                        });
    if (it != students.end()) {
        students.erase(it, students.end());
        cout << "Student removed successfully.\n";

        // Update the file after removing the student
        ofstream outfile("data 2.txt");
        if (outfile.is_open()) {
            for (const auto& student : students) {
                outfile << student.name << " " << student.rollNumber << " " << student.cgpa << " "
                        << student.phnNo << " " << student.studentClass << " " << student.division << endl;
            }
            outfile.close();
        } else {
            cout << "Error: Unable to open the file for writing.\n";
        }
    } else {
        cout << "Student not found. Cannot remove.\n";
    }
}

int main() {
    ifstream infile("data 2.txt");
    if (infile.is_open()) {
        string name;
        int rollNumber;
        double cgpa;
        string phnNo;
        string studentClass;
        string division;

        while (infile >> name >> rollNumber >> cgpa >> phnNo >> studentClass >> division) {
            Student loadedStudent{ name, rollNumber, cgpa, phnNo, studentClass, division };
            students.push_back(loadedStudent);
        }

        infile.close();
    } else {
        cout << "No student data file found. Starting with an empty database.\n";
    }

    while (true) {
        cout << "\nStudent Management System\n";
        cout << "1. Add Student\n";
        cout << "2. Display Student Info\n";
        cout << "3. Remove Student\n";
        cout << "4. Quit\n";

        int choice;
        cout << "Enter your choice: ";

        while (!(cin >> choice)) {
            cout << "Invalid input. Please enter a valid number.\n";
            cin.clear();
            cin.ignore(numeric_limits<streamsize>::max(), '\n');
        }

        switch (choice) {
            case 1:
                addStudent();
                break;
            case 2:
                int rollNumber;
                cout << "Enter roll number to display student info: ";
                cin >> rollNumber;
                displayStudentInfo(rollNumber);
                break;
            case 3:
                int removeRollNumber;
                cout << "Enter roll number to remove the student: ";
                cin >> removeRollNumber;
                removeStudent(removeRollNumber);
                break;
            case 4:
                cout << "Exiting Student Management System.\n";
                return 0;
            default:
                cout << "Invalid choice. Please try again.\n";
        }
    }

    return 0;
}
