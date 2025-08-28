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
const string REGISTRATION_FILE = "registrations.txt";

struct UserRating {
    string userId;
    int score;
    string comment;
    string date;
};

struct Event {
    int eventId;
    string eventName;
    string eventDate;
    string eventLocation;
    string eventDescription;
    string requiredEquipment;
    int maxParticipants;
    int currentParticipants;
    double originalPrice;
    double discount;
    double discountedPrice;
    bool isFeatured;
    bool isTopRated;
    bool isUpcoming;
    vector<UserRating> ratings;

    // Calculate average rating
    double getAverageRating() const {
        if (ratings.empty()) return 0.0;
        double total = 0.0;
        for (const auto& rating : ratings) {
            total += rating.score;
        }
        return total / ratings.size();
    }
};

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

struct Registration {
    int registrationID;
    string participantName;
    string participantID;
    string companyName;
    string phoneNumber;
    int eventID;
    string eventName;
    string eventDate;
    string eventVenue;
    string registrationDate;
    double amountPaid;
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
    vector<Registration> registrations;
    vector<Feedback> feedbacks;
    int nextRegistrationID;
    int nextEventID;

    SystemData() : nextRegistrationID(1), nextEventID(1) {}
};

void initializeSystem(SystemData& data);
void displayHeader(const string& title);
void displayMainMenu(SystemData& data);
void login(SystemData& data);
void signUp(SystemData& data);
void forgotPassword(SystemData& data);
void checkRememberedUser(SystemData& data);
void saveRememberedUser(const string& userId);
void clearRememberedUser();
string getRememberedUser();
void userDashboard(SystemData& data, const string& userId);
void organizerDashboard(SystemData& data, const string& userId);
void displayEvents(const SystemData& data, bool showFeatured = false, bool showDiscounted = false, bool showTopRated = false);
void registerForEvent(SystemData& data, const string& userId);
void provideFeedback(SystemData& data, const string& userId);
void saveUserData(const SystemData& data);
void loadUserData(SystemData& data);
void saveEventData(const SystemData& data);
void loadEventData(SystemData& data);
void loadDiscountedEvents(SystemData& data);
void loadRegistrations(SystemData& data);
void saveRegistrations(SystemData& data);
void loadFeedbacks(SystemData& data);
void saveFeedbacks(const SystemData& data);
void displayEvent(const Event& event);
bool isEventFull(const Event& event);
void addSampleData(SystemData& data);
void createEvent(SystemData& data);
void viewEvents(const SystemData& data);
void viewFeedback(const SystemData& data);
void viewMyRegistrations(const SystemData& data, const string& userId);
bool isValidInput(const string& input);
bool validatePhoneNum(const string& phoneNum);
bool validatePassword(const string& password);
bool validateDate(const string& date);
bool isValidPhoneNum(const string& phoneNum);
string getCurrentDate();
string generateParticipantID();
string generateUserID();
vector<Registration> getUserRegistrations(const string& participantID, const SystemData& data);
vector<Registration> getUserRegistrationsByName(const string& participantName, const SystemData& data);
void clearScreen();

int main() {
    SystemData systemData;
    initializeSystem(systemData);
    displayMainMenu(systemData);
    return 0;
}

void initializeSystem(SystemData& data) {
    loadUserData(data);
    loadEventData(data);
    loadDiscountedEvents(data);
    loadRegistrations(data);
    loadFeedbacks(data);
    checkRememberedUser(data);

    // Add sample data if no events exist
    if (data.events.empty()) {
        addSampleData(data);
    }
}

