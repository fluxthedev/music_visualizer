# About
Welcome to the project page for my music visualizer :)

Here is a comparison between what my app looks like and what other music
visualizers might look like when visualizing the same sound.

![](/docs/anim.gif)

# Examples

<img width="400" height="200" src="example0.PNG"> <img width="400" height="200" src="example3.png"> <img width="400" height="200" src="example6.PNG">
<img width="400" height="200" src="example1.png"> <img width="400" height="200" src="example4.png"> <img width="400" height="200" src="example7.PNG">
<img width="400" height="200" src="example2.png"> <img width="400" height="200" src="example5.png"> <img width="400" height="200" src="example8.PNG">

# Building

First get the sources:
```
git clone --recursive https://github.com/xdaimon/music_visualizer.git
```
Then to build on Ubuntu with gcc version >= 5.5:
```
sudo apt install cmake libglfw3-dev libglew-dev libpulse-dev
cd music_visualizer
mkdir build
mkdir build_result
cd build
cmake ..
&& make -j4
&& mv main ../build_result/music_visualizer
&& cp -r ../src/shaders ../build_result/shaders
```
And on Windows 10 with Visual Studio (VS 2019 recommended for x64, VS 2017 also suitable):

1.  **Open the Solution:**
    *   Launch Visual Studio.
    *   On the start screen, choose "Open a project or solution".
    *   Navigate to the root directory of the cloned repository and select the `music_visualizer.sln` file. Click "Open".

2.  **Select Build Configuration:**
    *   Once the project is loaded, look for the toolbar at the top. You should see dropdown menus for "Solution Configurations" and "Solution Platforms".
    *   Select **Release** from the "Solution Configurations" dropdown.
    *   Select **x64** from the "Solution Platforms" dropdown.

3.  **Build the Project:**
    *   In the Visual Studio menu bar, go to **Build > Build Solution**.
    *   Alternatively, you can press the keyboard shortcut `Ctrl+Shift+B`.
    *   The build process will compile the project. You can monitor the progress in the "Output" window at the bottom of Visual Studio.

4.  **Locate the Executable and Prepare for Quick Start:**
    *   After a successful build, the executable `music_visualizer.exe` will typically be located in the `x64/Release` directory inside your solution directory (the main `music_visualizer` folder). For example, if your solution is in `C:\projects\music_visualizer\`, the executable will be in `C:\projects\music_visualizer\x64\Release\`.
    *   **Important for Quick Start:** The "Quick Start" section and default shader loading behavior expect the executable and shaders to be in a `build_result` directory. To align with this:
        *   Create a new folder named `build_result` in the root of your repository (e.g., `C:\projects\music_visualizer\build_result\`).
        *   Copy the compiled `music_visualizer.exe` from `x64/Release/` into the `build_result/` folder.
        *   Copy the default shaders by copying the entire `src/shaders` directory into your `build_result/` folder (it should become `build_result/shaders`).

**Note for Windows users:** Ensure you have the "Desktop development with C++" workload installed in Visual Studio. Visual Studio 2019 (which uses the v142 toolset for x64) is recommended as it matches the project files, but VS 2017 (v141 toolset) should also work. The great news is that the `libs_win` directory in the repository contains pre-built versions of GLEW and GLFW and the Visual Studio project is already configured to use them. This means you **do not** need to download or configure these libraries separately.

# Quick Start

After successfully building the project, you will find the executable (e.g., `music_visualizer` on Linux or `music_visualizer.exe` on Windows) in the `build_result` directory.

The build process also copies a default set of shaders into a `shaders` subdirectory within `build_result` (i.e., `build_result/shaders`).

To run the visualizer with the default shaders:
1. Navigate to the `build_result` directory.
   ```bash
   cd build_result
   ```
2. Execute the program:
   - On Linux: `./music_visualizer`
   - On Windows: `music_visualizer.exe`

This will load the shaders from the `build_result/shaders` directory.

To run the visualizer with a specific shader set (for example, the `retrowave` shaders located in `shaders/retrowave`):
1. Navigate to the `build_result` directory.
   ```bash
   cd build_result
   ```
2. Run the executable with the path to the desired shader directory as an argument:
   ```bash
   ./music_visualizer shaders/retrowave 
   ```
   (Use `music_visualizer.exe` on Windows)

# Usage

If you want to go beyond running pre-existing shaders and start creating or modifying your own, this section explains how the shader system works.

The user writes a .frag file that renders to a window sized quad. If the user wants multipass buffers, then multiple .frag files should be written. When a frag file is saved the app automatically reloads the changes. If the frag file compiles correctly, then the changes are presented to the user otherwise the app ignores the changes.

The name of a buffer is the file name of the frag file without the .frag extension. A buffer's output is available in all buffers as i{Filename w/o extension}. So if the files A.frag and B.frag exist, then buffer B can access the contents of buffer A by doing texture(iA, pos);.

Every shader must contain an image.frag file, just like shadertoy.

Buffers are rendered in alphabetical order and image.frag is always rendered last. If two buffers have the same name but different case, such as A.frag and a.frag, then the render order is unspecified. Do not use non ascii characters in file names ( I use tolower in the code to alphabetize the buffer file list ).

When creating new shaders, you should typically do so within the `build_result/shaders/` directory. Create a new subdirectory for your custom shader (e.g., `build_result/shaders/my_custom_shader/`) and place your `.frag` files there. Subdirectories of the currently active shader path (e.g., `build_result/shaders/my_custom_shader/some_other_folder/`) will not be considered part of the currently rendered shader.

A typical folder layout for a custom shader might look like this:

```
build_result/
    music_visualizer.exe (or ./music_visualizer)
    shaders/
        default_shader_files... (e.g., from the initial `cp -r ../src/shaders ../build_result/shaders` step)
        my_custom_shader/
            image.frag
            buffA.frag
            buffB.frag 
