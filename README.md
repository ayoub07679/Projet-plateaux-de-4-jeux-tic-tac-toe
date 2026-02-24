# Projet-plateaux-de-4-jeux-tic-tac-toe#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
#include <windows.h>

#define BOARD_SIZE 3
#define X 'X'
#define O 'O'
#define MAX_MSG 100
#define MAX_LEN 256

// --------------------------- Authentification ---------------------------
typedef struct {
    char nom[30];
    char prenom[30];
    int age;
    char email[50];
    char username[30];
    char password[30];
} User;

int registerUser() {
    User u;
    FILE *file = fopen("users.txt", "a");
    if(!file){ printf("Erreur fichier\n"); return 0; }

    printf("Nom : "); scanf("%s", u.nom);
    printf("Prenom : "); scanf("%s", u.prenom);
    printf("Age : "); scanf("%d", &u.age);
    printf("Email : "); scanf("%s", u.email);
    printf("Username : "); scanf("%s", u.username);
    printf("Password : "); scanf("%s", u.password);

    fprintf(file, "%s;%s;%d;%s;%s;%s\n", u.nom,u.prenom,u.age,u.email,u.username,u.password);
    fclose(file);
    printf("Inscription reussie !\n");
    return 1;
}

int loginUser(User *u){
    char username[30], password[30];
    FILE *file = fopen("users.txt", "r");
    if(!file){ printf("Aucun utilisateur\n"); return 0; }

    printf("Username : "); scanf("%s", username);
    printf("Password : "); scanf("%s", password);

    while(fscanf(file,"%[^;];%[^;];%d;%[^;];%[^;];%s\n",
                 u->nom, u->prenom, &u->age,
                 u->email, u->username, u->password)!=EOF){
        if(strcmp(username,u->username)==0 && strcmp(password,u->password)==0){
            fclose(file);
            printf("Connexion reussie !\n");
            return 1;
        }
    }
    fclose(file);
    printf("Identifiants incorrects\n");
    return 0;
}

void logoutUser(User *u){
    printf("Deconnexion de %s %s\n", u->prenom, u->nom);
    u->username[0]='\0';
}

// --------------------------- Tic-Tac-Toe ---------------------------
char board[BOARD_SIZE][BOARD_SIZE];
typedef struct { int player; int computer; int draw; } Score;
Score score;

void setColor(int color){ SetConsoleTextAttribute(GetStdHandle(STD_OUTPUT_HANDLE), color);}
void clearScreen(){ system("cls"); }

void initializeBoard(){ for(int i=0;i<BOARD_SIZE;i++) for(int j=0;j<BOARD_SIZE;j++) board[i][j]=' '; }

void printBoard(){
    clearScreen(); printf("\n");
    for(int i=0;i<BOARD_SIZE;i++){
        for(int j=0;j<BOARD_SIZE;j++){
            if(board[i][j]==X) setColor(1);
            if(board[i][j]==O) setColor(4);
            printf(" %c ",board[i][j]); setColor(7);
            if(j<BOARD_SIZE-1) printf("|");
        }
        printf("\n");
        if(i<BOARD_SIZE-1){
            for(int k=0;k<BOARD_SIZE;k++){ printf("---"); if(k<BOARD_SIZE-1) printf("+"); }
            printf("\n");
        }
    }
    printf("\n");
}

int isValidMove(int row,int col){ return row>=0 && row<BOARD_SIZE && col>=0 && col<BOARD_SIZE && board[row][col]==' '; }

int checkWin(char player){
    for(int i=0;i<BOARD_SIZE;i++){
        int win=1; for(int j=0;j<BOARD_SIZE;j++) if(board[i][j]!=player) win=0;
        if(win) return 1;
    }
    for(int j=0;j<BOARD_SIZE;j++){
        int win=1; for(int i=0;i<BOARD_SIZE;i++) if(board[i][j]!=player) win=0;
        if(win) return 1;
    }
    int win=1; for(int i=0;i<BOARD_SIZE;i++) if(board[i][i]!=player) win=0; if(win) return 1;
    win=1; for(int i=0;i<BOARD_SIZE;i++) if(board[i][BOARD_SIZE-1-i]!=player) win=0; return win;
}

