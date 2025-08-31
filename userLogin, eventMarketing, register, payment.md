#ifndef COMMON_H
#define COMMON_H

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
#include <random>
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

#endif

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
    double pricePerHour;
    double durationHours;
    double totalPrice;
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
    bool isOrganizer;
    vector<string> registeredEvents;
    vector<string> interests;
};

struct Registration {
    int registrationId;
    string participantName;
    string participantId;
    string companyName;
    string phoneNumber;
    int eventId;
    string eventName;
    string eventDate;
    string eventLocation;
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

// Payment struct to store payment records
struct Payment {
    int userId = 0;
    int registrationId = 0;  // Link to registration instead of bookingID
    string paymentMethod = "";
    double paymentAmount = 0.0;
    string paymentDate = "";
    string receiptId = "";
    string eventName = "";
    string participantName = "";
    string participantId = "";
    string cardNumber = "";      // For credit/debit card (masked)
    string walletId = "";        // For TnG e-wallet
    string eventDate = "";
    string eventLocation = "";
};

// Function declarations
int registrationMain() {

}
int paymentMain();
void processImmediatePayment(const Registration& registration);

struct SystemData {
    vector<User> users;
    vector<Event> events;
    vector<Registration> registrations;
    vector<Feedback> feedbacks;
    int nextRegistrationId;
    int nextEventId;

    SystemData() : nextRegistrationId(1), nextEventId(1) {}
};

void initializeSystem(SystemData& data);
void addSampleData(SystemData& data);
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
void saveRegistrations(const SystemData& data);
void loadFeedbacks(SystemData& data);
void saveFeedbacks(const SystemData& data);
void displayEvent(const Event& event);
bool isEventFull(const Event& event);
void createEvent(SystemData& data);
void viewEvents(const SystemData& data);
void viewFeedback(const SystemData& data);
void viewMyRegistrations(const SystemData& data, const string& userId);
bool isValidInput(const string& input);
bool validatePhoneNum(const string& phoneNum);
bool validatePassword(const string& password);
bool validateDate(const string& date);
string getCurrentDate();
string generateParticipantId();
string generateUserId();
vector<Registration> getUserRegistrations(const string& participantId, const SystemData& data);
vector<Registration> getUserRegistrationsByName(const string& participantName, const SystemData& data);
vector<Registration> loadRegistrationsFromFile();
vector<Event> loadEventsFromFile();
vector<Registration> getRegistrationsByParticipantName(const string& participantName);
void clearScreen();

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
    event1.eventId = data.nextEventId++;
    event1.eventName = "Team Building Workshop";
    event1.eventDate = "15/09/2025";
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
    event2.eventId = data.nextEventId++;
    event2.eventName = "Leadership Training";
    event2.eventDate = "20/09/2025";
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
    event3.eventId = data.nextEventId++;
    event3.eventName = "Communication Skills";
    event3.eventDate = "25/09/2025";
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

    // Add sample organizer accounts (pre-registered)
    User organizer1;
    organizer1.userId = "ORG1001";
    organizer1.userName = "pingting";
    organizer1.phoneNum = "0111111111";
    organizer1.companyName = "Event Pro Management";
    organizer1.password = "pro123";
    organizer1.isOrganizer = true;

    User organizer2;
    organizer2.userId = "ORG1002";
    organizer2.userName = "chiewchin";
    organizer2.phoneNum = "0222222222";
    organizer2.companyName = "Team Builders Incorporated";
    organizer2.password = "team456";
    organizer2.isOrganizer = true;

    User organizer3;
    organizer3.userId = "ORG1003";
    organizer3.userName = "yongxiang";
    organizer3.phoneNum = "0333333333";
    organizer3.companyName = "Corporate Events Limited";
    organizer3.password = "corp789";
    organizer3.isOrganizer = true;

    User organizer4;
    organizer4.userId = "ORG1004";
    organizer4.userName = "Premium Workshops";
    organizer4.phoneNum = "0444444444";
    organizer4.companyName = "Premium Workshops International";
    organizer4.password = "premium123";
    organizer4.isOrganizer = true;

    data.users.push_back(organizer1);
    data.users.push_back(organizer2);
    data.users.push_back(organizer3);
    data.users.push_back(organizer4);

    saveEventData(data);
    saveUserData(data);
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
    cout << "\n\t  Login";

    string identifier, password;
    cout << "\nEnter Phone Number or Organizer ID: ";
    cin >> identifier;

    cout << "Enter Password: ";
    cin >> password;

    string userId;
    string userName;
    bool isOrganizer = false;
    bool found = false;

