#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define SIZE 100
#define NAME_LEN 40
#define AILMENT_LEN 80
#define DATAFILE "hospital_data.txt"

typedef struct {
    int id;
    char name[NAME_LEN];
    int age;
    char ailment[AILMENT_LEN];
    int priority; /* higher = more urgent */
} Patient;

Patient q[SIZE];
int n = 0; /* number of patients */

/* safe input line */
void read_line(char *buf, int len) {
    if (!fgets(buf, len, stdin)) return;
    buf[strcspn(buf, "\n")] = '\0';
}

/* find index by id */
int find_idx(int id) {
    for (int i = 0; i < n; ++i) if (q[i].id == id) return i;
    return -1;
}

/* insert by priority (higher first), stable for equal priority */
void enqueue(Patient p) {
    if (n >= SIZE) { puts("Queue full."); return; }
    int pos = n;
    for (int i = 0; i < n; ++i) {
        if (p.priority > q[i].priority) { pos = i; break; }
    }
    for (int i = n; i > pos; --i) q[i] = q[i-1];
    q[pos] = p;
    n++;
    printf("Added: %d - %s (priority %d)\n", p.id, p.name, p.priority);
}

void admit() {
    if (n == 0) { puts("No patients."); return; }
    Patient p = q[0];
    printf("Admitting: %d - %s | Age: %d | Ailment: %s | Priority: %d\n",
           p.id, p.name, p.age, p.ailment, p.priority);
    for (int i = 1; i < n; ++i) q[i-1] = q[i];
    n--;
}

void show_queue() {
    if (n == 0) { puts("No patients waiting."); return; }
    puts("Pos | ID   | Name                 | Age | Prio | Ailment");
    puts("----------------------------------------------------------");
    for (int i = 0; i < n; ++i)
        printf("%3d | %4d | %-20s | %3d |  %3d | %s\n",
               i+1, q[i].id, q[i].name, q[i].age, q[i].priority, q[i].ailment);
}

/* add interactively */
void add_interactive() {
    Patient p;
    char buf[128];
    printf("Enter ID: ");
    if (scanf("%d", &p.id) != 1) { while(getchar()!='\n'); puts("Invalid."); return; }
    while(getchar()!='\n');
    if (find_idx(p.id) != -1) { puts("ID exists."); return; }
    printf("Enter name: "); read_line(p.name, NAME_LEN);
    printf("Enter age: ");
    if (scanf("%d", &p.age) != 1) { while(getchar()!='\n'); puts("Invalid."); return; }
    while(getchar()!='\n');
    printf("Enter ailment: "); read_line(p.ailment, AILMENT_LEN);
    printf("Enter priority (0 low, higher = urgent): ");
    if (scanf("%d", &p.priority) != 1) { while(getchar()!='\n'); puts("Invalid."); return; }
    while(getchar()!='\n');
    enqueue(p);
}

/* search by id */
void search_patient() {
    int id; printf("Enter ID to search: ");
    if (scanf("%d", &id) != 1) { while(getchar()!='\n'); puts("Invalid."); return; }
    while(getchar()!='\n');
    int i = find_idx(id);
    if (i == -1) { puts("Not found."); return; }
    Patient *p = &q[i];
    printf("Found: %d | %s | Age %d | Prio %d | %s\n",
           p->id, p->name, p->age, p->priority, p->ailment);
}

/* update patient by id */
void update_patient() {
    int id; printf("Enter ID to update: ");
    if (scanf("%d", &id) != 1) { while(getchar()!='\n'); puts("Invalid."); return; }
    while(getchar()!='\n');
    int idx = find_idx(id);
    if (idx == -1) { puts("Not found."); return; }
    Patient p = q[idx];
    char buf[128];
    printf("Leave blank to keep current value.\n");
    printf("Name (%s): ", p.name); read_line(buf, sizeof(buf));
    if (buf[0]) strncpy(p.name, buf, NAME_LEN);
    printf("Age (%d): ", p.age); read_line(buf, sizeof(buf));
    if (buf[0]) p.age = atoi(buf);
    printf("Ailment (%s): ", p.ailment); read_line(buf, sizeof(buf));
    if (buf[0]) strncpy(p.ailment, buf, AILMENT_LEN);
    printf("Priority (%d): ", p.priority); read_line(buf, sizeof(buf));
    if (buf[0]) p.priority = atoi(buf);
    /* remove old and re-insert to maintain order */
    for (int i = idx+1; i < n; ++i) q[i-1] = q[i];
    n--;
    enqueue(p);
    puts("Updated.");
}

/* delete by id */
void delete_by_id() {
    int id; printf("Enter ID to delete: ");
    if (scanf("%d", &id) != 1) { while(getchar()!='\n'); puts("Invalid."); return; }
    while(getchar()!='\n');
    int idx = find_idx(id);
    if (idx == -1) { puts("Not found."); return; }
    for (int i = idx+1; i < n; ++i) q[i-1] = q[i];
    n--;
    printf("Deleted ID %d.\n", id);
}

/* save to file */
void save_file() {
    FILE *f = fopen(DATAFILE, "w");
    if (!f) { perror("Save error"); return; }
    fprintf(f, "%d\n", n);
    for (int i = 0; i < n; ++i) {
        fprintf(f, "%d\n%s\n%d\n%s\n%d\n",
                q[i].id, q[i].name, q[i].age, q[i].ailment, q[i].priority);
    }
    fclose(f); puts("Saved.");
}

/* load from file */
void load_file() {
    FILE *f = fopen(DATAFILE, "r");
    if (!f) { puts("No save file. Starting fresh."); return; }
    int total = 0;
    if (fscanf(f, "%d\n", &total) != 1) { fclose(f); puts("Load error."); return; }
    n = 0;
    char buf[128];
    for (int i = 0; i < total && n < SIZE; ++i) {
        Patient p;
        if (!fgets(buf, sizeof(buf), f)) break;
        p.id = atoi(buf);
        if (!fgets(buf, sizeof(buf), f)) break;
        buf[strcspn(buf, "\n")] = '\0'; strncpy(p.name, buf, NAME_LEN);
        if (!fgets(buf, sizeof(buf), f)) break; p.age = atoi(buf);
        if (!fgets(buf, sizeof(buf), f)) break;
        buf[strcspn(buf, "\n")] = '\0'; strncpy(p.ailment, buf, AILMENT_LEN);
        if (!fgets(buf, sizeof(buf), f)) break; p.priority = atoi(buf);
        q[n++] = p;
    }
    fclose(f); printf("Loaded %d patients.\n", n);
}

void stats() { printf("Patients waiting: %d\n", n); }

int main() {
    load_file();
    while (1) {
        puts("\n--- Hospital Management ---");
        puts("1.Add  2.Admit  3.Show  4.Search  5.Update");
        puts("6.Delete 7.Save 8.Load 9.Stats 0.Exit");
        printf("Choice: ");
        int ch;
        if (scanf("%d", &ch) != 1) { while(getchar()!='\n'); puts("Invalid."); continue; }
        while(getchar()!='\n');
        switch (ch) {
            case 1: add_interactive(); break;
            case 2: admit(); break;
            case 3: show_queue(); break;
            case 4: search_patient(); break;
            case 5: update_patient(); break;
            case 6: delete_by_id(); break;
            case 7: save_file(); break;
            case 8: load_file(); break;
            case 9: stats(); break;
            case 0: save_file(); puts("Exiting."); return 0;
            default: puts("Invalid choice.");
        }
    }
    return 0;
}
