#include <iostream>
#include <fstream>
#include <sstream>
#include <ctime>
#include <iomanip>
#include <cctype>
#include <limits>
#include <algorithm>
#include <vector>
#include <string>
using namespace std;

const int MAX_EVENTS = 100;
const int MAX_PARTICIPANTS = 500;
const int PHONE_NUM_LENGTH = 10;
const int MIN_PASSWORD_LENGTH = 6;
const string USER_FILE = "user_data.txt";
const string EVENT_FILE = "event_data.txt";
const string FEEDBACK_FILE = "feedback_data.txt";
const string DISCOUNT_FILE = "discounted_event.txt";

struct User {
    string userId;
    string userName;
    string phoneNum;
    string companyName;
    string password;
    bool rememberMe;
    vector<string> registeredEvents;
    vector<string> interests;
};

struct UserRating {
    string userId;
    int score;
    string comment;
};

struct Event {
    string eventId;
    string eventName;
    string eventDate;
    string eventLocation;
    string eventDescription;
    string requiredEquipment;
    int maxParticipants;
    double originalPrice;
    double discount;
    double discountedPrice;
    bool isFeatured;
    bool isTopPick;
    bool isUpcoming;
    vector<UserRating> ratings;
};

struct Feedback {
    string eventId;
    string userId;
    string feedbackText;
    int rating;
    string date;
};

struct SystemData {
    vector<User> users;
    vector<Event> events;
};

void displayHeader(const string &title);
void displayMainMenu(SystemData &data);
void login(SystemData &data);
void signUp(SystemData &data);
void forgotPassword(SystemData& data);
void checkRememberedUser(SystemData& data);
void saveRememberedUser(const string& userId);
void clearRememberedUser();
string getRememberedUser();
void userDashboard(SystemData &data, const string &userId);
void displayEvents(const SystemData &data, bool showFeatured = false, bool showDiscounted = false, bool showTopRated = false);
void registerForEvent(SystemData &data, const string &userId);
void provideFeedback(SystemData &data, const string &userId);
void saveFeedback(const string& userId, const string& feedback, int rating);
void saveUserData(const SystemData &data);
void loadUserData(SystemData &data);
void saveEventData(const SystemData &data);
void loadEventData(SystemData& data);
void loadDiscountedEvents(SystemData& data);
bool validatePhoneNum(const string &phone);
bool validatePassword(const string &password);
bool validateDate(const string &date);
string getCurrentDate();
void clearScreen();

int main() {
    SystemData systemData;
    loadUserData(systemData);
    loadEventData(systemData);
    loadDiscountedEvents(systemData);
    checkRememberedUser(systemData);
    displayMainMenu(systemData);
    return 0;
}

void displayHeader(const string& title) {
    cout << "\n _____                    ____                   _ " << endl;
    cout << "|_   _|__  __ _ _ __ ___ / ___| _ __   __ _ _ __| | __" << endl;
    cout << "  | |/ _ \\/ _` | '_ ` _ \\___ \\| '_ \\ / _` | '__| |/ /" << endl;
    cout << "  | |  __/ (_| | | | | | |___) | |_) | (_| | |  |   < " << endl;
    cout << "  |_|\\___|\\__,_|_| |_| |_|____/| .__/ \\__,_|_|  |_|\\_\\" << endl;
    cout << "                               |_|                    \n" << endl;
    cout << string(40, '=') << endl;
    cout << setw(20 + title.length() / 2) << title << endl;
    cout << string(40, '=') << endl;
}

void displayMainMenu(SystemData& data) {
    int choice;
    
    do {
        clearScreen();
        displayHeader("\t\tTeam Building Event System");
        cout << "\t\tTeam Building Event System";
        cout << "\n1. Login";
        cout << "\n2. Sign Up";
        cout << "\n3. View Featured Events";
        cout << "\n4. View Discounted Events";
        cout << "\n5. View Top Rated";
        cout << "\n6. Exit";
        cout << string(40, '=') << endl;
        cout << "\nEnter your choice (1-6): ";
        cin >> choice;

        switch (choice) {
        case 1:
            login(data);
            break;
        case 2:
            signUp(data);
            break;
        case 3:
            displayEvents(data, true);
            break;
        case 4:
            displayEvents(data, false, true);
            break;
        case 5:
            displayEvents(data, false, false, true);
            break;
        case 6:
            saveUserData(data);
            saveEventData(data);
            cout << "Saving data and exiting...\n" << endl;
            exit(0);
        default:
            cout << "Invalid choice! Please try again: " << endl;
        }
    } while (choice != 6);
}