void addSampleData(SystemData& data) {
    // Add sample events
    Event event1;
    event1.eventId = data.nextEventID++;
    event1.eventName = "Team Building Workshop";
    event1.eventDate = "2024-09-15";
    event1.eventLocation = "Conference Room A";
    event1.eventDescription = "A fun team building workshop to improve collaboration";
    event1.requiredEquipment = "Projector, Whiteboard";
    event1.maxParticipants = 50;
    event1.currentParticipants = 0;
    event1.originalPrice = 100.0;
    event1.discount = 10.0;
    event1.discountedPrice = 90.0;
    event1.isFeatured = true;
    event1.isTopRated = true;
    event1.isUpcoming = true;

    Event event2;
    event2.eventId = data.nextEventID++;
    event2.eventName = "Leadership Training";
    event2.eventDate = "2024-09-20";
    event2.eventLocation = "Training Hall B";
    event2.eventDescription = "Develop leadership skills for your team";
    event2.requiredEquipment = "Microphone, Laptop";
    event2.maxParticipants = 30;
    event2.currentParticipants = 0;
    event2.originalPrice = 150.0;
    event2.discount = 0.0;
    event2.discountedPrice = 150.0;
    event2.isFeatured = false;
    event2.isTopRated = true;
    event2.isUpcoming = true;

    Event event3;
    event3.eventId = data.nextEventID++;
    event3.eventName = "Communication Skills";
    event3.eventDate = "2024-09-25";
    event3.eventLocation = "Seminar Room C";
    event3.eventDescription = "Improve communication within your team";
    event3.requiredEquipment = "Flipchart, Markers";
    event3.maxParticipants = 40;
    event3.currentParticipants = 0;
    event3.originalPrice = 80.0;
    event3.discount = 20.0;
    event3.discountedPrice = 60.0;
    event3.isFeatured = true;
    event3.isTopRated = false;
    event3.isUpcoming = true;

    data.events.push_back(event1);
    data.events.push_back(event2);
    data.events.push_back(event3);

    saveEventData(data);
}

void displayHeader(const string& title) {
    cout << "\n _____                    ____                   _ " << endl;
    cout << "|_   _|__  __ _ _ __ ___ / ___| _ __   __ _ _ __| | __" << endl;
    cout << "  | |/ _ \\/ _` | '_ ` _  \\___ \\| '_ \\ / _` | '__| |/ /" << endl;
    cout << "  | |  __/ (_| | | | | | |___) | |_) | (_| | |  |   < " << endl;
    cout << "  |_|\\___|\\__,_|_| |_| |_|____/| .__/ \\__,_|_|  |_|\\_\\" << endl;
    cout << "                               |_|                    \n" << endl;
    cout << "\t" + string(40, '=') << endl;
    cout << "\t" << setw(20 + title.length() / 2) << title << endl;
    cout << "\t" + string(40, '=') << endl;
}

