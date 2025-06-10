CC=g++
CFLAGS= -g -Wall -pthread # Add -pthread to CFLAGS for consistency

all: proxy

# Compile proxy_parse.c into proxy_parse.o
proxy_parse.o: proxy_parse.c proxy_parse.h
	$(CC) $(CFLAGS) -c proxy_parse.c -o proxy_parse.o

# Compile proxy_server_with_cache.c into proxy.o
proxy.o: proxy_server_with_cache.c proxy_parse.h
	$(CC) $(CFLAGS) -c proxy_server_with_cache.c -o proxy.o

# Link all object files to create the final executable 'proxy'
proxy: proxy_parse.o proxy.o
	$(CC) $(CFLAGS) -o proxy proxy_parse.o proxy.o

clean:
	rm -f proxy *.o

tar:
	tar -cvzf ass1.tgz proxy_server_with_cache.c README Makefile proxy_parse.c proxy_parse.h

# Add a .PHONY rule for common targets that don't produce a file with the same name
.PHONY: all clean tar