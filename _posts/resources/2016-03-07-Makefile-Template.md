---
layout: post
title:  "Makefile Template"
date:   2016-03-07 16:30:00
tags: [makefile, template, linux]
categories: Resources
---

> Makefile template to integrate into the project.

```
CC=g++
OBJ_PATH = ./obj
EXE = main
CXXFLAGS = -O2 -Werror -DCPU_ONLY -D__STDC_CONSTANT_MACROS -std=c++11 

LIBRARY = -L/usr/local/lib/ -lm -lpthread -lavutil -lavformat -lavcodec -lswscale -lopencv_core -lopencv_imgproc -lopencv_highgui \
  -L/usr/lib/python2.7/config-x86_64-linux-gnu/ -lpython2.7 \
  -L/sur/lib32/ -lrt

OBJ_SRCS := a.cpp b.cpp c.cpp d.cpp e.cpp \
      f.cpp g.cpp h.cpp
ALL_OBJ = $(patsubst %.cpp, %.o, $(OBJ_SRCS))
OBJ = $(addprefix $(OBJ_PATH)/, $(ALL_OBJ))

all: $(EXE)

$(EXE):  main.cpp $(OBJ)
  $(CC) $(CXXFLAGS) $(OBJ) $< -o $@ $(LIBRARY)

$(OBJ_PATH)/%.o: %.cpp
  @ mkdir -p $(OBJ_PATH) 
  $(CC) -c $(CXXFLAGS) $< -o $@ $(LIBRARY)

clean:
  rm -rf $(OBJ_PATH) $(EXE)
```
