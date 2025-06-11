# Raw Pixel Data Visualizer

A simple yet powerful browser-based tool to visualize raw image data from various pixel formats. This tool allows users to upload raw binary files, specify dimensions and pixel formats, and see the rendered image directly in their web browser. All processing is done client-side.

## Features

*   **Upload Raw Image Files:** Supports uploading raw binary data files.
*   **Specify Image Dimensions:** Input fields for image width and height in pixels.
*   **Comprehensive Pixel Format Selection:** Choose from a wide range of pixel formats, including:
    *   RGB variants (RGB, BGR, GBR)
    *   Grayscale (8-bit)
    *   Planar YUV formats (YUV420p, YUV422p, YUV444p)
    *   Semi-Planar YUV formats (NV12, NV21)
*   **Optional Stride Input:** For packed pixel formats (RGB, BGR, GBR currently), allows specifying a custom row stride (pitch) in bytes if it differs from `width * bytes_per_pixel`.
*   **Client-Side Processing:** All file reading, parsing, and rendering happen directly in the browser using JavaScript. No data is uploaded to a server.
*   **Live Preview:** The image automatically re-renders when any input (file, format, dimensions, stride) is changed.

## How to Use

1.  **Open the Tool:**
    *   Simply open the `index.html` file in a modern web browser (Chrome, Firefox, Edge, Safari).
    *   Alternatively, if hosted (e.g., via Docker or GitHub Pages), navigate to the provided URL.
2.  **Upload Image File:** Click the "Select Image File" button to select your raw image data file.
3.  **Select Pixel Format:** Choose the correct pixel format of your raw data from the "Select Format" dropdown menu.
4.  **Enter Dimensions:**
    *   Input the image "Width" in pixels.
    *   Input the image "Height" in pixels.
5.  **Enter Stride (Optional for RGB/BGR/GBR):**
    *   If your image data for RGB, BGR, or GBR formats has a specific row stride (also known as pitch) that is different from `width * 3 bytes`, enter this value in the "Stride (bytes, optional)" field.
    *   For example, if each row of a 100px wide RGB image occupies 320 bytes in memory (instead of the default 100*3=300), enter `320`.
    *   If the stride is the default (`width * bytes_per_pixel`), you can leave this field empty.
    *   Stride is not currently implemented for YUV or Grayscale formats in this tool.
6.  **View Image:** The image should render automatically on the canvas below the controls. Any changes to the input fields will trigger a re-render.

## Supported Pixel Formats

The tool supports the following pixel formats:

### RGB-like (Packed Pixels)

These formats store pixel data with color components packed together for each pixel. Assumed 8 bits per component (e.g., R8G8B8).

*   **RGB:** Each pixel is stored as Red, Green, Blue components (3 bytes per pixel).
*   **BGR:** Each pixel is stored as Blue, Green, Red components (3 bytes per pixel).
*   **GBR:** Each pixel is stored as Green, Blue, Red components (3 bytes per pixel).
*   **Stride:** For these formats, if a custom stride is provided, it defines the number of bytes from the start of one row of pixels to the start of the next. If the stride is greater than `width * 3`, there is padding at the end of each row.

### Grayscale

*   **Grayscale (8-bit):** Each byte in the input data represents a single pixel's intensity. This value is used for Red, Green, and Blue components, with Alpha set to 255. (1 byte per pixel).
*   **Stride:** Not currently affected by the stride input. Assumes a stride of `width`.

### YUV Planar Formats

YUV data is stored in separate "planes" for Luminance (Y) and Chrominance (U, V).

*   **YUV420p:**
    *   **Y Plane:** `width * height` bytes (full resolution luminance).
    *   **U Plane:** `(width/2) * (height/2)` bytes (chrominance, subsampled by 2 horizontally and vertically).
    *   **V Plane:** `(width/2) * (height/2)` bytes (chrominance, subsampled by 2 horizontally and vertically).
    *   Total bytes: `width * height * 1.5`.
*   **YUV422p:**
    *   **Y Plane:** `width * height` bytes.
    *   **U Plane:** `(width/2) * height` bytes (chrominance, subsampled by 2 horizontally).
    *   **V Plane:** `(width/2) * height` bytes (chrominance, subsampled by 2 horizontally).
    *   Total bytes: `width * height * 2`.
*   **YUV444p:**
    *   **Y Plane:** `width * height` bytes.
    *   **U Plane:** `width * height` bytes (full resolution chrominance).
    *   **V Plane:** `width * height` bytes (full resolution chrominance).
    *   Total bytes: `width * height * 3`.
*   **Stride:** Not currently affected by the stride input for YUV formats. Assumes tightly packed planes.

### YUV Semi-Planar Formats

YUV data is stored with the Y plane first, followed by a single plane containing interleaved chrominance data.

*   **NV12:**
    *   **Y Plane:** `width * height` bytes.
    *   **UV Plane:** `width * (height/2)` bytes. Contains interleaved U and V samples (U0, V0, U1, V1,...). Each pair corresponds to a 2x2 block of Y samples.
    *   Total bytes: `width * height * 1.5`.
*   **NV21:**
    *   **Y Plane:** `width * height` bytes.
    *   **VU Plane:** `width * (height/2)` bytes. Contains interleaved V and U samples (V0, U0, V1, U1,...).
    *   Total bytes: `width * height * 1.5`.
*   **Stride:** Not currently affected by the stride input for YUV formats. Assumes tightly packed planes.

**Note on YUV Subsampling:** For formats with subsampled chroma (YUV420p, YUV422p, NV12, NV21), the U and V values are shared across multiple Y samples. The conversion to RGB reconstructs full color for each pixel. Plane sizes are based on `Math.floor()` for pixel indexing, while file size validation uses `Math.ceil()` to be more tolerant of files produced with padding or slightly different plane dimension calculations.

## Running Locally

Simply download `index.html` and open it directly in any modern web browser. No web server is required for basic functionality.

## Running with Docker (Optional Hosting)

If you wish to host this tool as a web application (e.g., for easy access within a network), you can use Docker:

1.  **Build the Docker image:**
    ```bash
    docker build . -t rawpixels-viewer
    ```
2.  **Run the Docker container:**
    ```bash
    docker run -d -p 8080:80 rawpixels-viewer
    ```
    This will start a container with an Nginx server hosting the `index.html` file.
3.  **Access the tool:** Open your browser and navigate to `http://localhost:8080`.

## GitHub Pages Hosting

This site is a single HTML file with embedded JavaScript and CSS. This makes it ideal for hosting on static site services like GitHub Pages.
To deploy on GitHub Pages:
1.  Ensure `index.html` is in the root of your `gh-pages` branch.
2.  Alternatively, place `index.html` in a `docs/` folder on your `main` (or `master`) branch and configure GitHub Pages to serve from there.
The site will then be available at `your-username.github.io/your-repository-name/`.

## Error Handling Notes

The tool includes basic client-side error handling:
*   Alerts if no file is selected.
*   Alerts if width or height are invalid (not positive numbers).
*   Alerts if the selected file's size does not reasonably match the expected size for the given dimensions and pixel format. This helps catch common errors like incorrect dimensions or format selection.
*   Console warnings may appear for minor discrepancies in file size (e.g., due to padding) or if the end of the file is reached unexpectedly during rendering.

These checks aim to guide the user in correctly inputting parameters for successful image visualization.
"# raw_pixels_viewer" 
