
// pipe_file_example.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <string.h>
#include <limits.h>
#define BUF 1024
void child(int fd[2],  char *input) {
    FILE *fil = fopen(input, "r");
    if (!fil) {
        perror("fopen");
        exit(EXIT_FAILURE);
    }
    close(fd[0]);
    char buffer[BUF];
    while (fgets(buffer, sizeof(buffer), fil)) {
        if (write(fd[1], buffer, strlen(buffer)) == -1) {
            perror("write");
            exit(EXIT_FAILURE);
        }
    }
    close(fd[1]);
    fclose(fil);
    exit(EXIT_SUCCESS);
}
void parent(int fd[2],  char *output) {
    FILE *x = fopen(output, "w");
    if (!x) {
        perror("fopen");
        exit(EXIT_FAILURE);
    }
  close(fd[1]);
    char buffer[BUF];
    ssize_t bit;
    while ((bit = read(fd[0], buffer, sizeof(buffer))) > 0) {
        if (fwrite(buffer, 1, bit, x) != bit) {
            perror("fwrite");
            exit(EXIT_FAILURE);
        }
    }
    close(fd[0]);
    fclose(x);
}
int main(int argc, char *argv[]) {
     pid_t id;
    if (argc != 4) {
        fprintf(stderr, "Usage: %s <input_dir> <input_file> <output_dir>\n", argv[0]);
        exit(EXIT_FAILURE);
     }
    char input[PATH_MAX];
    char output[PATH_MAX];
    snprintf(input, sizeof(input), "%s/%s", argv[1], argv[2]);
    snprintf(output, sizeof(output), "%s/out.txt", argv[3]);
    int fd[2];
    if (pipe(fd) == -1) {
        perror("pipe");
        exit(EXIT_FAILURE);
    }
       id = fork();
    if (id == -1) {
        perror("fork");
        exit(EXIT_FAILURE);
    }
    if (id == 0) { 
        child(fd, input);
    } else { 
        parent(fd, output);
        wait(NULL); 
    }
    return 0;
}
