# Image Resizer

Image Resizer is a simple desktop application for batch-resizing images and exporting PDF pages as JPEG or PNG files.

## Features

- Resize a folder of images while preserving their aspect ratio
- Set the maximum output dimension in pixels
- Export images as JPEG or PNG
- Select JPEG quality
- Skip images that have already been processed
- Convert each page of a PDF into a separate JPEG or PNG image
- Adjust the PDF render delay if pages are not processed correctly
- Follow progress and errors in the built-in log

## Resize images

1. Open the **Resize images** tab.
2. Select the folder containing the source images.
3. Select a target folder for the resized files.
4. Enter the maximum dimension in pixels.
5. Choose JPEG or PNG and, when applicable, the output quality.
6. Optionally enable **Skip images already processed?**
7. Click the green process triangle.

## Export PDF pages

1. Open the **PDF -> page images** tab.
2. Select the source folder containing the PDF files.
3. Select a target folder for the exported page images.
4. Enter the maximum dimension in pixels.
5. Choose JPEG or PNG.
6. Click the PDF export button.

If some PDF pages are not processed correctly, increase the **Render delay** and try again.

The log panel displays progress and processing details for both operations.
