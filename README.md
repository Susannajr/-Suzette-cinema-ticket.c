#include <stdio.h>
#include <stdbool.h>

#define DAYS 7
#define FILMS 5
#define HALLS 5
#define ROWS 5
#define COLS 10
#define SEATS 50

typedef struct { // Structure defining a Film with a title and duration
    char title[50];
    int duration;
} Film;
 
typedef struct { // Structure defining the schedule with day, hour, and movie index
    int day;
    int hour;
    int filmIndex;
} Schedule;

typedef struct { // Structure defining the seat with a booked status
    bool booked;
} Seat;

Film films[FILMS] = {
    {"The school for good and evil", 120}, //Array of movies with the names and durations
    {"Alise in the wonderland", 105},
    {"The shape of water", 90},
    {"Nowhere", 110},
    {"Meg 2", 95}
};

Schedule schedules[DAYS][HALLS]; //2D array of Schedule structures for movie schedules
Seat seats[DAYS][HALLS][ROWS][COLS]; //4D array of the seat structures to represent available seats

void displayMovies() {
    printf("Movies Available:\n");
    for (int i = 0; i < FILMS; i++) {
        printf("%d. %s - Duration: %d minutes\n", i + 1, films[i].title, films[i].duration);
    }
}

void displaySeats(int filmIndex, int day) {
    printf("Movie: %s\n", films[filmIndex].title);
    printf("Available Seats for Day %d:\n", day + 1);
    for (int hall = 0; hall < HALLS; hall++) {
        if (schedules[day][hall].filmIndex == filmIndex) { // Checking if the movie is scheduled for the given day and hall
            printf("Hall %d: Time - ", hall + 1);
            switch (schedules[day][hall].hour) { // Displaying the scheduled time for the movie
            case 12:
                printf("12:00");
                break;
            case 16:
                printf("16:00");
                break;
            case 20:
                printf("20:00");
                break;
            default:
                printf(" ");
            }
            printf("\n");
            for (int i = 0; i < ROWS; i++) {
                for (int j = 0; j < COLS; j++) {
                    printf("[%c] ", seats[day][hall][i][j].booked ? 'x' : ' ');
                }
                printf("\n");
            }
            printf("\n");
        }
    }
}

void bookSeats(int filmIndex) { // Function to book seats for a specific movie
    int day, hall, seatsBooked, selectedTime;
    printf("Enter the Day you want to watch the movie (1 to 7): ");
    scanf("%d", &day);
    day--;

    if (day < 0 || day >= DAYS) { // Checking if the entered day is valid
        printf("Invalid day!\n");
        return;
    }

    displaySeats(filmIndex, day); // Displaying available seats for the selected day and movie
    
    printf("Enter the Movie session time:\n");
    printf("1. 12:00 am\n2. 16:00\n3. 20:00\n");
    printf("Choose the session time (1-3): ");
    scanf("%d", &selectedTime);

    switch (selectedTime) { // Setting the movie session time based on user input
    case 1:
        schedules[day][0].hour = 12; // Setting the hour for the selected movie session
        break;
    case 2:
        schedules[day][0].hour = 16;
        break;
    case 3:
        schedules[day][0].hour = 20;
        break;
    default:
        printf("Invalid choice for session time!\n");
        return;
    }

    printf("Enter the Hall for the selected movie: ");
    scanf("%d", &hall);
    hall--;

    if (hall < 0 || hall >= HALLS || schedules[day][hall].filmIndex != filmIndex) { // Validating selected hall for the movie
        printf("Invalid hall for the selected movie!\n");
        return;
    }

    printf("How many seats do you want to book?: ");
    scanf("%d", &seatsBooked);

    printf("Available Seats for Day %d, Hall %d, Movie: %s - Time: %d:00\n", day + 1, hall + 1, films[filmIndex].title, schedules[day][hall].hour);
    printf("Enter Row and Column for each seat (e.g., 1 2): \n");

    for (int i = 0; i < seatsBooked; i++) {
        int row, col;
        while (1) {
            printf("Seat %d: ", i + 1);
            scanf("%d %d", &row, &col);

            if (row < 1 || row > ROWS || col < 1 || col > COLS || seats[day][hall][row - 1][col - 1].booked) {  // Validating seat selection
                printf("Invalid seat. Please select another seat.\n");
            }
            else {
                seats[day][hall][row - 1][col - 1].booked = true; // Booking the selected seat
                printf("Seat booked successfully!\n");
                break;
            }
        }
    }
    // Rest of the function follows to book seats and save booking information to a file
    FILE* file = fopen("booking_info.txt", "a"); // Opening file to store booking information
    if (file == NULL) {
        printf("Error opening file!\n");
        return;
    }

    fprintf(file, "Movie: %s\n", films[filmIndex].title);  // Writing booking details to file
    fprintf(file, "Day: %d, Hall: %d, Time: %d:00\n", day + 1, hall + 1, schedules[day][hall].hour);
    fprintf(file, "Number of Seats Booked: %d\n", seatsBooked);
    fprintf(file, "\n");

    fclose(file); //Closing file
    printf("Data saved to booking_info.txt\n");
}

int main() {
    printf("Welcome to Suzette - Booking Cinema Tickets\n");
    printf("Press Enter to continue...");
    getchar(); // Waiting for Enter key press

    int filmIndex;
    displayMovies(); // Display available movies
    printf("Enter the number of the film you want to watch: ");
    scanf("%d", &filmIndex);
    filmIndex--;

    if (filmIndex < 0 || filmIndex >= FILMS) { // Validating movie choice
        printf("Invalid choice!\n");
        return 1;
    }

    bookSeats(filmIndex); // Booking seats for the selected movie

    printf("Thank you for choosing Suzette!\n");

    return 0;
}