int checkDraw(){ for(int i=0;i<BOARD_SIZE;i++) for(int j=0;j<BOARD_SIZE;j++) if(board[i][j]==' ') return 0; return 1; }

void playerMove(){
    int row,col;
    do{ printf("Votre coup (ligne colonne) : "); scanf("%d %d",&row,&col); row--; col--; } while(!isValidMove(row,col));
    board[row][col]=X;
}

void computerMove(){
    int row,col; srand(time(NULL));
    do{ row=rand()%BOARD_SIZE; col=rand()%BOARD_SIZE; } while(!isValidMove(row,col));
    board[row][col]=O;
}

void updateScore(const char *username){
    FILE *file=fopen("scores.txt","r+"); int found=0; char line[200]; FILE *tmp=fopen("tmp.txt","w");
    if(file){
        while(fgets(line,sizeof(line),file)){
            char name[50]; int p,c,d;
            sscanf(line,"%[^;];%d;%d;%d",name,&p,&c,&d);
            if(strcmp(name,username)==0){ p+=score.player; c+=score.computer; d+=score.draw; fprintf(tmp,"%s;%d;%d;%d\n",username,p,c,d); found=1; }
            else fprintf(tmp,"%s",line);
        }
        fclose(file);
    }
    if(!found) fprintf(tmp,"%s;%d;%d;%d\n",username,score.player,score.computer,score.draw);
    fclose(tmp); remove("scores.txt"); rename("tmp.txt","scores.txt");
}

void playTicTacToe(const char *username){
    score.player=0; score.computer=0; score.draw=0; char again='y';
    while(again=='y'){
        initializeBoard(); char current=X;
        while(1){
            printBoard();
            if(current==X) playerMove(); else computerMove();
            if(checkWin(current)){ printBoard(); if(current==X){ printf("Vous gagnez !\n"); score.player++; } else { printf("Ordinateur gagne !\n"); score.computer++; } break;}
            if(checkDraw()){ printBoard(); printf("Match nul !\n"); score.draw++; break;}
            current = (current==X)?O:X;
        }
        updateScore(username);
        printf("Voulez-vous rejouer ? (y/n) : "); scanf(" %c",&again);
    }
}

// --------------------------- Tours de Hanoï ---------------------------
void playHanoi(){ printf("[INFO] Tours de Hanoï - Fonction non complète\n"); }

// --------------------------- Jeu de Dames ---------------------------
void playDames(){ printf("[INFO] Jeu de Dames - Fonction non complète\n"); }

// --------------------------- Resolver ---------------------------
void playResolver(){ printf("[INFO] Resolver - Fonction non complète\n"); }

// --------------------------- Menu principal ---------------------------
int main(){
    User currentUser;
    int connected=0;
    int choix;

    do{
        printf("===== Authentification =====\n1. Register\n2. Login\n3. Quit\nChoix: ");
        scanf("%d",&choix);
        switch(choix){
            case 1: registerUser(); break;
            case 2: connected = loginUser(&currentUser); break;
        }
    }while(choix!=3 && !connected);

    if(!connected) return 0;

    printf("Bienvenue %s %s !\n", currentUser.prenom, currentUser.nom);

    do{
        printf("\n===== Menu Jeux =====\n1. Tic-Tac-Toe\n2. Tours de Hanoï\n3. Dames\n4. Resolver\n5. Quit\nChoix: ");
        scanf("%d",&choix);
        switch(choix){
            case 1: playTicTacToe(currentUser.username); break;
            case 2: playHanoi(); break;
            case 3: playDames(); break;
            case 4: playResolver(); break;
        }
    }while(choix != 5);

    printf("Merci %s d'avoir joué !\n", currentUser.prenom);
    return 0;
}
