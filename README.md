# Room Booking System (C++17)

This project is an **Advanced Room Allotment & Booking System** written
in **C++17**, using: - Ordered Maps - Event-Sweep Algorithm - Efficient
Booking Search - MinGW-safe Input Handling

------------------------------------------------------------------------

## 📌 Features

-   Add new rooms with capacity & description\
-   Remove rooms (only if no bookings exist)\
-   Book a specific room\
-   Book the best-fit room automatically\
-   Cancel bookings\
-   List all rooms\
-   List all bookings\
-   List bookings for a specific room\
-   Check room status at any time\
-   Search available rooms for a given time + chair requirement

------------------------------------------------------------------------

## 🛠 Requirements

-   MinGW g++ (Windows)
-   C++17 support

------------------------------------------------------------------------

## 📦 Compilation (MinGW)

Open a terminal in the project folder and run:

    g++ -std=c++17 room_booking.cpp -O2 -o room_booking.exe

------------------------------------------------------------------------

## ▶ Running the Program

    .oom_booking.exe

You will see the menu:

    Commands:
     1 - Add Room
     2 - Remove Room
     3 - List Rooms
     4 - Book Specific Room
     5 - Book Best-Fit Room
     6 - Cancel Booking
     7 - List All Bookings
     8 - List Bookings For Room
     9 - Search Available Rooms
     10 - Show Room Status at Time
     11 - Help
     0 - Exit

------------------------------------------------------------------------

## 🎮 Usage Example

### Add Room

    1
    Room number (int): 301
    Capacity (chairs): 15
    Description: Training Hall

### Book Room

    4
    Preferred Room number: 102
    Host name: Alice
    Start time: 10:00
    End time: 12:00
    Chairs required: 8
    Note: Workshop

### List All Bookings

    7

------------------------------------------------------------------------

## 💾 Data Persistence

Currently **no file storage** is used.\
All data is in-memory and resets when the program closes.

(Ask if you want persistent data / saving to file!)

------------------------------------------------------------------------

## 🧩 File Structure

    room_booking.cpp   <-- main program
    README.md          <-- this file

------------------------------------------------------------------------

## 📧 Support

If you need: - A GUI version\
- File saving\
- Color terminal\
- Admin password system\
- Installer

Just ask!

Enjoy your Room Booking System!
