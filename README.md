# Simple 2D fork

This is a fork of [Simple2D](https://github.com/simple2d/simple2d) to fix some stuff. Whether you want to use it is up to you.

---


Simple 2D is a small, open-source graphics engine providing essential 2D drawing, media, and input capabilities. It's written in C and works across many platforms, creating native windows and interacting with hardware using [SDL](https://www.libsdl.org) while rendering content with [OpenGL](https://www.opengl.org).

Please note this README will be continuously updated as new features are added, bugs are fixed, and other changes are made. [View the release notes](https://github.com/simple2d/simple2d/releases) for a link to that version's documentation.

Open an issue on GitHub if you encounter any problems, have a feature request, or simply want to ask a question. Learn more about [contributing](#contributing) below.

# Building from source
First clone this repo using:

```bash
git clone https://github.com/simple2d/simple2d.git
```

Next, build and install on Unix-like systems, including Windows using MinGW, by running:

```bash
mkdir build; cd build
cmake .. -G Ninja
ninja
```

# Tests
- [`triangle.c`](test/triangle.c) — The "Hello Triangle" example in this README.
- [`testcard.c`](test/testcard.c) — A graphical card, similar to [TV test cards](https://en.wikipedia.org/wiki/Test_card), with the goal of ensuring visuals and inputs are working properly.

## Building and running tests
Enable the CMake `SIMPLE2D_TESTS` option to build tests.


---

# Creating apps with Simple 2D

Making 2D apps is simple! Let's create a window and draw a triangle...

```c
#include <simple2d.h>

void render() {
  S2D_DrawTriangle(
    320,  50, 1, 0, 0, 1,
    540, 430, 0, 1, 0, 1,
    100, 430, 0, 0, 1, 1
  );
}

int main() {

  S2D_Window *window = S2D_CreateWindow(
    "Hello Triangle", 640, 480, NULL, render, 0
  );

  S2D_Show(window);
  return 0;
}
```

Save the code above to a file called `triangle.c` and compile it by running `simple2d build triangle.c` on the command line (in MinGW, run this in a Bash prompt). Now run the app using `./triangle` on macOS and Linux, or `triangle.exe` on Windows, and enjoy your stunning triangle in a 640x480 window at 60 frames per second!

The `simple2d build` command is a helpful shortcut for compiling a single source file. Of course, you can also use a compiler directly, for example on Unix-like systems:

```bash
cc triangle.c `simple2d --libs` -o triangle
```

And on Windows using Visual C++ in a developer command prompt:

```bash
cl triangle.c /I %LOCALAPPDATA%\simple2d /link /LIBPATH %LOCALAPPDATA%\simple2d\simple2d.lib /SUBSYSTEM:CONSOLE

# as a PowerShell command
iex "cl triangle.c $(simple2d --libs)"
```

## 2D basics

Let's learn about structuring applications for 2D drawing and more.

### The window

All rendered content, input, and sound is controlled by the window, so creating a window is the first thing you'll do. Start by declaring a pointer to a `S2D_Window` structure and initialize it using `S2D_CreateWindow()`.

```c
S2D_Window *window = S2D_CreateWindow(
  "Hello World!",  // title of the window
  800, 600,        // width and height
  update, render,  // callback function pointers (these can be NULL)
  0                // flags
);
```

To set the window's width and height the same as the display's, use `S2D_DISPLAY_WIDTH` and `S2D_DISPLAY_HEIGHT` for those values, respectively.

The window flags can be `0` or any one of the following:

```c
S2D_RESIZABLE   // allow window to be resized
S2D_BORDERLESS  // show window without a border
S2D_FULLSCREEN  // show window at fullscreen
S2D_HIGHDPI     // enable high DPI mode
```

Flags can also be combined using the bitwise OR operator, for example: `S2D_RESIZABLE | S2D_BORDERLESS`

The size of the viewport, the area where graphics are drawn in the window, can be set independently of the window size like so:

```c
window->viewport.width  = 400;
window->viewport.height = 300;
```

The viewport has various scaling modes, such as `S2D_FIXED` (viewport stays the same size as the window size changes), `S2D_EXPAND` (viewport expands to fill the window when resized), `S2D_SCALE` (the default, where the viewport scales proportionately and is centered in the window), or `S2D_STRETCH` (viewport stretches to fill the entire window). Set the mode like this:

```c
window->viewport.mode = S2D_FIXED;
```

Before showing the window, these attributes can be set:

```c
window->vsync = false;     // set the vertical sync, true by default
window->icon = "app.png";  // set the icon for the window
```

Once your window is ready to go, show it using:

```c
S2D_Show(window);
```

Any time before or during the window is being shown, these attributes can be set:

```c
// Cap the frame rate, 60 frames per second by default
window->fps_cap = 30;

// Set the window background color, black by default
window->background.r = 1.0;
window->background.g = 0.5;
window->background.b = 0.8;
window->background.a = 1.0;
```

Callback functions can also be changed any time — more on that below. Many values can be read from the `S2D_Window` structure, see the [`simple2d.h`](include/simple2d.h) header file for details.

The window icon can be changed using:

```c
S2D_SetIcon(window, "new_icon.png");
```

Take a screenshot of the window in PNG format, providing a file path:

```c
S2D_Screenshot(window, "./screenshot.png");
```

When you're done with the window, free it using:

```c
S2D_FreeWindow(window);
```

### Update and render

The window loop is where all the action takes place: the frame rate is set, input is handled, the app state is updated, and visuals are rendered. You'll want to declare two essential functions which will be called by the window loop: `update()` and `render()`. Like a traditional game loop, `update()` is used for updating the application state, and `render()` is used for drawing the scene. Simple 2D optimizes both functions for performance and accuracy, so it's good practice to keep those updating and rendering tasks separate.

The update and render functions should look like this:

```c
void update() { /* update your application state */ }
void render() { /* draw stuff */ }
```

Remember to add these function names when calling `S2D_CreateWindow()` (see ["The window"](#the-window) section above for an example).

To exit the window loop at any time, use:

```c
S2D_Close(window);
```

## Drawing

All kinds of shapes and textures can be drawn in the window. Learn about each of them below.

### Shapes

Several geometric shapes are available, like triangles, quadrilaterals (which rectangles and squares can be made from), lines, and circles. Every shape contains vertices, that is, places where two lines meet to form an angle (a triangle has three, for example). For each vertex of a triangle and quadrilateral, there are six values which need to be set: the `x` and `y` coordinates, and four color values. Lines have two vertices, although colors for each corner can be set. Circles have a single center point and color that can be set. When vertices have different color values, the space between them are blended in a gradient.

The shorthand for the examples below are:

```c
x = the x coordinate
y = the y coordinate

// Color range is from 0.0 to 1.0
r = red
g = green
b = blue
a = alpha (opacity)
```

Using this notation, `x2` would be the second `x` coordinate, and `b2` would be the blue value at that vertex.

To draw a triangle, use:

```c
S2D_DrawTriangle(x1, y1, r1, g1, b1, a1,
                 x2, y2, r2, g2, b2, a2,
                 x3, y3, r3, g3, b3, a3);
```

To draw a quadrilateral, use:

```c
S2D_DrawQuad(x1, y1, r1, g1, b1, a1,
             x2, y2, r2, g2, b2, a2,
             x3, y3, r3, g3, b3, a3,
             x4, y4, r4, g4, b4, a4);
```

To draw a line, use:

```c
S2D_DrawLine(x1, y1, x2, y2,
             width,
             r1, g1, b1, a1,
             r2, g2, b2, a2,
             r3, g3, b3, a3,
             r4, g4, b4, a4);
```

To draw a circle, use:

```c
S2D_DrawCircle(x, y, radius, sectors, r, g, b, a);
```

### Images

Images in many popular formats, like JPEG, PNG, and BMP can be drawn in the window. Unlike shapes, images need to be read from files and stored in memory. Simply declare a pointer to an `S2D_Image` structure and initialize it using `S2D_CreateImage()` providing the file path to the image.

```c
S2D_Image *img = S2D_CreateImage("image.png");
```

If the image can't be found, it will return `NULL`.

Once you have your image, you can then change its `x, y` position like so:

```c
img->x = 125;
img->y = 350;
```

Change the size of the image by adjusting its width and height:

```c
img->width  = 256;
img->height = 512;
```

Rotate the image like so:

```c
// Angle should be in degrees
// The last parameter is the point the image should rotate around, either:
//   S2D_CENTER, S2D_TOP_LEFT, S2D_TOP_RIGHT, S2D_BOTTOM_LEFT, or S2D_BOTTOM_RIGHT
S2D_RotateImage(img, angle, S2D_CENTER);

// Or, set a custom point to rotate around
img->rx = 50;
img->ry = 75;

// Set the rotation angle directly
img->rotate = 90;
```

You can also adjust the color of the image like this:

```c
// Default is 1.0 for each, a white color filter
img->color.r = 1.0;
img->color.g = 0.8;
img->color.b = 0.2;
img->color.a = 1.0;
```

Finally, draw the image using:

```c
S2D_DrawImage(img);
```

Since images are allocated dynamically, free them using:

```c
S2D_FreeImage(img);
```

### Sprites

Sprites are special kinds of images which can be used to create animations. To create a sprite, declare a pointer to an `S2D_Sprite` structure and initialize it using `S2D_CreateSprite()` providing the file path to the sprite sheet image.

```c
S2D_Sprite *spr = S2D_CreateSprite("sprite_sheet.png");
```

If the sprite image can't be found, it will return `NULL`.

Clip the sprite sheet to a single image by providing a clipping rectangle:

```c
S2D_ClipSprite(spr, x, y, width, height);
```

The `x, y` position of the sprite itself can be changed like so:

```c
spr->x = 150;
spr->y = 275;
```

Change the size of the sprite by adjusting its width and height:

```c
spr->width  = 100;
spr->height = 100;
```

Rotate the sprite like so:

```c
// Angle should be in degrees
// The last parameter is the point the sprite should rotate around, either:
//   S2D_CENTER, S2D_TOP_LEFT, S2D_TOP_RIGHT, S2D_BOTTOM_LEFT, or S2D_BOTTOM_RIGHT
S2D_RotateSprite(spr, angle, S2D_CENTER);

// Or, set a custom point to rotate around
spr->rx = 50;
spr->ry = 75;

// Set the rotation angle directly
spr->rotate = 90;
```

You can also adjust the color of the sprite image like this:

```c
// Default is 1.0 for each, a white color filter
spr->color.r = 1.0;
spr->color.g = 0.8;
spr->color.b = 0.2;
spr->color.a = 1.0;
```

Finally, draw the sprite using:

```c
S2D_DrawSprite(spr);
```

Since sprites are allocated dynamically, free them using:

```c
S2D_FreeSprite(spr);
```

### Text

Text is drawn much like images. Start by finding your favorite OpenType font (with a `.ttf` or `.otf` file extension), then declare a pointer to a `S2D_Text` structure and initialize it using `S2D_CreateText()` providing the file path to the font, the message to display, and the size.

```c
S2D_Text *txt = S2D_CreateText("vera.ttf", "Hello world!", 20);
```

If the font file can't be found, it will return `NULL`.

You can then change the `x, y` position of the text, for example:

```c
txt->x = 127;
txt->y = 740;
```

Rotate the text like so:

```c
// Angle should be in degrees
// The last parameter is the point the text should rotate around, either:
//   S2D_CENTER, S2D_TOP_LEFT, S2D_TOP_RIGHT, S2D_BOTTOM_LEFT, or S2D_BOTTOM_RIGHT
S2D_RotateText(txt, angle, S2D_CENTER);

// Or, set a custom point to rotate around
txt->rx = 50;
txt->ry = 75;

// Set the rotation angle directly
txt->rotate = 90;
```

Change the color of the text like this:

```c
// Default is 1.0 for each, a white color filter
txt->color.r = 0.5;
txt->color.g = 1.0;
txt->color.b = 0.0;
txt->color.a = 0.7;
```

Finally, draw the text using:

```c
S2D_DrawText(txt);
```

You can also change the text message at any time:

```c
S2D_SetText(txt, "A different message!");

// Format text just like `printf`
S2D_SetText(txt, "Welcome %s!", player);
```

Since text is allocated dynamically, free them using:

```c
S2D_FreeText(txt);
```

## Audio

Simple 2D supports a number of popular audio formats, including WAV, MP3, Ogg Vorbis, and FLAC. There are two kinds of audio concepts: sounds and music. Sounds are intended to be short samples, played without interruption, like an effect. Music is for longer pieces which can be played, paused, stopped, resumed, and faded out, like a background soundtrack.

### Sounds

Create a sound by first declaring a pointer to a `S2D_Sound` structure and initialize it using `S2D_CreateSound()` providing the path to the audio file.

```c
S2D_Sound *snd = S2D_CreateSound("sound.wav");
```

If the audio file can't be found, it will return `NULL`.

Play the sound like this:

```c
S2D_PlaySound(snd);
```

You can get and set the volume of a sound like so:

```c
int volume = S2D_GetSoundVolume(snd);
S2D_SetSoundVolume(snd, 50);  // set volume 50%
```

In addition, get and set the volume of all sounds like this, where the volume is a range between 0 (softest) and 100 (loudest):

```c
int volume = S2D_GetSoundMixVolume();
S2D_SetSoundMixVolume(50);  // set volume 50%
```

Since sounds are allocated dynamically, free them using:

```c
S2D_FreeSound(snd);
```

### Music

Similarly, create some music by declaring a pointer to a `S2D_Music` structure and initialize it using `S2D_CreateMusic()` providing the path to the audio file.

```c
S2D_Music *mus = S2D_CreateMusic("music.ogg");
```

If the audio file can't be found, it will return `NULL`.

Play the music like this, where the second parameter is a boolean value indicating whether the music should be repeated:

```c
S2D_PlayMusic(mus, true);  // play on a loop
```

Only one piece of music can be played at a time. The following functions for pausing, resuming, getting and setting volume, stopping, and fading out apply to whatever music is currently playing:

```c
S2D_PauseMusic();
S2D_ResumeMusic();
S2D_StopMusic();

int volume = S2D_GetMusicVolume();
S2D_SetMusicVolume(50);  // set volume 50%

// Fade out over 2000 milliseconds, or 2 seconds
S2D_FadeOutMusic(2000);
```

Since music is allocated dynamically, free them using:

```c
S2D_FreeMusic(mus);
```

## Input

Simple 2D can capture input from just about anything. Let's learn how to grab input events from the mouse, keyboard, and game controllers.

### Keyboard

There are three types of keyboard events captured by the window: when a key is pressed down, a key is being held down, and a key is released. When a keyboard event takes place, the window calls its `on_key()` function.

To capture keyboard input, first define the `on_key()` function and read the event details from the `S2D_Event` structure, for example:

```c
void on_key(S2D_Event e) {
  // Check `e.key` for the key being interacted with

  switch (e.type) {
    case S2D_KEY_DOWN:
      // Key was pressed
      break;

    case S2D_KEY_HELD:
      // Key is being held down
      break;

   case S2D_KEY_UP:
      // Key was released
      break;
  }
}
```

Then, attach the callback to the window:

```c
window->on_key = on_key;
```

### Mouse

The cursor position of the mouse or trackpad can be read at any time from the window. Note that the top, left corner is the origin, `(0, 0)`.

```c
window->mouse.x;
window->mouse.y;
```

To capture mouse button input, first define the `on_mouse()` function and read the event details from the `S2D_Event` structure, for example:

```c
// `e.button` can be one of:
//   S2D_MOUSE_LEFT
//   S2D_MOUSE_MIDDLE
//   S2D_MOUSE_RIGHT
//   S2D_MOUSE_X1
//   S2D_MOUSE_X2

void on_mouse(S2D_Event e) {
  switch (e.type) {
    case S2D_MOUSE_DOWN:
      // Mouse button was pressed
      // Use `e.button` to see what button was clicked
      // Check `e.dblclick` to see if was a double click
      break;

    case S2D_MOUSE_UP:
      // Mouse button was released
      // Use `e.button` to see what button was clicked
      // Check `e.dblclick` to see if was a double click
      break;

    case S2D_MOUSE_SCROLL:
      // Mouse was scrolled
      // Check `e.direction` for direction being scrolled, normal or inverted:
      //   S2D_MOUSE_SCROLL_NORMAL
      //   S2D_MOUSE_SCROLL_INVERTED
      // Check `e.delta_x` and `e.delta_y` for the difference in x and y position
      break;

    case S2D_MOUSE_MOVE:
      // Mouse was moved
      // Check `e.delta_x` and `e.delta_y` for the difference in x and y position
      break;
  }
}
```

Then, attach the callback to the window:

```c
window->on_mouse = on_mouse;
```

Hide the cursor over the window (and show it again) using:

```c
S2D_HideCursor();
S2D_ShowCursor();
```

### Game controllers

All game controllers are automatically detected, added, and removed. There are two types of events captured by the window: axis motion and button presses. When a button is pressed or a joystick moved, the window calls its `on_controller()` function. Buttons and axes are mapped to a generic Xbox controller layout.

To capture controller input, first define the `on_controller()` function and read the event details from the `S2D_Event` structure, for example:

```c
void on_controller(S2D_Event e) {
  // Check `e.which` for the controller being interacted with

  switch (e.type) {
    case S2D_AXIS:
      // Controller axis was moved
      // Use `e.axis` to get the axis, either:
      //   S2D_AXIS_LEFTX, S2D_AXIS_LEFTY,
      //   S2D_AXIS_RIGHTX, S2D_AXIS_RIGHTY,
      //   S2D_AXIS_TRIGGERLEFT, S2D_AXIS_TRIGGERRIGHT,
      //   or S2D_AXIS_INVALID
      // Use `e.value` to get the value of the axis
      break;

    // For the following button events, use `e.button`
    // to get the button pressed or released, which can be:
    //   S2D_BUTTON_A, S2D_BUTTON_B, S2D_BUTTON_X, S2D_BUTTON_Y,
    //   S2D_BUTTON_BACK, S2D_BUTTON_GUIDE, S2D_BUTTON_START,
    //   S2D_BUTTON_LEFTSTICK, S2D_BUTTON_RIGHTSTICK,
    //   S2D_BUTTON_LEFTSHOULDER, S2D_BUTTON_RIGHTSHOULDER,
    //   S2D_BUTTON_DPAD_UP, S2D_BUTTON_DPAD_DOWN,
    //   S2D_BUTTON_DPAD_LEFT, S2D_BUTTON_DPAD_RIGHT,
    //   or S2D_BUTTON_INVALID

    case S2D_BUTTON_DOWN:
      // Controller button was pressed
      break;

    case S2D_BUTTON_UP:
      // Controller button was released
      break;
  }
}
```

Then, attach the callback to the window:

```c
window->on_controller = on_controller;
```

See the [`controller.c`](test/controller.c) test for an exhaustive example of how to interact with game controllers.

You're certain to find controllers that don't yet have button mappings, especially if they're brand new. See the [community-sourced database](https://github.com/gabomdq/SDL_GameControllerDB) of controller mappings for examples of how to generate mapping strings. Once you have the mapping string, you can register it using `S2D_AddControllerMapping()`, or add several mappings from a file using `S2D_AddControllerMappingsFromFile()` and providing the file path.

# Contributing

> "Simple can be harder than complex: You have to work hard to get your thinking clean to make it simple. But it's worth it in the end because once you get there, you can move mountains." — [Steve Jobs](https://en.wikiquote.org/wiki/Steve_Jobs)

Despite the continuing advancements in computer graphics hardware and software, getting started with simple graphics programming isn't that easy or accessible. We're working to change that.

If you like the project, please consider contributing! Check out the [open issues](https://github.com/simple2d/simple2d/issues) for ideas, or suggest your own. We're always looking for ways to make the project more inviting and improve the developer experience on every platform. Don't worry if you're not an expert in C or graphics APIs — we'll be happy to walk you through it all.

If you _are_ a hardcore C and OS hacker, you should seriously consider contributing to [SDL](https://www.libsdl.org) so we can continue writing games without worrying about the platform details underneath. Take a look at the talks from [Steam Dev Days](http://steamcommunity.com/devdays), especially [Ryan C. Gordon's](https://twitter.com/icculus) talk on [Game Development with SDL 2.0](https://www.youtube.com/watch?v=MeMPCSqQ-34&list=UUStZs-X5W6V3TFJLnwkzN5w).

# About the project

Simple 2D was created by [Tom Black](http://www.blacktm.com), who thought simple graphics programming was way too difficult and decided to do something about it.

Everything is [MIT Licensed](LICENSE.md), so hack away.

Enjoy!
