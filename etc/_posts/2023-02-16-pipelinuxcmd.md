---
layout: post
title: Linux Command pipe로 변수값으로 끌고오기
subtitle: c++, linux, command, pipe, arp
gh-repo: harley-hwan/harley-hwan.github.io
gh-badge: [star, fork, follow]
tags: [c, c++, linux, command, pipe, arp]
comments: true
---

# Linux Command pipe로 변수값으로 끌고오기 pipe()
- 최초 작성일: 2023년 2월 16일 (목)

## 목차

[TOC]

<br/>

## 내용

### 소스코드 1

```c++
#include <iostream>
#include <cstdio>
#include <string>

int main() {
    FILE* pipe = popen("arp -a", "r");
    if (!pipe) return 1;

    char buffer[128];
    std::string result = "";
    while (!feof(pipe)) {
        if (fgets(buffer, 128, pipe) != nullptr)
            result += buffer;
    }

    pclose(pipe);

    std::cout << result << std::endl;
    return 0;
}
```

</br>

### 결과 1

```c++
? (192.168.8.152) at 88:36:6c:fc:2c:4f [ether] on wlan0
? (192.168.8.114) at 5a:ff:ec:d1:cb:a4 [ether] on wlan0
```

</br>

</br>

### 소스코드 2

```c++
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>
#include <string.h>

std::vector<std::string> getE6ServerIPpipe()
{
    std::vector<std::string> ip_list;
    //static bool initflag=false;
    //if(initflag)
    //	return ip_list;
    //initflag=true;
    int my_pipe[2];
    char* arguments[] = {"arp",NULL}; 

    if(pipe(my_pipe) == -1)
    {
        fprintf(stderr, "Error creating pipe\n");
    }

	//std::string ip;
	//system("arp -a > /home/pi/test/e6/ip.txt");


    pid_t child_id;
    child_id = fork();
    if(child_id == -1)
    {
        fprintf(stderr, "Fork error\n");
    }
    if(child_id == 0) // child process
    {
        close(my_pipe[0]); // child doesn't read
        dup2(my_pipe[1], 1); // redirect stdout

        execvp(arguments[0], arguments);

        fprintf(stderr, "Exec failed\n");
    }
    else
    {
        close(my_pipe[1]); // parent doesn't write

        char reading_buf[1024];
        char *ptr=reading_buf;
        while(read(my_pipe[0], ptr, 1) > 0)
        {
            //write(1, reading_buf, 1); // 1 -> stdout
            ptr++;
        }

        (*ptr)='\0';
        char *line=strtok(reading_buf,"\n"); // skip
        line=strtok(NULL,"\n");

        while(line)
        {
            int i;
            for(i=0;!isspace(line[i]);i++);

            line[i]='\0';
            ip_list.push_back(line);

            //printf("%s--------------------\n",line);
            line=strtok(NULL,"\n");
        }
        close(my_pipe[0]);

        
        wait();
    }

 
	return ip_list;
    
}
```

<br/>

### 결과 2

```c++
192.168.8.152
192.168.8.114
```

<br/>

<br/>

### 소스코드 3

```c++
std::vector<std::string> getIPList() {
    FILE* pipe = popen("arp -a", "r");
    if (!pipe) {
        std::cerr << "popen() failed!" << std::endl;
        exit(1);
    }

    std::vector<std::string> ip_list;
    char buffer[128];
    std::string arpOutput = "";
    while (!feof(pipe)) {
        if (fgets(buffer, 128, pipe) != nullptr) {
            arpOutput += buffer;
        }
    }
    pclose(pipe);

    size_t pos_left, pos_right;
    while ((pos_left = arpOutput.find("(")) != std::string::npos) {
        pos_right = arpOutput.find(")", pos_left);
        if (pos_right != std::string::npos) {
            std::string token = arpOutput.substr(pos_left + 1, pos_right - pos_left - 1);
            ip_list.push_back(token);
        }
        arpOutput.erase(0, pos_right + 1);
    }
    return ip_list;
}
```

<br/>

### 결과 3

```c++
192.168.8.152
192.168.8.114
```