void login(SystemData& data) {
    clearScreen();
    displayHeader("Team Building Event System");
    cout << "\t\tLogin";

    string phoneNum, password;
    cout << "\nEnter Phone Number (" << PHONE_NUM_LENGTH << " digits): ";
    cin >> phoneNum;

    while (!validatePhoneNum(phoneNum)) {
        cout << "Invalid phone number format! Please try again: ";
        cin >> phoneNum;
    }

    cout << "Enter Password: ";
    cin >> password;

    string userId;
    string userName;
    bool found = false;
    for (const auto& user : data.users) {
        if (user.phoneNum == phoneNum && user.password == password) {
            userId = user.userId;
            userName = user.userName;
            found = true;

            // Remember me functionality
            char remember;
            cout << "Remember me? (y/n): ";
            cin >> remember;
            if (tolower(remember) == 'y') {
                for (auto& u : data.users) {
                    if (u.userId == userId) {
                        u.rememberMe = true;
                        saveRememberedUser(userId);
                        break;
                    }
                }
            }
            break;
        }
    }

    if (found) {
        cout << "Login successful! Welcome, " << userName << "!" << endl;
        system("pause");
        userDashboard(data, userId);
    }
    else {
        cout << "\nLogin failed! Invalid user ID or password." << endl;
        cout << "\n1. Try again";
        cout << "\n2. Forgot Password";
        cout << "\n3. Back to Main Menu";
        int choice;
        cout << "\nEnter your choice (1-3): ";
        cin >> choice;

        switch (choice) {
        case 1:
            login(data);
            break;
        case 2:
            forgotPassword(data);
            break;
        case 3:
            displayMainMenu(data);
            break;
        default:
            cout << "Invalid choice! Please try again: " << endl;
        }
    }
}

void signUp(SystemData &data) {
    clearScreen();
    User newUser;
    displayHeader("Team Building Event System");
    cout << "\t\tSign Up ====";

    cout << "\nEnter User ID: ";
    cin >> newUser.userId;

    cin.ignore();
    cout << "Enter User Name: ";
    getline(cin, newUser.userName);

    cout << "Enter Company Name: ";
    getline(cin, newUser.companyName);

    bool phoneExists;
do {
    phoneExists = false;
    cout << "Enter Phone Number (" << PHONE_NUM_LENGTH << " digits): ";
    cin >> newUser.phoneNum;

    if (!validatePhoneNum(newUser.phoneNum)) {
        cout << "Invalid phone number format.\n";
        phoneExists = true;
        continue;
    }

    for (const auto& user : data.users) {
        if (user.phoneNum == newUser.phoneNum) {
            phoneExists = true;
            cout << "This phone number is already registered. Please try again: ";
            phoneExists = true;
            break;
        }
    }
} while (phoneExists);

    do {
        cout << "Enter Password (min " << MIN_PASSWORD_LENGTH << " chars): ";
        cin >> newUser.password;
    } while (!validatePassword(newUser.password));
    
    data.users.push_back(newUser);
    saveUserData(data);

    cout << "Registration successful! Welcome, " << newUser.userName << "!" << endl;
    system("pause");
}

