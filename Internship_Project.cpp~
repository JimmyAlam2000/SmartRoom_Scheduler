// room_booking.cpp
// Compile (MinGW): g++ -std=c++17 room_booking.cpp -O2 -o room_booking.exe
// Run from terminal: .\room_booking.exe

#include <bits/stdc++.h>
using namespace std;

/*
 Advanced Room Allotment & Booking CLI (MinGW-friendly)
 - Uses ordered maps and event-sweep per room for capacity-aware scheduling.
 - Booking IDs are incremental integers.
 - Time is handled as minutes since midnight for easy arithmetic.
 - Robust trimming of CRLF and safe getline usage.
*/

static string trim_crlf_and_spaces(string s) {
    // remove '\r' (Windows CR) and trim leading/trailing whitespace
    s.erase(remove(s.begin(), s.end(), '\r'), s.end());
    size_t p = 0;
    while (p < s.size() && isspace((unsigned char)s[p])) ++p;
    size_t q = s.size();
    while (q > p && isspace((unsigned char)s[q-1])) --q;
    return s.substr(p, q - p);
}

struct Time {
    int minutes; // minutes since 00:00
    Time(): minutes(0) {}
    Time(int m): minutes(m) {}
    static bool valid_hm(int h, int m) {
        return (h>=0 && h<24 && m>=0 && m<60);
    }
    static Time from_hm(int h, int m) {
        return Time(h*60 + m);
    }
    static Time parse(const string &raw) {
        string s = raw;
        s.erase(remove(s.begin(), s.end(), '\r'), s.end()); // defensive
        int h=0, m=0;
        char c;
        stringstream ss(s);
        ss >> h >> c >> m;
        if (!ss || c!=':' || !valid_hm(h,m)) throw runtime_error("Invalid time format");
        return from_hm(h,m);
    }
    string str() const {
        int h = minutes / 60;
        int m = minutes % 60;
        char buf[6];
        snprintf(buf, sizeof(buf), "%02d:%02d", h, m);
        return string(buf);
    }
    bool operator<(const Time& o) const { return minutes < o.minutes; }
    bool operator<=(const Time& o) const { return minutes <= o.minutes; }
    bool operator==(const Time& o) const { return minutes == o.minutes; }
};

struct Booking {
    long long id;
    int roomNumber;
    string host;
    Time start;
    Time end; // exclusive: booking covers [start, end)
    int chairs;
    string note;
    Booking() = default;
    Booking(long long id_, int rn, const string &host_, Time s, Time e, int ch, const string &note_)
        : id(id_), roomNumber(rn), host(host_), start(s), end(e), chairs(ch), note(note_) {}
};

struct Room {
    int number;
    int capacity; // chairs available
    string description;
    map<int,int> events; // time -> delta chairs
    unordered_map<long long, Booking> bookings;

    Room() = default;
    Room(int number_, int capacity_, const string &desc_)
        : number(number_), capacity(capacity_), description(desc_) {}

    bool can_book(Time s, Time e, int chairs) const {
        if (s.minutes >= e.minutes) return false;
        int curr = 0;
        if (!events.empty()) {
            auto it2 = events.upper_bound(s.minutes - 1); // events <= s-1
            for (auto it3 = events.begin(); it3 != it2; ++it3) curr += it3->second;
        }
        auto itStart = events.lower_bound(s.minutes);
        int temp = curr;
        if (temp + chairs > capacity) return false;
        for (auto it4 = itStart; it4 != events.end() && it4->first < e.minutes; ++it4) {
            temp += it4->second;
            if (temp + chairs > capacity) return false;
        }
        return true;
    }

    void add_booking(const Booking &bk) {
        events[bk.start.minutes] += bk.chairs;
        events[bk.end.minutes] -= bk.chairs;
        bookings[bk.id] = bk;
    }

    bool remove_booking(long long bookingId) {
        auto it = bookings.find(bookingId);
        if (it == bookings.end()) return false;
        Booking bk = it->second;
        events[bk.start.minutes] -= bk.chairs;
        events[bk.end.minutes] += bk.chairs;
        if (events[bk.start.minutes] == 0) events.erase(bk.start.minutes);
        if (events[bk.end.minutes] == 0) events.erase(bk.end.minutes);
        bookings.erase(it);
        return true;
    }

    int chairs_at(Time t) const {
        int sum = 0;
        for (auto &p : events) {
            if (p.first > t.minutes) break;
            sum += p.second;
        }
        return sum;
    }

    vector<Booking> list_bookings() const {
        vector<Booking> out;
        out.reserve(bookings.size());
        for (auto &kv : bookings) out.push_back(kv.second);
        sort(out.begin(), out.end(), [](const Booking &a, const Booking &b){
            if (a.start.minutes != b.start.minutes) return a.start.minutes < b.start.minutes;
            return a.id < b.id;
        });
        return out;
    }
};

struct RoomManager {
    map<int, Room> rooms;
    unordered_map<long long,int> bookingIndex;
    long long nextBookingId = 1;

