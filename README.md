#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_QUEUE 5   // Maximum waiting queue size

// Structure to store vehicle details
typedef struct {
    char vehicle_id[10];
    char vehicle_type[10];  // "car" or "bike"
} Vehicle;

// Structure for parking slot
typedef struct {
    int slot_id;
    char slot_type[10];  // "car" or "bike"
    int is_occupied;
    Vehicle vehicle;
} ParkingSlot;

// Queue for managing waiting vehicles
typedef struct {
    Vehicle queue[MAX_QUEUE];
    int front, rear;
} VehicleQueue;

ParkingSlot *parking_slots; // Pointer to dynamically allocated parking slots
VehicleQueue waiting_queue; // Waiting queue for vehicles
int total_slots;           // Total number of parking slots

// Function prototypes
void initialize_parking_lot();
void park_vehicle(Vehicle vehicle);
void remove_vehicle(char vehicle_id[]);
void display_parking_status();
void display_waiting_queue();
void enqueue(Vehicle vehicle);
Vehicle dequeue();
int is_queue_empty();
int is_queue_full();
void free_parking_slots();

int main() {
    int choice;
    printf("Enter the total number of parking slots: ");
    scanf("%d", &total_slots);

    // Dynamically allocate memory for parking slots based on user input
    parking_slots = (ParkingSlot *)malloc(total_slots * sizeof(ParkingSlot));

    if (parking_slots == NULL) {
        printf("Memory allocation failed!\n");
        return -1;
    }

    initialize_parking_lot();

    while (1) {
        printf("\nParking Lot Management System\n");
        printf("1. Park Vehicle\n");
        printf("2. Remove Vehicle\n");
        printf("3. Display Parking Lot Status\n");
        printf("4. Display Waiting Queue\n");
        printf("5. Exit\n");
        printf("Enter your choice: ");
        scanf("%d", &choice);

        switch (choice) {
            case 1: {
                Vehicle v;
                printf("Enter Vehicle ID: ");
                scanf("%s", v.vehicle_id);
                printf("Enter Vehicle Type (car/bike): ");
                scanf("%s", v.vehicle_type);
                park_vehicle(v);
                break;
            }
            case 2: {
                char vehicle_id[10];
                printf("Enter Vehicle ID to remove: ");
                scanf("%s", vehicle_id);
                remove_vehicle(vehicle_id);
                break;
            }
            case 3:
                display_parking_status();
                break;
            case 4:
                display_waiting_queue();
                break;
            case 5:
                free_parking_slots();
                printf("Exiting program.\n");
                return 0;
            default:
                printf("Invalid choice! Please try again.\n");
        }
    }

    return 0;
}

// Function to initialize parking slots
void initialize_parking_lot() {
    waiting_queue.front = 0;
    waiting_queue.rear = -1;

    for (int i = 0; i < total_slots; i++) {
        parking_slots[i].slot_id = i + 1;
        strcpy(parking_slots[i].slot_type, (i % 2 == 0) ? "car" : "bike");
        parking_slots[i].is_occupied = 0;
    }
    printf("Parking lot initialized with %d slots.\n\n", total_slots);
}

// Function to park a vehicle
void park_vehicle(Vehicle vehicle) {
    for (int i = 0; i < total_slots; i++) {
        if (!parking_slots[i].is_occupied && strcmp(parking_slots[i].slot_type, vehicle.vehicle_type) == 0) {
            parking_slots[i].is_occupied = 1;
            parking_slots[i].vehicle = vehicle;
            printf("Vehicle %s parked in slot %d.\n", vehicle.vehicle_id, parking_slots[i].slot_id);
            return;
        }
    }
    if (!is_queue_full()) {
        printf("No available slot for %s. Added to the queue.\n", vehicle.vehicle_id);
        enqueue(vehicle);
    } else {
        printf("Parking lot and waiting queue are full. Vehicle %s cannot be accommodated.\n", vehicle.vehicle_id);
    }
}

// Function to remove a vehicle from the parking lot
void remove_vehicle(char vehicle_id[]) {
    for (int i = 0; i < total_slots; i++) {
        if (parking_slots[i].is_occupied && strcmp(parking_slots[i].vehicle.vehicle_id, vehicle_id) == 0) {
            printf("Vehicle %s removed from slot %d.\n", vehicle_id, parking_slots[i].slot_id);
            parking_slots[i].is_occupied = 0;

            // If there are vehicles in the queue, assign the slot
            if (!is_queue_empty()) {
                Vehicle next_vehicle = dequeue();
                park_vehicle(next_vehicle);
            }
            return;
        }
    }
    printf("Vehicle %s not found in the parking lot.\n", vehicle_id);
}

// Function to display parking lot status
void display_parking_status() {
    printf("\nParking Lot Status:\n");
    for (int i = 0; i < total_slots; i++) {
        printf("Slot %d (%s): %s\n", parking_slots[i].slot_id, parking_slots[i].slot_type,
               parking_slots[i].is_occupied ? parking_slots[i].vehicle.vehicle_id : "Empty");
    }
}

// Function to display the waiting queue
void display_waiting_queue() {
    if (is_queue_empty()) {
        printf("\nNo vehicles in the waiting queue.\n");
    } else {
        printf("\nWaiting Queue:\n");
        for (int i = waiting_queue.front; i <= waiting_queue.rear; i++) {
            printf("Vehicle ID: %s, Type: %s\n", waiting_queue.queue[i].vehicle_id, waiting_queue.queue[i].vehicle_type);
        }
    }
}

// Queue functions
void enqueue(Vehicle vehicle) {
    if (is_queue_full()) {
        printf("Queue is full! Cannot add vehicle %s to the waiting queue.\n", vehicle.vehicle_id);
        return;
    }
    waiting_queue.queue[++waiting_queue.rear] = vehicle;
}

Vehicle dequeue() {
    if (is_queue_empty()) {
        printf("Queue is empty! No vehicles to dequeue.\n");
        Vehicle empty_vehicle = {"", ""};
        return empty_vehicle;
    }
    return waiting_queue.queue[waiting_queue.front++];
}

int is_queue_empty() {
    return waiting_queue.front > waiting_queue.rear;
}

int is_queue_full() {
    return waiting_queue.rear == MAX_QUEUE - 1;
}

// Free dynamically allocated memory for parking slots
void free_parking_slots() {
    free(parking_slots);
}
