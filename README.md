# Disaster-management-project
Fast reach out who will need the help most
#include <iostream>
#include <cstring>
using namespace std;

#define MAX 100

// ================= STRUCT =================
struct Request {
    int id;
    char name[50];
    char area[50];
    char need[50];
    int priority; // 1 = emergency, 0 = normal
};

// ================= QUEUE =================
struct Queue {
    Request data[MAX];
    int front, rear;

    Queue() {
        front = -1;
        rear = -1;
    }

    bool isEmpty() {
        return front == -1;
    }

    void enqueue(Request r) {
        if (rear == MAX - 1) {
            cout << "Queue Full!\n";
            return;
        }
        if (front == -1) front = 0;
        data[++rear] = r;
    }

    Request dequeue() {
        Request temp;
        if (isEmpty()) {
            cout << "Queue Empty!\n";
            temp.id = -1;
            return temp;
        }

        temp = data[front];

        if (front == rear)
            front = rear = -1;
        else
            front++;

        return temp;
    }
};

// ================= STACK =================
struct Stack {
    Request data[MAX];
    int top;

    Stack() {
        top = -1;
    }

    bool isEmpty() {
        return top == -1;
    }

    void push(Request r) {
        if (top == MAX - 1) {
            cout << "Stack Full!\n";
            return;
        }
        data[++top] = r;
    }

    Request pop() {
        Request temp;
        if (isEmpty()) {
            cout << "Stack Empty!\n";
            temp.id = -1;
            return temp;
        }
        return data[top--];
    }
};

// ================= HASH TABLE =================
Request hashTable[MAX];
bool occupied[MAX] = {false};

// ================= GLOBAL =================
Queue emergencyQ, normalQ;
Stack undoStack;
int currentID = 1;

// ================= HASH FUNCTION =================
int hashFunc(int id) {
    return id % MAX;
}

// ================= ADD REQUEST =================
void addRequest() {
    Request r;

    r.id = currentID++;

    cout << "Enter Name: ";
    cin >> r.name;

    cout << "Enter Area: ";
    cin >> r.area;

    cout << "Enter Need (Food/Medicine): ";
    cin >> r.need;

    cout << "Priority (1 = Emergency, 0 = Normal): ";
    cin >> r.priority;

    if (r.priority == 1)
        emergencyQ.enqueue(r);
    else
        normalQ.enqueue(r);

    // Hash insert with linear probing
    int index = hashFunc(r.id);
    while (occupied[index]) {
        index = (index + 1) % MAX;
    }

    hashTable[index] = r;
    occupied[index] = true;

    cout << "Request Added! ID: " << r.id << endl;
}

// ================= ADD DONATION =================
void addDonation() {
    char name[50], item[50];
    int qty;

    cout << "Donor Name: ";
    cin >> name;

    cout << "Item: ";
    cin >> item;

    cout << "Quantity: ";
    cin >> qty;

    cout << "Donation Recorded Successfully!\n";
}

// ================= DISTRIBUTE =================
void distribute() {
    Request r;

    if (!emergencyQ.isEmpty()) {
        r = emergencyQ.dequeue();
    }
    else if (!normalQ.isEmpty()) {
        r = normalQ.dequeue();
    }
    else {
        cout << "No Requests!\n";
        return;
    }

    cout << "Distributed to: " << r.name << " (ID: " << r.id << ")\n";

    undoStack.push(r);
}

// ================= SEARCH =================
void searchRequest() {
    int id;
    cout << "Enter ID: ";
    cin >> id;

    int index = hashFunc(id);

    for (int i = 0; i < MAX; i++) {
        int pos = (index + i) % MAX;

        if (occupied[pos] && hashTable[pos].id == id) {
            Request r = hashTable[pos];
            cout << "Found: " << r.name << ", " << r.area << ", " << r.need << endl;
            return;
        }
    }

    cout << "Not Found!\n";
}

// ================= UNDO =================
void undo() {
    if (undoStack.isEmpty()) {
        cout << "Nothing to undo!\n";
        return;
    }

    Request r = undoStack.pop();

    if (r.priority == 1)
        emergencyQ.enqueue(r);
    else
        normalQ.enqueue(r);

    cout << "Undo Successful! Restored ID: " << r.id << endl;
}

// ================= DISPLAY QUEUE =================
void displayQueue() {
    cout << "\n--- Emergency Queue ---\n";
    if (!emergencyQ.isEmpty()) {
        for (int i = emergencyQ.front; i <= emergencyQ.rear; i++) {
            cout << emergencyQ.data[i].id << " - " << emergencyQ.data[i].name << endl;
        }
    } else {
        cout << "Empty\n";
    }

    cout << "\n--- Normal Queue ---\n";
    if (!normalQ.isEmpty()) {
        for (int i = normalQ.front; i <= normalQ.rear; i++) {
            cout << normalQ.data[i].id << " - " << normalQ.data[i].name << endl;
        }
    } else {
        cout << "Empty\n";
    }
}

// ================= MAIN =================
int main() {
    int choice;

    while (true) {
        cout << "\n===== Disaster Relief System =====\n";
        cout << "1. Add Request\n";
        cout << "2. Add Donation\n";
        cout << "3. Distribute Relief\n";
        cout << "4. Search Request\n";
        cout << "5. Undo Last Action\n";
        cout << "6. Display Queue\n";
        cout << "7. Exit\n";

        cout << "Enter choice: ";
        cin >> choice;

        switch (choice) {
            case 1: addRequest(); break;
            case 2: addDonation(); break;
            case 3: distribute(); break;
            case 4: searchRequest(); break;
            case 5: undo(); break;
            case 6: displayQueue(); break;
            case 7: return 0;
            default: cout << "Invalid choice!\n";
        }
    }
}