void forgotPassword(SystemData &data) {
    clearScreen();
    displayHeader("Team Building Event System");
    cout << "\t\tReset Password";
    string phoneNum, newPassword;
    cout << "\nEnter your registered phone number: ";
    cin >> phoneNum;

    while (!validatePhoneNum(phoneNum)) {
        cout << "Invalid phone number. Please try again: ";
        cin >> phoneNum;
    }

    // Find user by phone number
    User *targetUser = nullptr;
    for (auto &user : data.users) {
        if (user.phoneNum == phoneNum) {
            targetUser = &user;
            break;
        }
    }
    if (targetUser) {
        string newPassword, confirmPassword;
        do {
            cout << "\nEnter new password (min " << MIN_PASSWORD_LENGTH << " chars): ";
            cin >> newPassword;
            cout << "Confirm new password: ";
            cin >> confirmPassword;

            if (newPassword != confirmPassword) {
                cout << "Password not match!\n";
            }
            else if (!validatePassword(newPassword)) {
                cout << "Error: Password must be at least " << MIN_PASSWORD_LENGTH << " characters!\n";
            }
        } while (newPassword != confirmPassword || !validatePassword(newPassword));

        targetUser->password = newPassword;
        targetUser->rememberMe = false;
        saveUserData(data);

        cout << "\nPassword changed successfully!\n";
        cout << "You can now login with your new password.\n";
    }
    else {
        cout << "\nError: Phone number not found in our system.";
    }

    cout << "\nPress enter to return...";
    cin.ignore();
    cin.get();
}

void checkRememberedUser(SystemData& data) {
    string rememberedUserId = getRememberedUser();

    if (!rememberedUserId.empty()) {
        User* rememberedUser = nullptr;
        for (auto& user : data.users) {
            if (user.userId == rememberedUserId && user.rememberMe) {
                rememberedUser = &user;
                break;
            }
        }

        if (rememberedUser != nullptr) {
            cout << "\nWelcome back, " << rememberedUser->userName << "!" << endl;
            cout << "Would you like to login automatically? (y/n): ";

            char choice;
            cin >> choice;

            if (tolower(choice) == 'y') {
                cout << "Logging you in automatically..." << endl;
                system("pause");
                userDashboard(data, rememberedUser->userId);
            }
            else {
                cout << "Returning to main menu..." << endl;
                clearRememberedUser();
                system("pause");
            }
        }
        else {
            cout << "Remembered user not found or 'Remember me' was not enabled." << endl;
            clearRememberedUser();
            system("pause");
        }
    }
}

void saveRememberedUser(const string& userId) {
    ofstream outFile("remembered_user.txt");
    if (outFile.is_open()) {
        outFile << userId;
        outFile.close();
    }
}

void clearRememberedUser() {
    ofstream outFile("remembered_user.txt");
    outFile.close();
}

string getRememberedUser() {
    ifstream inFile("remembered_user.txt");
    string userId;
    if (inFile.is_open()) {
        getline(inFile, userId);
        inFile.close();
    }
    return userId;
}

void userDashboard(SystemData &data, const string &userId) {
    int choice;
    do {
        clearScreen();
        displayHeader("Team Building Event System");
        cout << "\t\tUser Dashboard";

        // Find current user
        const User* currentUser = nullptr;
        for (const auto &user : data.users) {
            if (user.userId == userId) {
                currentUser = &user;
                break;
            }
        }

        if (!currentUser) {
            cout << "User not found!\n";
            return;
        }

        cout << "\nWelcome, " << currentUser->userName << "!\n";
        cout << "\n1. View All Events";
        cout << "\n2. Register for Event";
        cout << "\n3. Provide Feedback for an Event";
        cout << "\n4. View My Events";
        cout << "\n5. Logout";
        cout << "\nEnter your choice (1-5): ";
        cin >> choice;

        switch (choice) {
        case 1:
            displayEvents(data);
            break;
        case 2:
            registerForEvent(data, userId);
            break;
        case 3:
            provideFeedback(data, userId);
            break;
        case 4:
            // view registered events
            break;
        case 5:
            cout << "Logging out..." << endl;
            saveUserData(data);
            displayMainMenu(data);
            break;
        default:
            cout << "Invalid choice! Please try again: " << endl;
        }
    } while (choice != 5);
}