void displayMainMenu(SystemData& data) {
    int choice;
    
    do {
        clearScreen();
        displayHeader("\tTeam Building Event System");
        cout << "\n1. Login";
        cout << "\n2. Sign Up";
        cout << "\n3. View Featured Events";
        cout << "\n4. View Discounted Events";
        cout << "\n5. View Top Rated";
        cout << "\n6. Exit\n";
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
    cout << "\n\tLogin";

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
                for (auto& user : data.users) {
                    if (user.userId == userId) {
                        user.rememberMe = true;
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

        if (userId.substr(0, 2) == "OR") {
            organizerDashboard(data, userId);
        }
        else {
            userDashboard(data, userId);
        }
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

void signUp(SystemData& data) {
    clearScreen();
    User newUser;
    displayHeader("Team Building Event System");
    cout << "\n\tSign Up";

    newUser.userId = generateUserID();
    cout << "\nYour User ID: " << newUser.userId << endl;

    cin.ignore();
    cout << "Enter User Name: ";
    getline(cin, newUser.userName);

    while (!isValidInput(newUser.userName)) {
        cout << "Error: User name cannot be empty! Please try again: ";
        getline(cin, newUser.userName);
    }

    cout << "Enter Company Name: ";
    getline(cin, newUser.companyName);

    while (!isValidInput(newUser.companyName)) {
        cout << "Error: Company name cannot be empty! Please try again: ";
        getline(cin, newUser.companyName);
    }

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
                break;
            }
        }
    } while (phoneExists);

    bool validPassword = false;
    do {
        cout << "Enter Password (min " << MIN_PASSWORD_LENGTH << " chars): ";
        cin >> newUser.password;

        if (!validatePassword(newUser.password)) {
            cout << "Error: Password must be at least " << MIN_PASSWORD_LENGTH
                << " chars! Please try again." << endl;
        }
        else {
            validPassword = true;
        }
    } while (!validPassword);

    newUser.rememberMe = false;
    data.users.push_back(newUser);
    saveUserData(data);

    cout << "Registration successful! Welcome, " << newUser.userName << "!" << endl;
    system("pause");
}

void forgotPassword(SystemData& data) {
    clearScreen();
    displayHeader("Team Building Event System");
    cout << "\n\tReset Password";
    string phoneNum;
    cout << "\nEnter your registered phone number: ";
    cin >> phoneNum;

    while (!validatePhoneNum(phoneNum)) {
        cout << "Invalid phone number. Please try again: ";
        cin >> phoneNum;
    }

    // Find user by phone number
    User *targetUser = nullptr;
    for (auto& user : data.users) {
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
                if (rememberedUser->userId.substr(0, 2) == "OR") {
                    organizerDashboard(data, rememberedUser->userId);
                }
                else {
                    userDashboard(data, rememberedUser->userId);
                }
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

void userDashboard(SystemData& data, const string& userId) {
    int choice;

    // Find current user
    const User* currentUser = nullptr;
    for (const auto& user : data.users) {
        if (user.userId == userId) {
            currentUser = &user;
            break;
        }
    }

    if (!currentUser) {
        cout << "User not found!\n";
        return;
    }

    do {
        clearScreen();
        displayHeader("Team Building Event System");
        cout << "\n\tUser Dashboard";
        cout << "\nWelcome, " << currentUser->userName << "!\n\n";
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
            viewMyRegistrations(data, userId);
            break;
        case 5:
            cout << "Logging out..." << endl;
            saveUserData(data);
            break;
        default:
            cout << "Invalid choice! Please try again: " << endl;
        }
    } while (choice != 5);
}

void organizerDashboard(SystemData& data, const string& userId) {
    int choice;
    do {
        clearScreen();
        displayHeader("Team Building Event System");
        cout << "\n\tOrganizer Dashboard";
        cout << "\n1. Create Event";
        cout << "\n2. View Events";
        cout << "\n3. View Feedback";
        cout << "\n4. Logout";
        cout << "\nEnter your choice: ";
        cin >> choice;

        switch (choice) {
        case 1:
            createEvent(data);
            break;
        case 2:
            viewEvents(data);
            break;
        case 3:
            viewFeedback(data);
            break;
        case 4:
            cout << "Logging out..." << endl;
            saveEventData(data);
            break;
        default:
            cout << "Invalid choice! Please try again: " << endl;
        }
    } while (choice != 4);
}

void createEvent(SystemData& data) {

}

void viewEvents(const SystemData& data) {

}

void viewFeedback(const SystemData& data) {

}

void displayEvents(const SystemData& data, bool showFeatured, bool showDiscounted, bool showTopRated) {
    clearScreen();
    displayHeader("Team Building Event System");

    if (showFeatured) {
        cout << "\n\tFeatured Events\n";
    }
    else if (showDiscounted) {
        cout << "\n\tDiscounted Events\n";
    }
    else if (showTopRated) {
        cout << "\n\tTop Rated Events\n";
    }
    else {
        cout << "\n\tAll Events\n";
    }

    vector<Event> filteredEvents;
    for (const auto& event : data.events) {
        if ((showFeatured && event.isFeatured) || (showDiscounted && event.discount > 0) || (showTopRated && event.getAverageRating() >= 4.0) || (!showFeatured && !showDiscounted && !showTopRated)) {
            filteredEvents.push_back(event);
        }
    }

    if (filteredEvents.empty()) {
        cout << "No events found in this category.\n";
    }
    else {
        for (const auto& event : filteredEvents) {
            displayEvent(event);
        }
    }

    cout << "\nPress enter to return...";
    cin.ignore();
    cin.get();
}

void displayEvent(const Event& event) {
    cout << "[" << event.eventId << "] " << event.eventName << "\n";
    cout << "Date: " << event.eventDate << " | Location: " << event.eventLocation << "\n";
    cout << "Description: " << event.eventDescription << "\n";
    cout << "Max Participants: " << event.maxParticipants;
    cout << " | Registered: " << event.currentParticipants << "\n";
    cout << "Price: RM" << fixed << setprecision(2) << event.discountedPrice;
    if (event.discount > 0) {
        cout << " (Save RM" << event.discount << ")";
    }
    cout << "\nRating: " << fixed << setprecision(1) << event.getAverageRating() << "/5.0";

    if (event.isFeatured) cout << " | Featured Event";
    if (event.isTopRated) cout << " | Top Pick";
    if (event.isUpcoming) cout << " | Upcoming";

    cout << "\n" << string(50, '-') << "\n";
}

void registerForEvent(SystemData& data, const string& userId) {
    clearScreen();
    displayHeader("Team Building Event");
    cout << "\n\tRegister For Event\n";

    if (data.events.empty()) {
        cout << "No events available for registration." << endl;
        system("pause");
        return;
    }

    // Display available events
    cout << "Available Events:" << endl;
    cout << string(60, '-') << endl;
    for (const auto& event : data.events) {
        if (!isEventFull(event)) {
            cout << "[" << event.eventId << "] " << event.eventName;
            cout << " (Available: " << event.maxParticipants - event.currentParticipants << "/" << event.maxParticipants << ")" << endl;
            cout << "   Date: " << event.eventDate << " | Location: " << event.eventLocation << endl;
            cout << "   Price: RM" << fixed << setprecision(2) << event.discountedPrice;
            if (event.discount > 0) {
                cout << " (Save RM" << event.discount << ")";
            }
            cout << endl << string(60, '-') << endl;
        }
    }

    int eventId;
    cout << "\nEnter Event ID to register: ";
    while (!(cin >> eventId)) {
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
        cout << "Invalid input! Please enter a number: ";
    }
    cin.ignore();

    // Find the selected event
    Event* selectedEvent = nullptr;
    for (auto& event : data.events) {
        if (event.eventId == eventId) {
            selectedEvent = &event;
            break;
        }
    }

    if (!selectedEvent) {
        cout << "Event not found!" << endl;
        system("pause");
        return;
    }

    if (isEventFull(*selectedEvent)) {
        cout << "This event is full. Cannot register." << endl;
        system("pause");
        return;
    }

    // Find user details
    User currentUser;
    for (const auto& user : data.users) {
        if (user.userId == userId) {
            currentUser = user;
            break;
        }
    }

    // Create registration
    Registration newReg;
    newReg.registrationID = data.nextRegistrationID++;
    newReg.participantName = currentUser.userName;
    newReg.participantID = currentUser.userId;
    newReg.companyName = currentUser.companyName;
    newReg.phoneNumber = currentUser.phoneNum;
    newReg.eventID = selectedEvent->eventId;
    newReg.eventName = selectedEvent->eventName;
    newReg.eventDate = selectedEvent->eventDate;
    newReg.eventVenue = selectedEvent->eventLocation;
    newReg.registrationDate = getCurrentDate();
    newReg.amountPaid = selectedEvent->discountedPrice;

    data.registrations.push_back(newReg);
    selectedEvent->currentParticipants++;

    // Add to user's registered events
    for (auto& user : data.users) {
        if (user.userId == userId) {
            user.registeredEvents.push_back(to_string(selectedEvent->eventId));
            break;
        }
    }

    saveRegistrations(data);
    saveEventData(data);
    saveUserData(data);

    cout << "\nRegistration successful!" << endl;
    cout << "Registration ID: " << newReg.registrationID << endl;
    cout << "Event: " << newReg.eventName << endl;
    cout << "Date: " << newReg.eventDate << endl;
    cout << "Venue: " << newReg.eventVenue << endl;
    cout << "Amount Paid: RM" << fixed << setprecision(2) << newReg.amountPaid << endl;

    system("pause");
}

void provideFeedback(SystemData& data, const string& userId) {
    clearScreen();
    displayHeader("Team Building Event System");
    cout << "\n\tProvide Feedback\n";
    // Get user's registered events
    vector<int> registeredEventIds;
    for (const auto& user : data.users) {
        if (user.userId == userId) {
            for (const auto& eventIdStr : user.registeredEvents) {
                try {
                    registeredEventIds.push_back(stoi(eventIdStr));
                }
                catch (...) {
                    // Skip invalid event IDs
                }
            }
            break;
        }
    }

    if (registeredEventIds.empty()) {
        cout << "You haven't registered for any events yet.\n";
        system("pause");
        return;
    }

    // Display registered events
    cout << "Your registered events:\n";
    for (int eventId : registeredEventIds) {
        for (const auto& event : data.events) {
            if (event.eventId == eventId) {
                cout << "[" << event.eventId << "] " << event.eventName << "\n";
                break;
            }
        }
    }

    int eventId;
    cout << "\nEnter Event ID to provide feedback: ";
    while (!(cin >> eventId) || find(registeredEventIds.begin(), registeredEventIds.end(), eventId) == registeredEventIds.end()) {
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
        cout << "Invalid Event ID! Please select from your registered events: ";
    }
    cin.ignore();

    Feedback newFeedback;
    newFeedback.eventId = to_string(eventId);
    newFeedback.userId = userId;
    newFeedback.date = getCurrentDate();

    cout << "\nEnter your feedback: ";
    getline(cin, newFeedback.feedbackText);
    while (!(cin >> newFeedback.rating) || newFeedback.rating < 1 || newFeedback.rating > 5) {
        cin.clear();
        cin.ignore(numeric_limits<streamsize>::max(), '\n');
        cout << "Invalid rating! Please enter a number between 1-5: ";
    }

    data.feedbacks.push_back(newFeedback);
    
    // Also add to event's ratings
    for (auto& event : data.events) {
        if (event.eventId == eventId) {
            UserRating rating;
            rating.userId = userId;
            rating.score = newFeedback.rating;
            rating.comment = newFeedback.feedbackText;
            rating.date = newFeedback.date;
            event.ratings.push_back(rating);
            break;
        }
    }

    saveFeedbacks(data);
    saveEventData(data);

    cout << "Thank you for your feedback!\n";
    system("pause");
}

void viewMyRegistrations(const SystemData& data, const string& userId) {

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
        for (const auto& eventId : user.registeredEvents) {
            outFile << eventId << ",";
        }
        outFile << "|";

        // Save interests
        for (const auto& interest : user.interests) {
            outFile << interest << ",";
        }
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
    ifstream inFile(EVENT_FILE);
    if (!inFile.is_open()) {
        return;
    }

    data.events.clear();
    string line;
    while (getline(inFile, line)) {
        if (line.empty()) continue;

        Event event;
        stringstream ss(line);
        string token;

        getline(ss, token, '|');
        event.eventId = stoi(token);
        
        getline(ss, event.eventName, '|');
        getline(ss, event.eventDate, '|');
        getline(ss, event.eventLocation, '|');
        getline(ss, event.eventDescription, '|');
        getline(ss, event.requiredEquipment, '|');

        getline(ss, token, '|');
        event.maxParticipants = stoi(token);

        getline(ss, token, '|');
        event.currentParticipants = stoi(token);

        getline(ss, token, '|');
        event.originalPrice = stod(token);

        getline(ss, token, '|');
        event.discount = stod(token);

        getline(ss, token, '|');
        event.discountedPrice = stod(token);

        getline(ss, token, '|');
        event.isFeatured = (token == "1");

        getline(ss, token, '|');
        event.isTopRated = (token == "1");

        getline(ss, token, '|');
        event.isUpcoming = (token == "1");

        // Load ratings
        getline(ss, token, '|');
        if (!token.empty()) {
            stringstream ratingsStream(token);
            string ratingEntry;
            while (getline(ratingsStream, ratingEntry, ';')) {
                if (!ratingEntry.empty()) {
                    stringstream ratingSS(ratingEntry);
                    UserRating rating;

                    getline(ratingSS, rating.userId, ',');

                    getline(ratingSS, token, ',');
                    rating.score = stoi(token);

                    getline(ratingSS, rating.comment, ',');
                    getline(ratingSS, rating.date, ',');

                    event.ratings.push_back(rating);
                }
            }
        }

        data.events.push_back(event);
        if (event.eventId >= data.nextEventID) {
            data.nextEventID = event.eventId + 1;
        }
    }
    inFile.close();
}

void loadDiscountedEvents(SystemData& data) {
    ifstream discountFile(DISCOUNT_FILE);
    if (discountFile.is_open()) {
        string line;
        while (getline(discountFile, line)) {
            size_t pos = line.find("|");
            if (pos != string::npos) {
                string eventIdStr = line.substr(0, pos);
                double discount = stod(line.substr(pos + 1));

                int eventId = stoi(eventIdStr);

                // Update events with discounts
                for (auto& event : data.events) {
                    if (event.eventId == eventId) {
                        event.discount = discount;
                        event.discountedPrice = event.originalPrice - discount;
                        break;
                    }
                }
            }
        }
        discountFile.close();
    }
}

void loadRegistrations(SystemData& data) {
    ifstream file(REGISTRATION_FILE);
    data.registrations.clear();
    data.nextRegistrationID = 1;

    if (!file.is_open()) {
        return;
    }

    string line;
    while (getline(file, line)) {
        if (line.empty()) continue;
        
        Registration reg;
        stringstream ss(line);
        string token;

        getline(ss, token, '|');
        reg.registrationID = stoi(token);
        
        getline(ss, reg.participantName, '|');
        getline(ss, reg.participantID, '|');
        getline(ss, reg.companyName, '|');
        getline(ss, reg.phoneNumber, '|');

        getline(ss, token, '|');
        reg.eventID = stoi(token);

        getline(ss, reg.eventName, '|');
        getline(ss, reg.eventDate, '|');
        getline(ss, reg.eventVenue, '|');
        getline(ss, reg.registrationDate, '|');

        getline(ss, token, '|');
        reg.amountPaid = stod(token);

        data.registrations.push_back(reg);

        if (reg.registrationID >= data.nextRegistrationID) {
            data.nextRegistrationID = reg.registrationID + 1;
        }
    }
    file.close();
}

void saveRegistrations(SystemData& data) {
    ofstream file(REGISTRATION_FILE);

    if (!file.is_open()) {
        cout << "Error: Unable to save registrations to file!\n";
        return;
    }

    for (const auto& reg : data.registrations) {
        file << reg.registrationID << "|"
            << reg.participantName << "|"
            << reg.participantID << "|"
            << reg.companyName << "|"
            << reg.phoneNumber << "|"
            << reg.eventID << "|"
            << reg.eventName << "|"
            << reg.eventDate << "|"
            << reg.eventVenue << "|"
            << reg.registrationDate << "|"
            << reg.amountPaid << "\n";
    }
    file.close();
}

void loadFeedbacks(SystemData& data) {
    ifstream file(FEEDBACK_FILE);
    data.feedbacks.clear();

    if (!file.is_open()) {
        return;
    }

    string line;
    while (getline(file, line)) {
        if (line.empty()) continue;

        Feedback feedback;
        stringstream ss(line);
        string token;

        getline(ss, feedback.eventId, '|');
        getline(ss, feedback.userId, '|');
        getline(ss, feedback.feedbackText, '|');

        getline(ss, token, '|');
        feedback.rating = stoi(token);

        getline(ss, feedback.date, '|');

        data.feedbacks.push_back(feedback);
    }
    file.close();
}

void saveFeedbacks(const SystemData& data) {
    ofstream file(FEEDBACK_FILE);

    if (!file.is_open()) {
        cout << "Error: Unable to save feedback to file!\n";
        return;
    }

    for (const auto& feedback : data.feedbacks) {
        file << feedback.eventId << "|"
            << feedback.userId << "|"
            << feedback.feedbackText << "|"
            << feedback.rating << "|"
            << feedback.date << "\n";
    }
    file.close();
}

bool isValidInput(const string& input) {
    return !input.empty() && input.find_first_not_of(" \t\n\r") != string::npos;
}

bool validatePhoneNum(const string& phoneNum) {
    return phoneNum.length() == PHONE_NUM_LENGTH && all_of(phoneNum.begin(), phoneNum.end(), ::isdigit);
}

bool validatePassword(const string& password) {
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

bool isValidPhoneNum(const string& phoneNum) {
    return phoneNum.length() == PHONE_NUM_LENGTH && all_of(phoneNum.begin(), phoneNum.end(), ::isdigit);
}

string getCurrentDate() {
    time_t now = time(0);
    tm ltm;
    localtime_s(&ltm, &now);
    stringstream ss;
    ss << setw(2) << setfill('0') << ltm.tm_mday << "/" << setw(2) << setfill('0') << (1 + ltm.tm_mon) << "/" << (1900 + ltm.tm_year);
    return ss.str();
}

string generateParticipantID() {
    static int counter = 1000;
    return "P" + to_string(counter++);
}

string generateUserID() {
    static int userCounter = 1000;
    static int orgCounter = 100;

    // Randomly decide if this is a regular user or organizer (1 in 5 chance for organizer)
    if (rand() % 5 == 0) {
        return "OR" + to_string(orgCounter++);
    }
    else {
        return "US" + to_string(userCounter++);
    }
}

vector<Registration> getUserRegistrations(const string& participantID, const SystemData& data) {
    vector<Registration> userRegs;
    for (const auto& reg : data.registrations) {
        if (reg.participantID == participantID) {
            userRegs.push_back(reg);
        }
    }
    return userRegs;
}

vector<Registration> getUserRegistrationsByName(const string& participantName, const SystemData& data) {
    vector<Registration> userRegs;
    for (const auto& reg : data.registrations) {
        if (reg.participantName == participantName) {
            userRegs.push_back(reg);
        }
    }
    return userRegs;
}

bool isEventFull(const Event& event) {
    return event.currentParticipants >= event.maxParticipants;
}

void clearScreen() {
#ifdef _WIN32
    system("cls");
#else
    system("clear");
#endif
}
