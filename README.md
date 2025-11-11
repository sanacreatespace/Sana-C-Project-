//# Sana-C-Project-
//Login Authentication System

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <termios.h>
#include <unistd.h>

#define USERS_FILE "users.txt"
#define MAX_LINE 512
#define HASH_LEN 64

// Function to hide password input
void get_password(char *password, const char *prompt) {
    struct termios oldt, newt;
    int i = 0;
    char ch;

    printf("%s", prompt);
    tcgetattr(STDIN_FILENO, &oldt);
    newt = oldt;
    newt.c_lflag &= ~(ECHO); // disable echo
    tcsetattr(STDIN_FILENO, TCSANOW, &newt);

    while ((ch = getchar()) != '\n' && ch != EOF && i < 49) {
        password[i++] = ch;
    }
    password[i] = '\0';

    tcsetattr(STDIN_FILENO, TCSANOW, &oldt);
    printf("\n");
}

// Simple hash (for demo purposes only)
void simple_hash(const char *input, char *output) {
    unsigned long hash = 5381;
    int c;
    while ((c = *input++))
        hash = ((hash << 5) + hash) + c; // hash * 33 + c
    sprintf(output, "%lx", hash);
}

// Check if username already exists
int username_exists(const char *username, char *stored_hash) {
    FILE *f = fopen(USERS_FILE, "r");
    if (!f) return 0;
    char line[MAX_LINE];
    while (fgets(line, sizeof(line), f)) {
        line[strcspn(line, "\n")] = 0;
        char *sep = strchr(line, ':');
        if (!sep) continue;
        *sep = '\0';
        if (strcmp(line, username) == 0) {
            if (stored_hash) strcpy(stored_hash, sep + 1);
            fclose(f);
            return 1;
        }
    }
    fclose(f);
    return 0;
}

// Register user
void register_user() {
    char username[50], password[50], confirm[50], hash[HASH_LEN];

    printf("Enter new username: ");
    scanf("%49s", username);
    getchar(); // clear newline

    if (username_exists(username, NULL)) {
        printf("❌ Username already exists.\n");
        return;
    }

    get_password(password, "Enter password: ");
    get_password(confirm, "Confirm password: ");

    if (strcmp(password, confirm) != 0) {
        printf("❌ Passwords do not match.\n");
        return;
    }

    simple_hash(password, hash);

    FILE *f = fopen(USERS_FILE, "a");
    if (!f) {
        printf("Error opening file.\n");
        return;
    }
    fprintf(f, "%s:%s\n", username, hash);
    fclose(f);

    printf("✅ User '%s' registered successfully!\n", username);
}

// Login user
void login_user() {
    char username[50], password[50], hash[HASH_LEN], stored_hash[HASH_LEN];

    printf("Enter username: ");
    scanf("%49s", username);
    getchar(); // clear newline

    if (!username_exists(username, stored_hash)) {
        printf("❌ Username not found.\n");
        return;
    }

    get_password(password, "Enter password: ");
    simple_hash(password, hash);

    if (strcmp(hash, stored_hash) == 0)
        printf("✅ Login successful! Welcome, %s.\n", username);
    else
        printf("❌ Incorrect password.\n");
}

// Main menu
int main() {
    int choice;

    printf("===== LOGIN AUTHENTICATOR =====\n");
    printf("1. Register\n");
    printf("2. Login\n");
    printf("3. Exit\n");
    printf("===============================\n");

    printf("Enter your choice: ");
    scanf("%d", &choice);
    getchar(); // clear newline

    switch (choice) {
        case 1:
            register_user();
            break;
        case 2:
            login_user();
            break;
        case 3:
            printf("Exiting...\n");
            break;
        default:
            printf("Invalid choice.\n");
    }

    return 0;
}