//void organizerDashboard(vector<Event>& events, vector<User>& users) {
//    int choice;
//    do {
//        cout << "\n==== Organizer Dashboard ====";
//        cout << "\n1. Create Event";
//        cout << "\n2. View Events";
//        cout << "\n3. View Feedback";
//        cout << "\n4. Logout";
//        cout << "\nEnter your choice: ";
//        cin >> choice;
//
//        switch (choice) {
//        case 1:
//            createEvent(events);
//            break;
//        case 2:
//            viewEvents(events);
//            break;
//        case 3:
//            viewFeedback();
//            break;
//        case 4:
//            cout << "Logging out..." << endl;
//            saveEventData(events);
//            displayMainMenu();
//            break;
//        default:
//            cout << "Invalid choice! Please try again: " << endl;
//        }
//    } while (choice != 4);
//}

void displayEvents(const SystemData& data, bool showFeatured, bool showDiscounted, bool showTopRated) {
    clearScreen();

    if (showFeatured) {
        displayHeader("Team Building Event System");
        cout << "\t\tFeatured Events\n";
    }
    else if (showDiscounted) {
        displayHeader("Team Building Event System");
        cout << "\t\t Discounted Events\n";
    }
    else if (showTopRated) {
        displayHeader("Team Building Event System");
        cout << "\t\tTop Rated Events\n";
    }
    else {
        displayHeader("Team Building Event System");
        cout << "\t\tAll Events\n";
    }

    vector<Event> filteredEvents;
    for (const auto &event : data.events) {
        if ((showFeatured && event.isFeatured) || (showDiscounted && event.discount > 0) || (!showFeatured && !showDiscounted && !showTopRated)) {
            filteredEvents.push_back(event);
        }
    }

    // For top rated, we need to calculate ratings first
    if (showTopRated) {
        vector<pair<Event, double>> ratedEvents;
        for (const auto &event : data.events) {
            double totalRating = 0;
            int ratingCount = 0;
            for (const auto &rating : event.ratings) {
                totalRating += rating.score;
                ratingCount++;
            }
            if (ratingCount > 0 && (totalRating / ratingCount) >= 4.0) {
                ratedEvents.emplace_back(event, totalRating / ratingCount);
            }
        }

        // Sort by rating (high to low)
        sort(ratedEvents.begin(), ratedEvents.end(), [](const pair<Event, double>& a, const pair<Event, double>& b) {
            return a.second > b.second;
            });

        // Display top rated events
        if (!ratedEvents.empty()) {
            for (const auto &[event, rating] : ratedEvents) {
                cout << "[" << event.eventId << "] " << event.eventName << "\n";
                cout << "Date: " << event.eventDate << " | Location: " << event.eventLocation << "\n";
                cout << "Max Participants: " << event.maxParticipants;
                cout << "Price: RM" << fixed << setprecision(2) << (event.originalPrice - event.discount);
                if (event.discount > 0) {
                    cout << " (Save RM" << event.discount << ")";
                }
                cout << "\nRating: " << fixed << setprecision(1) << rating << "/5.0\n";
                cout << string(40, '-') << endl;
            }
        }
        else {
            cout << "No top rated events currently. Check back later.\n";
        }
    }
    else {
        // Display other types of events
        if (!filteredEvents.empty()) {
            for (const auto &event : filteredEvents) {
                cout << "[" << event.eventId << "] " << event.eventName << "\n";
                cout << "Date: " << event.eventDate << " | Location: " << event.eventLocation << "\n";
                cout << "Max Participants: " << event.maxParticipants << "\n";
                cout << "Price: RM" << fixed << setprecision(2) << (event.originalPrice - event.discount);
                if (event.discount > 0) {
                    cout << " (Save RM" << event.discount << ")";
                } 
                if (showFeatured) {
                    cout << "\nFeatured Event!";
                }
                cout << "\n" << string(40, '-') << endl;
            }
        }
        else {
            cout << "No events found in this category.\n";
        }
    }

    cout << "\nPress enter to return...";
    cin.ignore();
    cin.get();
}

void registerForEvent(SystemData &data, const string &userId) {

}

void provideFeedback(SystemData& data, const string& userId) {
    cin.ignore();
    string feedback;
    int rating;
    cout << "\n==== Provide Feedback ====";
    cout << "\nEnter your feedback about the event: ";
    getline(cin, feedback);
    cout << "Enter your rating (1-5): ";
    cin >> rating;
    while (rating < 1 || rating > 5) {
        cout << "Invalid rating! Please try again: ";
        cin >> rating;
    }
    saveFeedback(userId, feedback, rating);
    cout << "Thank you for your feedback!\n" << endl;
    system("pause");
}

