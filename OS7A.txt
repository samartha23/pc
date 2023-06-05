#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <string.h>
#include <ctype.h>

#define BUFFER_SIZE 1024

int main()
{
    int fd1, fd2;
    char *fifo1 = "/tmp/fifo1";
    char *fifo2 = "/tmp/fifo2";
    char buffer[BUFFER_SIZE];

    mkfifo(fifo1, 0666);
    mkfifo(fifo2, 0666);

    pid_t pid = fork();

    if (pid > 0) { // Parent process
        fd1 = open(fifo1, O_WRONLY);
        fd2 = open(fifo2, O_RDONLY);

        printf("Enter a sentence: ");
        fgets(buffer, BUFFER_SIZE, stdin);
        write(fd1, buffer, strlen(buffer)+1);
        close(fd1);

        read(fd2, buffer, BUFFER_SIZE);
        printf("%s", buffer);

        close(fd2);
    } else if (pid == 0) { // Child process
        fd1 = open(fifo1, O_RDONLY);
        fd2 = open(fifo2, O_WRONLY);

        read(fd1, buffer, BUFFER_SIZE);
        int num_chars = strlen(buffer) - 1;
        int num_words = 0;
        int num_lines = 1;

        for (int i = 0; buffer[i] != '\0'; i++) {
            if (isspace(buffer[i])) {
                num_words++;
            }
            if (buffer[i] == '\n') {
                num_lines++;
            }
        }

        char output[BUFFER_SIZE];
        sprintf(output, "Number of characters: %d\nNumber of words: %d\nNumber of lines: %d\n", num_chars, num_words, num_lines);
        int fd = open("output.txt", O_WRONLY | O_CREAT, 0666);
        write(fd, output, strlen(output)+1);
        close(fd);

        fd = open("output.txt", O_RDONLY);
        read(fd, buffer, BUFFER_SIZE);
        write(fd2, buffer, strlen(buffer)+1);
        close(fd);

        close(fd1);
        close(fd2);
    }

    unlink(fifo1);
    unlink(fifo2);

    return 0;
}
