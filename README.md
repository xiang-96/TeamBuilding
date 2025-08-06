#include <iostream>
#include <cstring>
#include <string>
#include <vector>
#include <algorithm>
#include <cmath>
#include <iomanip>

using namespace std;

struct User {
    string nameId;
    string name;
    int phoneNum;
    string companyName;
};

struct Event {
    string eventName;
    string eventDate;
    string eventTime;
    string eventLocation;
    vector<User> participants;
};

int main() {
    
}