void saveFeedback(const string& userId, const string& feedback, int rating) {
    ofstream outFile(FEEDBACK_FILE, ios::app);
    if (outFile.is_open()) {
        outFile << userId << "|" << feedback << "|" << rating << "\n";
        outFile.close();
    }
    else {
        cerr << "Error saving feedback!" << endl;
    }
}

void saveUserData(const SystemData& data) {
    ofstream outFile(USER_FILE);
    if (!outFile.is_open()) {
        cerr << "Error saving user data!\n";
        return;
    }

    for (const auto& user : data.users) {
        outFile << user.userId << "|" << user.userName << "|" << user.phoneNum << "|" << user.companyName << "|" << user.password << "|" << (user.rememberMe ? "1" : "0") << "|";
            
        // Save registered events
        for (const auto &eventId : user.registeredEvents) outFile << eventId << ",";
        outFile << "|";

        // Save interests
        for (const auto& interest : user.interests) outFile << interest << ",";
        outFile << "\n";
    }
    outFile.close();
}

void loadUserData(SystemData& data) {
    ifstream inFile(USER_FILE);
    if (!inFile.is_open()) return;

    data.users.clear();
    string line;
    while (getline(inFile, line)) {
        if (line.empty()) continue;

        User user;
        stringstream ss(line);
        string token;

        getline(ss, user.userId, '|');
        getline(ss, user.userName, '|');
        getline(ss, user.phoneNum, '|');
        getline(ss, user.companyName, '|');
        getline(ss, user.password, '|');
        getline(ss, token, '|'); // rememberMe
        user.rememberMe = (token == "1");
        
        // Registered events
        getline(ss, token, '|'); 
        stringstream evs(token);
        while (getline(evs, token, ',')) {
            if (!token.empty()) user.registeredEvents.push_back(token);
        }

        // Interests
        if (getline(ss, token, '|')) {
            stringstream ints(token);
            while (getline(ints, token, ',')) {
                if (!token.empty()) user.interests.push_back(token);
            }
        }
        data.users.push_back(user);
    }
    inFile.close();
}

void saveEventData(const SystemData& data) {

}

void loadEventData(SystemData& data) {

}

void loadDiscountedEvents(SystemData& data) {
    ifstream discountFile(DISCOUNT_FILE);
    if (discountFile.is_open()) {
        string line;
        while (getline(discountFile, line)) {
            size_t pos = line.find("|");
            string eventId = line.substr(0, pos);
            double discount = stod(line.substr(pos + 1));

            // Update events with discounts
            for (auto& event : data.events) {
                if (event.eventId == eventId) {
                    event.discount = discount;
                    event.discountedPrice = event.originalPrice - discount;
                    break;
                }
            }
        }
        discountFile.close();
    }
}

bool validatePhoneNum(const string &phoneNum) {
    return phoneNum.length() == PHONE_NUM_LENGTH && all_of(phoneNum.begin(), phoneNum.end(), ::isdigit);
}

bool validatePassword(const string &password) {
    return password.length() >= MIN_PASSWORD_LENGTH;
}

bool validateDate(const string& date) {
    if (date.length() != 10) return false;
    if (date[2] != '/' || date[5] != '/') return false;

    string day = date.substr(0, 2);
    string month = date.substr(3, 2);
    string year = date.substr(6, 4);

    return all_of(day.begin(), day.end(), ::isdigit) && all_of(month.begin(), month.end(), ::isdigit) && all_of(year.begin(), year.end(), ::isdigit);
}

string getCurrentDate() {
    time_t now = time(0);
    tm ltm;
    localtime_s(&ltm, &now);
    stringstream ss;
    ss << setw(2) << setfill('0') << ltm.tm_mday << "/" << setw(2) << setfill('0') << (1 + ltm.tm_mon) << "/" << (1900 + ltm.tm_year);
    return ss.str();
}

void clearScreen() {
#ifdef _WIN32
    system("cls");
#else
    system("clear");
#endif
}