    bool add_room(int number, int capacity, const string &desc) {
        if (rooms.count(number)) return false;
        rooms[number] = Room(number, capacity, desc);
        return true;
    }
    bool remove_room(int number) {
        auto it = rooms.find(number);
        if (it == rooms.end()) return false;
        if (!it->second.bookings.empty()) return false;
        rooms.erase(it);
        return true;
    }
    long long book_room_specific(int roomNumber, const string &host, Time s, Time e, int chairs, const string &note) {
        auto it = rooms.find(roomNumber);
        if (it == rooms.end()) return -1;
        Room &r = it->second;
        if (!r.can_book(s,e,chairs)) return -1;
        long long id = nextBookingId++;
        Booking bk(id, roomNumber, host, s, e, chairs, note);
        r.add_booking(bk);
        bookingIndex[id] = roomNumber;
        return id;
    }
    long long book_best_fit(const string &host, Time s, Time e, int chairs, const string &note) {
        vector<pair<int,int>> candidates;
        for (auto &kv : rooms) {
            const Room &r = kv.second;
            if (r.capacity >= chairs && r.can_book(s,e,chairs)) {
                candidates.emplace_back(r.capacity, r.number);
            }
        }
        if (candidates.empty()) return -1;
        sort(candidates.begin(), candidates.end());
        int chosenRoom = candidates[0].second;
        return book_room_specific(chosenRoom, host, s, e, chairs, note);
    }
    bool cancel_booking(long long id) {
        auto it = bookingIndex.find(id);
        if (it == bookingIndex.end()) return false;
        int rn = it->second;
        auto rit = rooms.find(rn);
        if (rit == rooms.end()) return false;
        bool ok = rit->second.remove_booking(id);
        if (ok) bookingIndex.erase(it);
        return ok;
    }
    vector<Booking> list_all_bookings() const {
        vector<Booking> out;
        for (auto &kv : rooms) {
            const Room &r = kv.second;
            auto bks = r.list_bookings();
            out.insert(out.end(), bks.begin(), bks.end());
        }
        sort(out.begin(), out.end(), [](const Booking &a, const Booking &b){
            if (a.start.minutes != b.start.minutes) return a.start.minutes < b.start.minutes;
            if (a.roomNumber != b.roomNumber) return a.roomNumber < b.roomNumber;
            return a.id < b.id;
        });
        return out;
    }
    vector<int> available_rooms(Time s, Time e, int chairs) const {
        vector<int> res;
        for (auto &kv : rooms) {
            const Room &r = kv.second;
            if (r.capacity >= chairs && r.can_book(s,e,chairs)) res.push_back(r.number);
        }
        return res;
    }
    void print_status_at(Time t) const {
        cout << "Status at " << t.str() << ":\n";
        for (auto &kv : rooms) {
            const Room &r = kv.second;
            int used = r.chairs_at(t);
            cout << " Room " << r.number << " (" << r.capacity << " chairs) - used: " << used
                 << " free: " << r.capacity - used << "\n";
        }
    }
};

// Utilities for CLI
static string read_nonempty_line(const string &prompt = "") {
    string s;
    while (true) {
        if (!prompt.empty()) {
            cout << prompt;
            cout.flush();
        }
        if (!getline(cin, s)) return string(); // EOF or error
        s = trim_crlf_and_spaces(s);
        if (!s.empty()) return s;
        cout << "Please enter a non-empty value.\n";
    }
}

static int read_int(const string &prompt, int minv = INT_MIN, int maxv = INT_MAX) {
    while (true) {
        string s = read_nonempty_line(prompt);
        if (s.empty()) return minv; // in case of EOF, return a safe value (caller should handle)
        try {
            long long x = stoll(s);
            if (x < minv || x > maxv) {
                cout << "Enter integer between " << minv << " and " << maxv << ".\n";
                continue;
            }
            return (int)x;
        } catch (...) {
            cout << "Invalid integer, try again.\n";
        }
    }
}

static Time read_time(const string &prompt) {
    while (true) {
        string s = read_nonempty_line(prompt);
        if (s.empty()) throw runtime_error("EOF while reading time");
        try {
            return Time::parse(s);
        } catch (const exception &e) {
            cout << "Invalid time (expected HH:MM). Try again.\n";
        }
    }
}

static void print_booking(const Booking &b) {
    cout << "BookingID: " << b.id << " | Room: " << b.roomNumber << " | Host: " << b.host
         << " | " << b.start.str() << " - " << b.end.str() << " | Chairs: " << b.chairs;
    if (!b.note.empty()) cout << " | Note: " << b.note;
    cout << "\n";
}

