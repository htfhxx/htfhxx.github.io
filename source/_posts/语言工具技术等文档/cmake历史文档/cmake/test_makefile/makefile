
all: hellospeak clean
hellospeak: hellospeak.o speak.o 
	g++ -o hellospeak hellospeak.o speak.o
hellospeak.o: hellospeak.cpp speak.h
	g++ -c hellospeak.cpp
speak.o: speak.cpp speak.h
	g++ -c speak.cpp
clean:
	rm -f  *o 