```
You would then run this custom shader by navigating to `build_result` and executing:
`./music_visualizer shaders/my_custom_shader` (or `music_visualizer.exe shaders/my_custom_shader` on Windows).

See [here](/docs/advanced.md) for details on how to configure the rendering process ( clear colors, render size, render order, render same buffer multiple times, geometry shaders, audio system toggle ).

Here is a list of uniforms available in all buffers
```
vec2 iMouse;
bool iMouseDown;     // whether left mouse button is down, in range [0, iRes]
vec2 iMouseDownPos;  // position of mouse when left mouse button was pressed down
vec2 iRes;           // resolution of window
vec2 iBuffRes;       // resolution of currently rendering buffer
float iTime;
int iFrame;
float iNumGeomIters; // how many times the geometry shader executed, useful for advanced mode rendering
sampler1D iSoundR;   // audio data, each element is in the range [-1, 1]
sampler1D iSoundL;
sampler1D iFreqR;    // each element is >= zero for frequency data, you many need to scale this in shader
sampler1D iFreqL;

// Samplers for your buffers, for example
sampler2D iMyBuff; // if you have MyBuff.frag

// Constant uniforms specified in shader.json, for example
uniform vec4 color_set_by_script;
```

By default, the program uses the shader defined in the `shaders` directory (relative to the current working directory, typically `build_result/shaders/`). You can override this by providing a different directory as the first argument when running the program (as shown in the "Quick Start" and custom shader example above).

# Troubleshooting

Here are a few common issues and how to address them:

**Shader Fails to Load/Compile:**
*   Ensure your `.frag` files are plain text files with correct GLSL (OpenGL Shading Language) syntax.
*   Check the console output of the `music_visualizer` executable when you run it. Error messages from the shader compiler are printed there and can help pinpoint syntax errors or other issues in your shader code.
*   Every shader directory must contain an `image.frag` file, which is the final pass rendered to the screen.
*   Verify that buffer names used in `texture()` calls (e.g., `texture(iBuffA, ...)` ) correctly match the corresponding file names (e.g., `BuffA.frag` results in a sampler named `iBuffA`). Case sensitivity might matter depending on the filesystem.

**No Sound Reactiveness / Audio Input Issues:**
*   Verify your system's audio input/output settings. The visualizer needs access to an active audio stream.
*   On Linux, the application often relies on PulseAudio. Ensure PulseAudio is running and correctly configured to capture the desired audio (e.g., monitor of an output device if you want to visualize desktop audio). Tools like `pavucontrol` can help manage PulseAudio settings.
*   On Windows, ensure the correct recording device is enabled and set as default in your Sound control panel.
*   The `iSoundL`, `iSoundR` (raw audio) and `iFreqL`, `iFreqR` (frequency data) samplers provide the audio information to your shaders. If shaders are not reacting to sound, try a very simple shader to dump values from these samplers to see if any audio data is coming through.

**Performance Issues:**
*   Complex shaders, especially those with many passes (multiple `.frag` files), high-resolution buffers, or computationally intensive algorithms, can be demanding on your GPU.
*   If you experience low frame rates, try simplifying your shaders or reducing the number of buffer passes.
*   The resolution of render buffers can also impact performance. See the [advanced documentation](/docs/advanced.md) for details on configuring buffer resolution if needed.

**Visuals Don't Update When Shader File is Saved:**
*   The file watcher should automatically detect changes to `.frag` files in the active shader directory and attempt to reload them.
*   If this isn't happening, ensure the application has the necessary file system permissions to monitor the shader directory.
*   Also, ensure you are saving the files in the correct directory that the visualizer is currently watching.

# Contact

Feel free to use the issues page as a general communication channel.

You can also message me on reddit at /u/xdaimon

# Thanks To

<a href="https://github.com/linkotec/ffts">ffts</a>
	Fast fft library<br>
<a href="https://github.com/karlstav/cava">cava</a>
	Pulseaudio setup code<br>
<a href="https://github.com/kritzikratzi/Oscilloscope">Oscilloscope</a>
	Shader code for drawing smooth lines<br>
<a href="https://github.com/shadowndacorner/SimpleFileWatcher">SimpleFileWatcher</a>
	Asyncronous recursive file watcher<br>
<a href="https://github.com/rapidjson/rapidjson">RapidJson</a>
	Fast json file reader<br>
<a href="https://github.com/catchorg/Catch2">Catch2</a>
	Convenient testing framework<br>