static void show_help() {
    cout << "Commands:\n";
    cout << " 1 - Add Room\n 2 - Remove Room\n 3 - List Rooms\n 4 - Book Specific Room\n 5 - Book Best-Fit Room\n 6 - Cancel Booking\n 7 - List All Bookings\n 8 - List Bookings For Room\n 9 - Search Available Rooms\n 10 - Show Room Status at Time\n 11 - Help\n 0 - Exit\n";
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);
    cout.tie(nullptr);

    cout << "Advanced Room Allotment & Booking CLI (MinGW-friendly)\n";
    cout << "Project brief loaded.\n";
    cout.flush();

    RoomManager manager;
    // Seed with demo rooms
    manager.add_room(101, 10, "Small meeting room");
    manager.add_room(102, 20, "Conference room");
    manager.add_room(201, 6, "Huddle room");

    show_help();
    cout.flush();

    while (true) {
        cout << "\nEnter command (11 for help): ";
        cout.flush();
        string cmdline;
        if (!getline(cin, cmdline)) { // EOF or stream closed
            cout << "\nInput closed. Exiting.\n";
            break;
        }
        cmdline = trim_crlf_and_spaces(cmdline);
        if (cmdline.empty()) continue;
        int cmd = -1;
        try { cmd = stoi(cmdline); } catch(...) { cout << "Invalid command\n"; continue; }

        if (cmd == 0) {
            cout << "Bye.\n";
            break;
        } else if (cmd == 1) {
            int rn = read_int("Room number (int): ");
            int cap = read_int("Capacity (chairs): ", 1, 1000000);
            string desc = read_nonempty_line("Description: ");
            if (manager.add_room(rn, cap, desc)) cout << "Added room " << rn << "\n";
            else cout << "Room " << rn << " already exists.\n";

        } else if (cmd == 2) {
            int rn = read_int("Room number to remove: ");
            if (manager.remove_room(rn)) cout << "Removed room " << rn << "\n";
            else cout << "Unable to remove room (not found or has bookings).\n";

        } else if (cmd == 3) {
            cout << "Rooms:\n";
            for (auto &kv : manager.rooms) {
                const Room &r = kv.second;
                cout << " Room " << r.number << " | cap: " << r.capacity << " | desc: " << r.description
                     << " | bookings: " << r.bookings.size() << "\n";
            }

        } else if (cmd == 4) {
            int rn = read_int("Preferred Room number: ");
            string host = read_nonempty_line("Host name: ");
            Time s = read_time("Start time (HH:MM): ");
            Time e = read_time("End time (HH:MM): ");
            int chairs = read_int("Chairs required: ", 1, 1000000);
            string note = read_nonempty_line("Note (optional, or '-' for none): ");
            if (note == "-") note.clear();
            long long id = manager.book_room_specific(rn, host, s, e, chairs, note);
            if (id < 0) cout << "Booking failed: room unavailable or doesn't exist.\n";
            else cout << "Booked. Booking ID: " << id << "\n";

        } else if (cmd == 5) {
            string host = read_nonempty_line("Host name: ");
            Time s = read_time("Start time (HH:MM): ");
            Time e = read_time("End time (HH:MM): ");
            int chairs = read_int("Chairs required: ", 1, 1000000);
            string note = read_nonempty_line("Note (optional, or '-' for none): ");
            if (note == "-") note.clear();
            long long id = manager.book_best_fit(host, s, e, chairs, note);
            if (id < 0) cout << "Booking failed: no suitable room available in that period.\n";
            else {
                int rn = manager.bookingIndex[id];
                cout << "Booked Room " << rn << ". Booking ID: " << id << "\n";
            }

        } else if (cmd == 6) {
            long long id = read_int("Booking ID to cancel: ", 1, INT_MAX);
            if (manager.cancel_booking(id)) cout << "Cancelled booking " << id << "\n";
            else cout << "Cancellation failed: booking not found.\n";

        } else if (cmd == 7) {
            cout << "All bookings:\n";
            auto all = manager.list_all_bookings();
            if (all.empty()) cout << " (no bookings)\n";
            for (auto &b : all) print_booking(b);

        } else if (cmd == 8) {
            int rn = read_int("Room number: ");
            auto it = manager.rooms.find(rn);
            if (it == manager.rooms.end()) { cout << "No such room.\n"; continue; }
            auto bks = it->second.list_bookings();
            cout << "Bookings for room " << rn << ":\n";
            if (bks.empty()) cout << " (none)\n";
            for (auto &b : bks) print_booking(b);

        } else if (cmd == 9) {
            Time s = read_time("Start time (HH:MM): ");
            Time e = read_time("End time (HH:MM): ");
            int chairs = read_int("Chairs required: ", 1, 1000000);
            auto av = manager.available_rooms(s,e,chairs);
            if (av.empty()) cout << "No rooms available for that interval with given chairs.\n";
            else {
                cout << "Available rooms: ";
                for (int r : av) cout << r << " ";
                cout << "\n";
            }

        } else if (cmd == 10) {
            Time t = read_time("Enter time (HH:MM): ");
            manager.print_status_at(t);

        } else if (cmd == 11) {
            show_help();

        } else {
            cout << "Unknown command. Enter 11 for help.\n";
        }
    }

    return 0;
}
