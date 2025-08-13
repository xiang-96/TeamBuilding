struct EventDetails{
  string name;
  string date;
  string venue;
  vector<string> equipment;
  vector<vector><string> attendanceMatrix;
};

struct Participant{
  int participantID;
  string name;
  int eventID;
  bool attendanceMarked;
  bool present;
};

struct AttendanceRecord{
  Participant participant;
  string status;
};
