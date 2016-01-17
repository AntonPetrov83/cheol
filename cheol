#include <cstdlib>
#include <cstdio>
#include <cstring>

#ifndef PATH_MAX
#define PATH_MAX 4096
#endif

static size_t setLF(char* dest, const char* src, size_t count)
{
	if (count <= 0) return 0;
	char* destpos = dest;
	for (size_t i = 0; i < count; ++i, ++src) {
		if (src[0] == '\r') // skip CR
			continue;

		*destpos++ = *src;
	}
	return destpos - dest;
}

static size_t setCRLF(char* dest, const char* src, size_t count)
{
	if (count <= 0) return 0;
	char* destpos = dest;
	for (size_t i = 0; i < count; ++i, ++src) {
		if (src[0] == '\r')
			continue;
		
		if (src[0] == '\n') {
			*destpos++ = '\r';
			*destpos++ = '\n';
		} else {
			*destpos++ = *src;
		}
	}
	return destpos - dest;
}

typedef size_t (*Converter)(char*, const char*, size_t);

int main(int argc, char* argv[])
{
	if (argc <= 1) {
		fprintf(stderr, "Usage: cheol [-crlf | -lf] <filename>\n");
		return -1;
	}
	
	char infname[PATH_MAX+1] = "";
	char outfname[PATH_MAX+1] = "";
	FILE* infile = NULL;
	FILE* outfile = NULL;
	
#if _WIN32
	Converter convert = setCRLF;
#else
	Converter convert = setLF;
#endif

	for (int i = 1; i < argc; ++ i) {
		if (0 == strcmp(argv[i], "-crlf")) {
			convert = setCRLF;
		} else if (0 == strcmp(argv[i], "-lf")) {
			convert = setLF;
		} else {
			strcpy( infname, argv[i] );
			break;
		}
	}

	infile = fopen(infname, "rb");
	if (!infile) {
		fprintf(stderr, "Failed opening '%s' for reading.\n", infname);
		return -1;
	}

	strcpy(outfname, infname);
	strcat(outfname, ".tmp");
	outfile = fopen(outfname, "wb");
	if (!outfile) {
		fprintf(stderr, "Failed opening '%s' for writing.\n", outfname);
		return -1;
	}

	const size_t BUF_SIZE = 4096;
	char ibuf[BUF_SIZE];
	char obuf[BUF_SIZE * 2];
	for(;;) {
		size_t ibytes = fread(ibuf, 1, BUF_SIZE, infile);
		if (ferror(infile)) {
			fprintf(stderr, "%s: Failed reading file.\n", infname);
			return -1;
		}
		size_t obytes = convert(obuf, ibuf, ibytes);
		if (obytes > 0) fwrite(obuf, 1, obytes, outfile);
		if (ferror(outfile)) {
			fprintf(stderr, "%s: Failed writing to file.\n", outfname);
			return -1;
		}
		if (ibytes < BUF_SIZE && feof(infile)) {
			fflush(outfile);
			break;
		}
	}

	fclose(infile);
	fclose(outfile);

	remove(infname);
	rename(outfname, infname);

	return 0;
}