    for (const auto& user : data.users) {
        if ((user.phoneNum == identifier || user.userId == identifier) && user.password == password) {
            userId = user.userId;
            userName = user.userName;
            isOrganizer = user.isOrganizer;
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

        if (isOrganizer) {
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
    cout << "\n\tCustomer Sign Up";

    newUser.userId = generateUserId();
    newUser.isOrganizer = false; // Only customers can sign up
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
    cout << "Note: Organizer accounts are created by administrators only." << endl;
    system("pause");
}

void forgotPassword(SystemData& data) {
    clearScreen();
    displayHeader("Team Building Event System");
    cout << "\n\tReset Password";
    string identifier;
    cout << "\nEnter your registered phone number or user ID: ";
    cin >> identifier;

    // Find user by phone number
    User *targetUser = nullptr;
    for (auto& user : data.users) {
        if (user.phoneNum == identifier || user.userId == identifier) {
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
        cout << "\nError: Phone number or user ID not found in our system.";
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
                if (rememberedUser->isOrganizer) {
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
    newReg.registrationId = data.nextRegistrationId++;
    newReg.participantName = currentUser.userName;
    newReg.participantId = currentUser.userId;
    newReg.companyName = currentUser.companyName;
    newReg.phoneNumber = currentUser.phoneNum;
    newReg.eventId = selectedEvent->eventId;
    newReg.eventName = selectedEvent->eventName;
    newReg.eventDate = selectedEvent->eventDate;
    newReg.eventLocation = selectedEvent->eventLocation;
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
    cout << "Registration ID: " << newReg.registrationId << endl;
    cout << "Event: " << newReg.eventName << endl;
    cout << "Date: " << newReg.eventDate << endl;
    cout << "Venue: " << newReg.eventLocation << endl;
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
        outFile << user.userId << "|" << user.userName << "|" << user.phoneNum << "|" << user.companyName << "|" << user.password << "|" << (user.rememberMe ? "1" : "0") << "|" << (user.isOrganizer ? "1" : "0") << "|";
            
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
        getline(ss, token, '|'); // isOrganizer
        user.isOrganizer = (token == "1");
        
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
        if (event.eventId >= data.nextEventId) {
            data.nextEventId = event.eventId + 1;
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
    data.nextRegistrationId = 1;

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
        reg.registrationId = stoi(token);
        getline(ss, reg.participantName, '|');
        getline(ss, reg.participantId, '|');
        getline(ss, reg.companyName, '|');
        getline(ss, reg.phoneNumber, '|');
        getline(ss, token, '|');
        reg.eventId = stoi(token);
        getline(ss, reg.eventName, '|');
        getline(ss, reg.eventDate, '|');
        getline(ss, reg.eventLocation, '|');
        getline(ss, reg.registrationDate, '|');
        getline(ss, token, '|');
        reg.amountPaid = stod(token);

        data.registrations.push_back(reg);

        if (reg.registrationId >= data.nextRegistrationId) {
            data.nextRegistrationId = reg.registrationId + 1;
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
        file << reg.registrationId << "|"
            << reg.participantName << "|"
            << reg.participantId << "|"
            << reg.companyName << "|"
            << reg.phoneNumber << "|"
            << reg.eventId << "|"
            << reg.eventName << "|"
            << reg.eventDate << "|"
            << reg.eventLocation << "|"
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

string getCurrentDate() {
    time_t now = time(0);
    tm ltm;
    localtime_s(&ltm, &now);
    stringstream ss;
    ss << setw(2) << setfill('0') << ltm.tm_mday << "/" << setw(2) << setfill('0') << (1 + ltm.tm_mon) << "/" << (1900 + ltm.tm_year);
    return ss.str();
}

string generateParticipantId() {
    static int counter = 1000;
    return "P" + to_string(counter++);
}

string generateUserId() {
    static int userCounter = 1000;
    return "US" + to_string(userCounter++);
}

vector<Registration> getUserRegistrations(const string& participantId, const SystemData& data) {
    vector<Registration> userRegs;
    for (const auto& reg : data.registrations) {
        if (reg.participantId == participantId) {
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

// Utility functions
vector<Registration> loadRegistrationsFromFile() {
    vector<Registration> registrations;
    ifstream file("registrations.txt");

    if (!file.is_open()) {
        return registrations;
}

    string line;
    while (getline(file, line)) {
        if (line.empty()) continue;

        stringstream ss(line);
        Registration reg;
        string token;

        getline(ss, token, '|');
        reg.registrationId = stoi(token);
        getline(ss, reg.participantName, '|');
        getline(ss, reg.participantId, '|');
        getline(ss, reg.companyName, '|');
        getline(ss, reg.phoneNumber, '|');
        getline(ss, token, '|');
        reg.eventId = stoi(token);
        getline(ss, reg.eventName, '|');
        getline(ss, reg.eventDate, '|');
        getline(ss, reg.eventLocation);

        registrations.push_back(reg);
    }
    file.close();
    return registrations;
}

vector<Event> loadEventsFromFile() {
    vector<Event> events;
    ifstream file("events.txt");

    if (!file.is_open()) {
        Event event1;
        event1.eventId = 1;
        event1.eventName = "Team Building Workshop";
        event1.eventDate = "15/09/2025";
        event1.eventLocation = "Conference Room A";
        event1.requiredEquipment = "Projector, Whiteboard";
        event1.pricePerHour = 75.0;
        event1.durationHours = 4;
        event1.totalPrice = 300.0;

        Event event2;
        event2.eventId = 2;
        event2.eventName = "Leadership Training";
        event2.eventDate = "20/09/2025";
        event2.eventLocation = "Training Hall B";
        event2.requiredEquipment = "Microphone, Laptop";
        event2.pricePerHour = 100.0;
        event2.durationHours = 6;
        event2.totalPrice = 600.0;

        Event event3; 
        event3.eventId = 3; 
        event3.eventName = "Communication Skills"; 
        event3.eventDate = "25/09/2025";
        event3.eventLocation = "Seminar Room C"; 
        event3.requiredEquipment = "Flipchart, Markers";
        event3.pricePerHour = 60.0; 
        event3.durationHours = 3; 
        event3.totalPrice = 180.0;

        Event event4; 
        event4.eventId = 4; 
        event4.eventName = "Problem Solving Session"; 
        event4.eventDate = "01/10/2025";
        event4.eventLocation = "Workshop Room D"; 
        event4.requiredEquipment = "Sticky Notes, Timer";
        event4.pricePerHour = 80.0; 
        event4.durationHours = 4; 
        event4.totalPrice = 320.0;

        Event event5; 
        event5.eventId = 5; 
        event5.eventName = "Networking Event"; 
        event5.eventDate = "05/10/2025";
        event5.eventLocation = "Main Hall"; 
        event5.requiredEquipment = "Sound System, Refreshments";
        event5.pricePerHour = 50.0; 
        event5.durationHours = 2; 
        event5.totalPrice = 100.0;
        
        events.push_back(event1);
        events.push_back(event2);
        events.push_back(event3);
        events.push_back(event4);
        events.push_back(event5);
        return events;
    }

    string line;
    while (getline(file, line)) {
        if (line.empty()) continue;

        stringstream ss(line);
        Event event;
        string token;

        getline(ss, token, '|');
        event.eventId = stoi(token);
        getline(ss, event.eventName, '|');
        getline(ss, event.eventDate, '|');
        getline(ss, event.eventLocation, '|');
        getline(ss, event.requiredEquipment);

        if (event.eventName.find("Leadership") != string::npos) {
            event.pricePerHour = 100.0;
            event.durationHours = 6;
        }
        else if (event.eventName.find("Team Building") != string::npos) {
            event.pricePerHour = 75.0;
            event.durationHours = 4;
        }
        else if (event.eventName.find("Communication") != string::npos) {
            event.pricePerHour = 60.0;
            event.durationHours = 3;
        }
        else if (event.eventName.find("Problem Solving") != string::npos) {
            event.pricePerHour = 80.0;
            event.durationHours = 4;
        }
        else if (event.eventName.find("Networking") != string::npos) {
            event.pricePerHour = 50.0;
            event.durationHours = 2;
        }
        else {
            event.pricePerHour = 70.0;
            event.durationHours = 4;
        }

        event.totalPrice = event.pricePerHour * event.durationHours;
        events.push_back(event);
    }
    file.close();
    return events;
}

vector<Registration> getRegistrationsByParticipantName(const string& participantName) {
    vector<Registration> allRegistrations = loadRegistrationsFromFile();
    vector<Registration> userRegistrations;

    for (const auto& reg : allRegistrations) {
        if (reg.participantName == participantName) {
            userRegistrations.push_back(reg);
    }
}

    return userRegistrations;
}

Event getEventById(int eventId, const vector<Event>& events) {
    for (const auto& event : events) {
        if (event.eventId == eventId) {
            return event;
        }
        }
    // Return a properly initialized empty event to avoid access violations
    Event emptyEvent;
    emptyEvent.eventId = 0;
    emptyEvent.eventName = "";
    emptyEvent.eventDate = "";
    emptyEvent.eventLocation = "";
    emptyEvent.requiredEquipment = "";
    emptyEvent.pricePerHour = 0.0;
    emptyEvent.durationHours = 0;
    emptyEvent.totalPrice = 0.0;
    return emptyEvent;
    }

class IntegratedPaymentSystem {
private:
    vector<Payment> payments;
    vector<Registration> registrations;
    vector<Event> events;
    static int nextPaymentId;

    // Helper functions
    string generateReceiptId();
    string getCurrentDateTime();
    string maskCardNumber(const string& cardNumber);
    bool validateCardNumber(const string& cardNumber);
    bool validateTnGWallet(const string& walletId);
    void savePaymentToFile(const Payment& payment);

public:
    // Constructor
    IntegratedPaymentSystem();

    // Main payment functions
    void displayUserRegistrations(const string& participantName);
    Registration selectRegistrationForPayment(const string& participantName);
    void displayPaymentSummary(const Registration& registration, const Event& event);
    int paymentMethod();
    bool processPayment(const Registration& registration, const Event& event, int method);
    string generateReceipt(const Payment& payment);

    // File operations
    void loadPayments();
    void savePayments();
    void loadData();

    // Validation functions
    bool validateCreditCard(const string& cardNumber, const string& expiryDate, const string& cvv);
    bool validateTnGEWallet(const string& walletId, const string& pin);

    // Display functions
    void displayPaymentHistory(const string& participantName = "");
    void displayAllPayments();

    // Utility functions
    Payment createPaymentRecord(const Registration& registration, const Event& event, int method, const string& paymentDetails);
    bool isValidAmount(double amount);
    bool hasUnpaidRegistrations(const string& participantName);
    bool isRegistrationPaid(int registrationId);
};

// Initialize static member
int IntegratedPaymentSystem::nextPaymentId = 1;

// Constructor
IntegratedPaymentSystem::IntegratedPaymentSystem() {
    loadData();
    loadPayments();
}

// Load all data from files
void IntegratedPaymentSystem::loadData() {
    registrations = loadRegistrationsFromFile();
    events = loadEventsFromFile();
}

// Display user's registrations
void IntegratedPaymentSystem::displayUserRegistrations(const string& participantName) {
    vector<Registration> userRegs = getRegistrationsByParticipantName(participantName);

    if (userRegs.empty()) {
        cout << "No registrations found for: " << participantName << "\n";
        return;
    }
    
    displayHeader("Team Building Event System");
    cout << "\n\tYOUR REGISTERED EVENTS\n";
    cout << left << setw(5) << "No." << setw(15) << "Reg ID"
        << setw(25) << "Event Name" << setw(15) << "Date"
        << setw(20) << "Venue" << setw(12) << "Status" << "\n";
    cout << string(80, '-') << "\n";

    for (size_t i = 0; i < userRegs.size(); i++) {
        string status = isRegistrationPaid(userRegs[i].registrationId) ? "PAID" : "UNPAID";

        cout << left << setw(5) << (i + 1)
            << setw(15) << userRegs[i].registrationId
            << setw(25) << userRegs[i].eventName
            << setw(15) << userRegs[i].eventDate
            << setw(20) << userRegs[i].eventLocation
            << setw(12) << status << "\n";
    }
    cout << string(80, '=') << "\n";
}

// Select registration for payment
Registration IntegratedPaymentSystem::selectRegistrationForPayment(const string& participantName) {
    vector<Registration> userRegs = getRegistrationsByParticipantName(participantName);
    vector<Registration> unpaidRegs;

    // Filter unpaid registrations
    for (const auto& reg : userRegs) {
        if (!isRegistrationPaid(reg.registrationId)) {
            unpaidRegs.push_back(reg);
        }
    }

    if (unpaidRegs.empty()) {
        cout << "All your registrations are already paid!\n";
        return Registration{ 0, "", "", "", "", 0, "", "", "" , "", 0.0 }; // Return empty registration
    }

    cout << "\nSelect registration to pay for:\n";
    for (size_t i = 0; i < unpaidRegs.size(); i++) {
        Event event = getEventById(unpaidRegs[i].eventId, events);
        cout << (i + 1) << ". " << event.eventName
            << " - RM " << fixed << setprecision(2) << event.totalPrice << "\n";
    }

    int choice;
    do {
        cout << "Enter choice (1-" << unpaidRegs.size() << "): ";
        cin >> choice;
        cin.ignore(numeric_limits<streamsize>::max(), '\n');

        if (choice < 1 || choice > static_cast<int>(unpaidRegs.size())) {
            cout << "Invalid choice! Please try again.\n";
        }
    } while (choice < 1 || choice > static_cast<int>(unpaidRegs.size()));

    return unpaidRegs[choice - 1];
}

// Display payment summary
void IntegratedPaymentSystem::displayPaymentSummary(const Registration& registration, const Event& event) {
    displayHeader("Team Building Event System");
    cout << "\n\tPAYMENT SUMMARY\n";
    cout << "Registration ID: " << registration.registrationId << "\n";
    cout << "Participant: " << registration.participantName << "\n";
    cout << "Participant ID: " << registration.participantId << "\n";
    cout << "Event: " << event.eventName << "\n";
    cout << "Date: " << event.eventDate << "\n";
    cout << "Venue: " << event.eventLocation << "\n";
    cout << "Duration: " << event.durationHours << " hours\n";
    cout << string(60, '-') << "\n";
    cout << "Rate per Hour: RM " << fixed << setprecision(2) << event.pricePerHour << "\n";
    cout << "Total Duration: " << event.durationHours << " hours\n";
    cout << string(60, '-') << "\n";
    cout << "TOTAL AMOUNT: RM " << fixed << setprecision(2) << event.totalPrice << "\n";
    cout << string(60, '=') << "\n";
}

// Payment method selection
int IntegratedPaymentSystem::paymentMethod() {
    int choice;

    displayHeader("Team Building Event System");
    cout << "\n\tPAYMENT METHODS\n";
    cout << "1. Credit/Debit Card\n";
    cout << "2. TnG E-Wallet\n";
    cout << "3. Cancel Payment\n";

    do {
        cout << "\nSelect payment method (1-3): ";
        cin >> choice;
        cin.ignore();

        if (choice < 1 || choice > 3) {
            cout << "Invalid choice! Please select 1, 2, or 3.\n";
        }
    } while (choice < 1 || choice > 3);

    return choice;
}

// Process payment based on selected method
bool IntegratedPaymentSystem::processPayment(const Registration& registration, const Event& event, int method) {
    string paymentDetails = "";

    switch (method) {
    case 1: { // Credit/Debit Card
        cout << "\n=== CREDIT/DEBIT CARD PAYMENT ===\n";

        string cardNumber, expiryDate, cvv, cardholderName;

        // Get card details with validation
        do {
            cout << "Enter Card Number (16 digits): ";
            getline(cin, cardNumber);

            if (!validateCardNumber(cardNumber)) {
                cout << "Error: Card number must be exactly 16 digits!\n";
            }
        } while (!validateCardNumber(cardNumber));

        cout << "Enter Cardholder Name: ";
        getline(cin, cardholderName);

        do {
            cout << "Enter Expiry Date (MM/YY): ";
            getline(cin, expiryDate);

            if (expiryDate.length() != 5 || expiryDate[2] != '/') {
                cout << "Error: Please enter date in MM/YY format!\n";
                continue;
            }
            break;
        } while (true);

        do {
            cout << "Enter CVV (3 digits): ";
            getline(cin, cvv);

            if (cvv.length() != 3 || !all_of(cvv.begin(), cvv.end(), ::isdigit)) {
                cout << "Error: CVV must be exactly 3 digits!\n";
            }
        } while (cvv.length() != 3 || !all_of(cvv.begin(), cvv.end(), ::isdigit));

        if (validateCreditCard(cardNumber, expiryDate, cvv)) {
            paymentDetails = maskCardNumber(cardNumber);
            cout << "\nProcessing payment...\n";
            cout << "Payment successful!\n";

            // Create and save payment record
            Payment payment = createPaymentRecord(registration, event, method, paymentDetails);
            payments.push_back(payment);
            savePaymentToFile(payment);

            // Display receipt
            cout << generateReceipt(payment);
            return true;
        }
        else {
            cout << "Payment failed! Invalid card details.\n";
            return false;
        }
    }

    case 2: { // TnG E-Wallet
        cout << "\n=== TNG E-WALLET PAYMENT ===\n";

        string walletId, pin;

        do {
            cout << "Enter TnG Wallet ID (10 digits): ";
            getline(cin, walletId);

            if (!validateTnGWallet(walletId)) {
                cout << "Error: Wallet ID must be exactly 10 digits!\n";
            }
        } while (!validateTnGWallet(walletId));

        do {
            cout << "Enter 6-digit PIN: ";
            getline(cin, pin);

            if (pin.length() != 6 || !all_of(pin.begin(), pin.end(), ::isdigit)) {
                cout << "Error: PIN must be exactly 6 digits!\n";
            }
        } while (pin.length() != 6 || !all_of(pin.begin(), pin.end(), ::isdigit));

        if (validateTnGEWallet(walletId, pin)) {
            paymentDetails = "TnG-" + walletId;
            cout << "\nProcessing payment...\n";
            cout << "Payment successful!\n";

            // Create and save payment record
            Payment payment = createPaymentRecord(registration, event, method, paymentDetails);
            payments.push_back(payment);
            savePaymentToFile(payment);

            // Display receipt
            cout << generateReceipt(payment);
            return true;
        }
        else {
            cout << "Payment failed! Invalid wallet credentials.\n";
            return false;
        }
    }

    case 3: // Cancel
        cout << "Payment cancelled.\n";
        return false;

    default:
        cout << "Invalid payment method!\n";
        return false;
    }
}

// Generate and display receipt
string IntegratedPaymentSystem::generateReceipt(const Payment& payment) {
    stringstream receiptStream;

    receiptStream << "\n" << string(50, '=') << "\n";
    receiptStream << "              PAYMENT RECEIPT\n";
    receiptStream << string(50, '=') << "\n";
    receiptStream << "Receipt ID: " << payment.receiptId << "\n";
    receiptStream << "Date/Time: " << payment.paymentDate << "\n";
    receiptStream << string(50, '-') << "\n";
    receiptStream << "Participant: " << payment.participantName << "\n";
    receiptStream << "Participant ID: " << payment.participantId << "\n";
    receiptStream << "Registration ID: " << payment.registrationId << "\n";
    receiptStream << "Event: " << payment.eventName << "\n";
    receiptStream << "Event Date: " << payment.eventDate << "\n";
    receiptStream << "Event Venue: " << payment.eventLocation << "\n";
    receiptStream << "Payment Method: " << payment.paymentMethod << "\n";

    if (payment.paymentMethod == "Credit/Debit Card") {
        receiptStream << "Card: " << payment.cardNumber << "\n";
    }
    else if (payment.paymentMethod == "TnG E-Wallet") {
        receiptStream << "Wallet: " << payment.walletId << "\n";
    }

    receiptStream << string(50, '-') << "\n";
    receiptStream << "Amount Paid: RM " << fixed << setprecision(2) << payment.paymentAmount << "\n";
    receiptStream << string(50, '=') << "\n";
    receiptStream << "Thank you for your payment!\n";
    receiptStream << "Keep this receipt for your records.\n";
    receiptStream << string(50, '=') << "\n";

    return receiptStream.str();
}

// Load payments from file
void IntegratedPaymentSystem::loadPayments() {
    ifstream file("payments.txt");
    payments.clear();
    nextPaymentId = 1;

    if (!file.is_open()) {
        return; // File doesn't exist yet
    }

    string line;
    while (getline(file, line)) {
        if (line.empty()) continue;

        stringstream ss(line);
        Payment payment;
        string token;

        getline(ss, token, '|');
        payment.userId = stoi(token);
        getline(ss, token, '|');
        payment.registrationId = stoi(token);
        getline(ss, payment.paymentMethod, '|');
        getline(ss, token, '|');
        payment.paymentAmount = stod(token);
        getline(ss, payment.paymentDate, '|');
        getline(ss, payment.receiptId, '|');
        getline(ss, payment.eventName, '|');
        getline(ss, payment.participantName, '|');
        getline(ss, payment.participantId, '|');
        getline(ss, payment.eventDate, '|');
        getline(ss, payment.eventLocation, '|');
        getline(ss, payment.cardNumber, '|');
        getline(ss, payment.walletId);

        payments.push_back(payment);

        if (payment.userId >= nextPaymentId) {
            nextPaymentId = payment.userId + 1;
        }
    }
    file.close();
}

// Save all payments to file
void IntegratedPaymentSystem::savePayments() {
    ofstream file("payments.txt");

    if (!file.is_open()) {
        cout << "Error: Unable to save payments to file!\n";
        return;
    }

    for (const auto& payment : payments) {
        file << payment.userId << "|"
            << payment.registrationId << "|"
            << payment.paymentMethod << "|"
            << payment.paymentAmount << "|"
            << payment.paymentDate << "|"
            << payment.receiptId << "|"
            << payment.eventName << "|"
            << payment.participantName << "|"
            << payment.participantId << "|"
            << payment.eventDate << "|"
            << payment.eventLocation << "|"
            << payment.cardNumber << "|"
            << payment.walletId << "\n";
    }
    file.close();
}

// Save single payment to file (append)
void IntegratedPaymentSystem::savePaymentToFile(const Payment& payment) {
    ofstream file("payments.txt", ios::app);

    if (!file.is_open()) {
        cout << "Error: Unable to save payment to file!\n";
        return;
    }

    file << payment.userId << "|"
        << payment.registrationId << "|"
        << payment.paymentMethod << "|"
        << payment.paymentAmount << "|"
        << payment.paymentDate << "|"
        << payment.receiptId << "|"
        << payment.eventName << "|"
        << payment.participantName << "|"
        << payment.participantId << "|"
        << payment.eventDate << "|"
        << payment.eventLocation << "|"
        << payment.cardNumber << "|"
        << payment.walletId << "\n";

    file.close();
}

// Validate credit card number (16 digits)
bool IntegratedPaymentSystem::validateCardNumber(const string& cardNumber) {
    if (cardNumber.length() != 16) {
        return false;
    }

    for (char c : cardNumber) {
        if (!isdigit(c)) {
            return false;
        }
    }
    return true;
}

// Validate TnG wallet ID (10 digits)
bool IntegratedPaymentSystem::validateTnGWallet(const string& walletId) {
    if (walletId.length() != 10) {
        return false;
    }

    for (char c : walletId) {
        if (!isdigit(c)) {
            return false;
        }
    }
    return true;
}

// Validate credit card details
bool IntegratedPaymentSystem::validateCreditCard(const string& cardNumber, const string& expiryDate, const string& cvv) {
    // Basic validation - in real system would check with payment gateway
    return validateCardNumber(cardNumber) &&
        expiryDate.length() == 5 &&
        cvv.length() == 3;
}

// Validate TnG e-wallet credentials
bool IntegratedPaymentSystem::validateTnGEWallet(const string& walletId, const string& pin) {
    // Basic validation - in real system would check with TnG API
    return validateTnGWallet(walletId) && pin.length() == 6;
}

// Generate unique receipt ID
string IntegratedPaymentSystem::generateReceiptId() {
    static random_device rd;
    static mt19937 gen(rd());
    static uniform_int_distribution<> dis(100000, 999999);

    return "RCP" + to_string(dis(gen));
}

// Get current date and time
string IntegratedPaymentSystem::getCurrentDateTime() {
    time_t now = time(0);
    char dt[26];
    ctime_s(dt, sizeof(dt), &now);
    string dateTime(dt);
    dateTime.pop_back(); // Remove newline character
    return dateTime;
}

// Mask card number for security
string IntegratedPaymentSystem::maskCardNumber(const string& cardNumber) {
    if (cardNumber.length() != 16) {
        return cardNumber;
    }

    return "**** **** **** ****" + cardNumber.substr(12, 4);
}

// Create payment record
Payment IntegratedPaymentSystem::createPaymentRecord(const Registration& registration, const Event& event, int method, const string& paymentDetails) {
    Payment payment;
    payment.userId = nextPaymentId++;
    payment.registrationId = registration.registrationId;
    payment.paymentAmount = event.totalPrice;
    payment.paymentDate = getCurrentDateTime();
    payment.receiptId = generateReceiptId();
    payment.eventName = event.eventName;
    payment.eventDate = event.eventDate;
    payment.eventLocation = event.eventLocation;
    payment.participantName = registration.participantName;
    payment.participantId = registration.participantId;

    if (method == 1) { // Credit/Debit Card
        payment.paymentMethod = "Credit/Debit Card";
        payment.cardNumber = paymentDetails;
    }
    else if (method == 2) { // TnG E-Wallet
        payment.paymentMethod = "TnG E-Wallet";
        payment.cardNumber = "";
        payment.walletId = paymentDetails;
    }
    else {
        payment.paymentMethod = "Unknown";
    }

    return payment;
}

// Check if registration is already paid
bool IntegratedPaymentSystem::isRegistrationPaid(int registrationId) {
    for (const auto& payment : payments) {
        if (payment.registrationId == registrationId) {
            return true;
        }
    }
    return false;
}

// Check if user has unpaid registrations
bool IntegratedPaymentSystem::hasUnpaidRegistrations(const string& participantName) {
    vector<Registration> userRegs = getRegistrationsByParticipantName(participantName);

    for (const auto& reg : userRegs) {
        if (!isRegistrationPaid(reg.registrationId)) {
            return true;
        }
    }
    return false;
}

// Display payment history for specific participant
void IntegratedPaymentSystem::displayPaymentHistory(const string& participantName) {
    vector<Payment> userPayments;

    if (participantName.empty()) {
        userPayments = payments;
    }
    else {
        for (const auto& payment : payments) {
            if (payment.participantName == participantName) {
                userPayments.push_back(payment);
            }
        }
    }

    if (userPayments.empty()) {
        cout << "No payment records found.\n";
        return;
    }

    cout << "\n=== PAYMENT HISTORY ===\n";
    cout << left << setw(8) << "ID" << setw(15) << "Participant"
        << setw(20) << "Event" << setw(15) << "Method"
        << setw(10) << "Amount" << setw(12) << "Receipt"
        << "Date\n";
    cout << string(100, '-') << "\n";

    for (const auto& payment : userPayments) {
        cout << left << setw(8) << payment.userId
            << setw(15) << payment.participantName
            << setw(20) << payment.eventName
            << setw(15) << payment.paymentMethod
            << setw(10) << fixed << setprecision(2) << payment.paymentAmount
            << setw(12) << payment.receiptId
            << payment.paymentDate << "\n";
    }
}

// Display all payments (admin function)
void IntegratedPaymentSystem::displayAllPayments() {
    displayPaymentHistory("");
}

// Validate amount
bool IntegratedPaymentSystem::isValidAmount(double amount) {
    return amount > 0.0;
}

// Integrated payment system main function
int paymentMain() {
    IntegratedPaymentSystem paymentSystem;
    int choice;

    do {
        displayHeader("Team Building Event System");
        cout << "\n\tINTEGRATED PAYMENT SYSTEM\n";
        cout << "1. Make Payment for Registered Event\n";
        cout << "2. View My Payment History\n";
        cout << "3. View All Payments (Admin)\n";
        cout << "4. Return to Main Menu\n";
        cout << "\nEnter choice: ";
        cin >> choice;
        cin.ignore();

        switch (choice) {
        case 1: {
            cout << "\n=== MAKE PAYMENT ===\n";

            string participantName;
            cout << "Enter your name: ";
            getline(cin, participantName);

            // Display user's registrations
            paymentSystem.displayUserRegistrations(participantName);

            // Check if user has unpaid registrations
            if (!paymentSystem.hasUnpaidRegistrations(participantName)) {
                cout << "All your registrations are already paid!\n";
                break;
            }

            // Select registration for payment
            Registration selectedReg = paymentSystem.selectRegistrationForPayment(participantName);

            if (selectedReg.registrationId == 0) {
                break; // No valid registration selected
            }

            // Get event details
            vector<Event> events = loadEventsFromFile();
            Event selectedEvent = getEventById(selectedReg.eventId, events);

            if (selectedEvent.eventId == 0) {
                cout << "Error: Event details not found!\n";
                break;
            }

            // Display payment summary
            paymentSystem.displayPaymentSummary(selectedReg, selectedEvent);

            // Get payment method
            int method = paymentSystem.paymentMethod();

            if (method != 3) {
                bool success = paymentSystem.processPayment(selectedReg, selectedEvent, method);
                if (success) {
                    cout << "\nPayment completed successfully!\n";
                    cout << "Your registration is now fully paid.\n";
                }
                else {
                    cout << "\nPayment failed! Please try again.\n";
                }
            }
            break;
        }
        case 2: {
            string name;
            cout << "Enter your name (or press Enter for all): ";
            getline(cin, name);
            paymentSystem.displayPaymentHistory(name);
            break;
        }
        case 3:
            paymentSystem.displayAllPayments();
            break;
        case 4:
            cout << "Returning to main menu...\n";
            break;
        default:
            cout << "Invalid choice! Please try again.\n";
        }

        if (choice != 4) {
            cout << "\nPress Enter to continue...";
            cin.get();
        }

    } while (choice != 4);

    return 0;
}

// Function for immediate payment after registration
void processImmediatePayment(const Registration& registration) {
    IntegratedPaymentSystem paymentSystem;

    // Get event details
    vector<Event> events = loadEventsFromFile();
    Event selectedEvent = getEventById(registration.eventId, events);

    if (selectedEvent.eventId == 0) {
        cout << "Error: Event details not found!\n";
        return;
    }

    // Display payment summary
    paymentSystem.displayPaymentSummary(registration, selectedEvent);

    // Get payment method
    int method = paymentSystem.paymentMethod();

    if (method != 3) {
        bool success = paymentSystem.processPayment(registration, selectedEvent, method);
        if (success) {
            cout << "\nPayment completed successfully!\n";
            cout << "Your registration is now fully paid.\n";
        }
        else {
            cout << "\nPayment failed! You can try again later through the Payment System menu.\n";
        }
    }
    else {
        cout << "\nPayment cancelled. You can make payment later through the Payment System menu.\n";
        cout << "Remember your Registration ID: " << registration.registrationId << "\n";
    }
}

void clearScreen() {
#ifdef _WIN32
    system("cls");
#else
    system("clear");
#endif
}

int main() {
    SystemData systemData;
    initializeSystem(systemData);
    displayMainMenu(systemData);
    return 0;
}
