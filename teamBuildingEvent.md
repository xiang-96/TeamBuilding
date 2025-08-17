#include <iostream>
#include <fstream>
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

struct Event {
    string eventId;
    string eventName;
    string eventDate;
    string eventLocation;
    string eventDescription;
    int maxParticipants;
    double originalPrice;
    double discount;
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

struct UserRating {
    string userId;
    int score;
    string comment;
};

struct SystemData {
    vector<User> users;
    vector<Event> events;
};

void displayMainMenu(SystemData &data);
void login(SystemData &data);
void signUp(SystemData &data);
void forgotPassword(SystemData& data);
void checkRememberedUser(SystemData& data);
void saveRememberedUser(const string& userId);
string getRememberedUser();
void userDashboard(SystemData &data, const string &userId);
void displayEvents(const SystemData &data, bool showFeatured = false, bool showDiscounted = false, bool showTopRated = false);
void registerForEvent(SystemData &data, const string &userId);
void provideFeedback(SystemData &data, const string &userId);
void saveUserData(const SystemData &data);
void loadUserData(SystemData &data);
void saveEventData(const SystemData &data);
void loadEventData(SystemData &data);
bool validatePhone(const string &phone);
bool validatePassword(const string &password);
bool validateDate(const string &date);
string getCurrentDate();
void clearScreen();
void displayHeader(const string &title);

int main() {
    SystemData systemData;
    loadUserData(systemData);
    loadEventData(systemData);
    loadDiscountedEvents(systemData);

    displayMainMenu(systemData);
    return 0;
}

void displayMainMenu(SystemData &data) {
    int choice;
    
    do {
        clearScreen();
        cout << "\n _____                    ____                   _ " << endl;
        cout << "|_   _|__  __ _ _ __ ___ / ___| _ __   __ _ _ __| | __" << endl;
        cout << "  | |/ _ \/ _` | '_ ` _ \\___ \| '_ \ / _` | '__| |/ /" << endl;
        cout << "  | |  __/ (_| | | | | | |___) | |_) | (_| | |  |   < " << endl;
        cout << "  |_|\___|\__,_|_| |_| |_|____/| .__/ \__,_|_|  |_|\_\\" << endl;
        cout << "                               |_|                    \n" << endl;
        displayHeader("\n==== Team Building Event System ====\n");
        cout << "1. Login";
        cout << "\n2. Sign Up";
        cout << "\n3. View Featured Events";
        cout << "\n4. View Discounted Events";
        cout << "\n5. View Top Rated";
        cout << "\n6. Exit";
        cout << string(40, "-") << endl;
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

void login(SystemData &data) {
    clearScreen();
    displayHeader("\n==== Login ====\n");
    string rememberedUserId = getRememberedUser();
    if (!rememberedUserId.empty()) {
        for (const auto &user : data.users) {
            if (user.userId == rememberedUserId && user.rememberMe) {
                cout << "Welcome back, " << user.userName << "!\n";
                cout << "Logging you in automatically...\n";
                system("pause");
                userDashboard(data, user.userId);
                return;
            }
        }
    }
    string phoneNum, password;
    cout << "Enter Phone Number (" << PHONE_NUM_LENGTH << " digits): ";
    cin >> phoneNum;

    while (!validatePhoneNum(phoneNum)) {
        cout << "Invalid phone number format! Please try again: ";
        cin >> phoneNum;
    }

    cout << "Enter Password: ";
    cin >> password;

    string userId;
    bool found = false;
    for (const auto &user : data.users) {
        if (user.phoneNum == phoneNum && user.password == password) {
            userId = user.userId;
            found = true;

            // Remember me functionality
            char remember;
            cout << "Remember me? (y/n): ";
            cin >> remember;
            if (tolower(remember) == 'y') {
                for (auto &u : data.users) {
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
        cout << "Login successful! Welcome, " << data.userName << "!" << endl;
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
    displayHeader("\n==== Sign Up ====");

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

        if (!validatePhone(newUser.phoneNum)) {
            cout << "Invalid phone number format.\n";
            phoneExists = true;
            continue;
        }

        for (const auto &user : data.users) {
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
    displayHeader("\n==== Reset Password ====\n");
    string phoneNum, newPassword;
    cout << "Enter your registered phone number: ";
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

void saveRememberedUser(const string& userId) {
    ofstream outFile("remembered_user.txt");
    if (outFile.is_open()) {
        outFile << userId;
        outFile.close();
    }
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

void clearRememberedUser() {
    ofstream outFile("remembered_user.txt");
    outFile.close();
}

void userDashboard(SystemData &data, const string &userId) {
    int choice;
    do {
        clearScreen();
        displayHeader("\n==== User Dashboard ====");

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
    } while (choice != 2);
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
        displayHeader("\n=== Featured Events ===\n");
    }
    else if (showDiscounted) {
        displayHeader("\n=== Discounted Events ===\n");
    }
    else if (showTopRated) {
        displayHeader("\n=== Top Rated Events ===\n");
    }
    else {
        displayHeader("\n=== All Events ===\n");
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
            return a.second, b.second;
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
                cout << "\nRating: " << fixed << setprecision(1) << rating << "5.0\n";
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

void loadDiscountedEvents() {
    ifstream discountFile(DISCOUNT_FILE);
    if (discountFile.is_open()) {
        string line;
        while (getline(discountFile, line)) {
            size_t pos = line.find("|");
            string eventId = line.substr(0, pos);
            double discount = stod(line.substr(pos + 1));

            // Update events with discounts
            for (auto &event : events) {
                if (event.eventId == eventId) {
                    event.discountedPrice = event.originalPrice - discount;
                    break;
                }
            }
        }
        discountFile.close();
    }
}

void provideFeedback(User& currentUser, vector<Event>& events, vector<User>& users) {
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
    saveFeedback(currentUser.userId, feedback, rating);
    cout << "Thank you for your feedback!\n" << endl;
    system("pause");
}

void saveUserData(const SystemData &data) {
    ofstream outFile(USER_FILE);
    if (outFile.is_open()) {
        for (const auto& user : data.users) {
            outFile << user.userId << "|" << user.userName << "|" << user.phoneNum << "|" << user.companyName << "|" << user.password << "|" << user.rememberMe << "\n";
            
            // Save registered events
            for (const auto &eventId : user.registeredEvents) {
                outFile << eventId << ",";
            }
            outFile << "|";

            // Save interests
            for (const auto &interest : user.interests) {
                outFile << interest << ",";
            }
            outFile << "\n";
        }
        outFile.close();
    }
    else {
        cerr << "Error saving user data!" << endl;
    }
}

void loadUserData(SystemData &data) {
    ifstream inFile(USER_FILE);
    if (inFile.is_open()) {
        data.users.clear();
        string line;
        while (getline(inFile, line)) {
            User user;
            size_t pos = 0;
            string token;
            const string delimiter = "|";

            // User ID
            pos = line.find(delimiter);
            user.userId = line.substr(0, pos);
            line.erase(0, pos + delimiter.length());

            // User Name
            pos = line.find(delimiter);
            user.userName = line.substr(0, pos);
            line.erase(0, pos + delimiter.length());

            // Company Name
            pos = line.find(delimiter);
            user.companyName = line.substr(0, pos);
            line.erase(0, pos + delimiter.length());

            // Phone Number
            pos = line.find(delimiter);
            user.phoneNum = line.substr(0, pos);
            line.erase(0, pos + delimiter.length());

            // Password
            pos = line.find(delimiter);
            user.password = line.substr(0, pos);
            line.erase(0, pos + delimiter.length());

            // Remember Me
            user.rememberMe = (line == "1");

            // Registered Events
            pos = line.find(delimiter);
            token = line.substr(0, pos);
            size_t subPos = 0;
            while ((subPos = token.find(",")) != string::npos) {
                string eventId = token.substr(0, subPos);
                if (!eventId.empty()) {
                    user.registeredEvents.push_back(eventId);
                }
                token.erase(0, subPos + 1);
            }
            line.erase(0, pos + delimiter.length());

            // Interests
            pos = line.find(delimiter);
            token = line.substr(0, pos);
            subPos = 0;
            while ((subPos = token.find(",")) != string::npos) {
                string interest = token.substr(0, subPos);
                if (!interest.empty()) {
                    user.interests.push_back(interest);
                }
                token.erase(0, subPos + 1);
            }
            data.users.push_back(user);
        }
        inFile.close();
    }
}

void saveFeedback(const string& userId, const string& feedback) {
    ofstream outFile(FEEDBACK_FILE, ios::app);
    if (outFile.is_open()) {
        outFile << userId << "|" << feedback << "\n";
        outFile.close();
    }
    else {
        cerr << "Error saving feedback!" << endl;
    }
}

bool validatePhoneNum(const string &phoneNum) {
    return phoneNum.length() == PHONE_NUM_LENGTH && all_of(phoneNum.begin(), phoneNum.end(), ::isdigit);
}

bool validatePassword(const string &password) {
    return password.length() >= MIN_PASSWORD_LENGTH;
}

bool validateDate(const string &date) {
    if (date.length() != 10) return false;
    if (date[2] != '/' || date[5] != '/') return false;

    string day = date.substr(0, 2);
    string month = date.substr(3, 2);
    string year = date.substr(6, 4);

    return all_of(day.begin(), day.end(), ::isdigit) && all_of(date.begin(), date.end(), ::isdigit) && all_of(year.begin(), year.end(), ::isdigit);
}

void clearScreen() {
    system("cls || clear");
}

void displayHeader(const string &title) {
    cout << string(40, '=') << endl;
    cout << setw(20 + title.length() / 2) << title << endl;
    cout << string(40, '=') << endl;
}
