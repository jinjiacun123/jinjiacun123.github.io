```
// exampe: sdl2 and open gl 
// compile command: g++ sdl_point.cpp -o sdl_point -lGL `sdl2-config --cflags --libs`
#include <SDL2/SDL.h>
#include <GL/gl.h> // Or include your GLAD header

int main() {
    // 1. SDL Initialization and Window/OpenGL Context Creation
    SDL_Init(SDL_INIT_VIDEO);
    SDL_GL_SetAttribute(SDL_GL_CONTEXT_MAJOR_VERSION, 2); // Example for OpenGL 2.1
    SDL_GL_SetAttribute(SDL_GL_CONTEXT_MINOR_VERSION, 1);
    SDL_GL_SetAttribute(SDL_GL_CONTEXT_PROFILE_MASK, SDL_GL_CONTEXT_PROFILE_COMPATIBILITY); // For glBegin/glEnd

    SDL_Window* window = SDL_CreateWindow("SDL2 OpenGL Point", SDL_WINDOWPOS_UNDEFINED, SDL_WINDOWPOS_UNDEFINED, 800, 600, SDL_WINDOW_OPENGL);
    SDL_GLContext context = SDL_GL_CreateContext(window);
    SDL_GL_MakeCurrent(window, context);

	// open gl setup
	glMatrixMode(GL_PROJECTION);
	glLoadIdentity();
	//glOrtho(0.0, 100.0, 0.0, 100.0, -1.0, 1.0);
	//glOrtho(0.0, 480.0, 0.0, 360.0, 0.0, 1.0);//left, right, down, up, near, far
	glOrtho(0.0, 480.0, 360.0, 0, -1.0, 1.0);//left, right, down, up, near, far


	//setup
	GLuint texture;
    glEnable(GL_TEXTURE_2D);
    glGenTextures(1, &texture);
    glBindTexture(GL_TEXTURE_2D, texture);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_S, GL_CLAMP_TO_EDGE);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_WRAP_T, GL_CLAMP_TO_EDGE);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MIN_FILTER, GL_LINEAR);
    glTexParameteri(GL_TEXTURE_2D, GL_TEXTURE_MAG_FILTER, GL_LINEAR);


    // Initialize GLAD or equivalent here

    bool quit = false;
    SDL_Event event;

    while (!quit) {
        while (SDL_PollEvent(&event)) {
            if (event.type == SDL_QUIT) {
                quit = true;
            }
        }

        // 2. OpenGL Rendering (Drawing a Point)
        glClearColor(1.0f, 1.0f, 1.0f, 1.0f); // Black background
        glClear(GL_COLOR_BUFFER_BIT);

        //glPointSize(10.0f); // Set point size to 10 pixels
        //glColor3f(1.0f, 1.0f, 0.0f); // Red color
	
#if 1
		char * path = "/tmp/my_video_3.bin";                           // [by jim] <------------ 采集出来的3d数据buffer to file as bin
//		uint32_t buffer_width = 8, buffer_height = 8;
		uint32_t buffer_width = 320, buffer_height = 237;              // [by jim] <-------------  需要依据rom buffer的width,height来改写
		uint32_t *data = (uint32_t*)malloc(buffer_width * buffer_height * sizeof(uint32_t));
		memset(data, 0x0, buffer_width * buffer_height * sizeof(uint32_t));
		// Try to open the specified ROM file
		FILE *videoBufFile = fopen(path, "rb");	
		if (!videoBufFile) {
			printf("open videoBufFile is failt\n");
			exit(-1);
		}
		printf("ready to read\n");
		fread(data, sizeof(uint32_t), buffer_width * buffer_height  , videoBufFile);
		fclose(videoBufFile);	
		glTexImage2D(GL_TEXTURE_2D, 0, GL_RGBA, buffer_width,
                    buffer_height, 0, GL_RGBA, GL_UNSIGNED_BYTE, data);

#endif		  
#if 0		
        glBegin(GL_POINTS);
			glVertex2f(0.0f, 0.0f); // Point at the center (normalized device coordinates)
			glVertex2f(1.0f, 1.0f);
        glEnd();
#endif

#if 0
		glBegin(GL_QUADS);
			glVertex2f(40.0f, 40.0f);
			glVertex2f(60.0f, 40.0f);
			glVertex2f(60.0f, 60.0f);
			glVertex2f(40.0f, 60.0f);
		glEnd();
#endif		
	 uint32_t x = 0, y = 0, width = 200, height = 200;
	 glBegin(GL_QUADS);	//[by jim] begin draw
        glTexCoord2i(1, 1);
        glVertex2i(x + width, y + height);
        glTexCoord2i(0, 1);
        glVertex2i(x, y + height);
        glTexCoord2i(0, 0);
        glVertex2i(x, y);
        glTexCoord2i(1, 0);
        glVertex2i(x + width, y);
        glEnd();	//[by jim] draw finish

		glFlush();


        // 3. Swap Buffers
        SDL_GL_SwapWindow(window);
    }

    SDL_GL_DeleteContext(context);
    SDL_DestroyWindow(window);
    SDL_Quit();

    return 0;
}
```