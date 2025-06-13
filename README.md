# image-metadata-exiftool
Instantly get exif data from individual or groups of images by selecting them in Finder and running this quick action

# EXIF Data Quick Action for macOS

A macOS Automator Quick Action that allows you to quickly extract EXIF (Exchangeable Image File Format) data from selected image files directly from Finder. The script uses Phil Harvey's `ExifTool` to provide detailed metadata in a single, consolidated TextEdit window, even when processing multiple images.

---

## Features

* **Quick Access:** Run directly from the right-click (or ctrl-click) "Quick Actions" (or "Services") menu in Finder.
* **Batch Processing:** Select multiple image files, and their EXIF data will be combined into one organized TextEdit window.
* **Detailed Output:** Provides comprehensive EXIF metadata for each image.
* **Error Handling:** Notifies you if `ExifTool` isn't installed or if no EXIF data is found.
* **Automatic Cleanup:** Temporary files created during processing are automatically removed.

---

## Requirements

* macOS (tested on Sequoia 15.5)
* **ExifTool:** The script relies on `ExifTool` being installed on your system.

### Installing ExifTool

The easiest way to install `ExifTool` is by using [Homebrew](https://brew.sh/), a popular package manager for macOS.

1.  **Install Homebrew** (if you haven't already) by opening Terminal.app and running:
    ```bash
    /bin/bash -c "$(curl -fsSL [https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh](https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh))"
    ```
    Follow the on-screen instructions to complete the installation.

3.  **Install ExifTool** using Homebrew by running:
    ```bash
    brew install exiftool
    ```

---

## Installation (Creating the Quick Action)

Once `ExifTool` is installed, follow these steps to set up the Automator Quick Action:

1.  **Open Automator:**
    * Go to your `Applications` folder and open `Automator.app`.
    * Choose **"New Document"** if prompted, or `File > New`.

2.  **Select "Quick Action":**
    * In the "Choose a type for your document" window, select **"Quick Action"** and click "Choose".

3.  **Configure the Quick Action:**
    * At the top of the workflow area, set the following:
        * **"Workflow receives current"**: Set this to **`image files`** or **`files or folders`**
        * **"in"**: Set this to **`Finder.app`**

4.  **Add a "Run Shell Script" action:**
    * In the Library pane on the left, search for **"Run Shell Script"**.
    * Drag and drop the "Run Shell Script" action into the workflow area on the right.

5.  **Configure the Shell Script Action:**
    * Inside the "Run Shell Script" action, set:
        * **"Shell"**: `/bin/bash` (or `/bin/zsh` if that's your preference, but `/bin/bash` is a safe default for Automator).
        * **"Pass input"**: `as arguments`
    * **Replace the default script content** 

6.  **Paste the Script:**

    ```bash
    #!/bin/bash

    # The full path to your exiftool executable.
    # This path is typically /usr/local/bin/exiftool for Homebrew installations.
    EXIFTOOL_PATH="/usr/local/bin/exiftool"

    # Check if exiftool is found at the specified path
    if ! command -v "${EXIFTOOL_PATH}" &> /dev/null
    then
        osascript -e 'display dialog "ExifTool is not found at the specified path or is not executable. Please verify your EXIFTOOL_PATH in the Automator script." with title "ExifTool Not Found" buttons {"OK"} default button "OK" with icon caution'
        exit 1
    fi

    # Create a unique temporary file to store all output.
    TEMP_EXIF_FILE=$(mktemp -t "exif_data.txt")
    # Ensure the temporary file is deleted when the script exits (good practice).
    # This 'trap' command ensures cleanup even if the script encounters an error.
    trap 'rm -f "$TEMP_EXIF_FILE"' EXIT

    # Loop through each selected file passed as an argument to the script
    for f in "$@"
    do
        # Check if the current item is actually a file (and not a folder or other item)
        if [ -f "$f" ]; then 
            # Add a clear header for the EXIF data of the current file
            echo "--- EXIF data for: $(basename "$f") ---" >> "$TEMP_EXIF_FILE"
            
            # Run exiftool on the current file, redirecting both standard output and standard error
            # to the temporary file. This captures all data and any warnings/errors from exiftool.
            "${EXIFTOOL_PATH}" "$f" >> "$TEMP_EXIF_FILE" 2>&1
            
            # Add a blank line for better readability between files' data
            echo "" >> "$TEMP_EXIF_FILE" 
        else
            # If the item is not a file, log a skip message to the temporary file
            echo "--- Skipping non-file item or item not found: $(basename "$f") ---" >> "$TEMP_EXIF_FILE"
            echo "" >> "$TEMP_EXIF_FILE"
        fi
    done

    # --- Final Action: Open the consolidated EXIF data or display a message ---
    # Check if the temporary file contains any content (meaning exiftool produced output).
    # If it has content, it indicates successful processing of at least some EXIF data.
    if [ -s "$TEMP_EXIF_FILE" ]; then 
        # Open the temporary file in TextEdit to display the collected EXIF data.
        open -a "TextEdit" "$TEMP_EXIF_FILE"
    else
        # If the temporary file is empty, display a dialog indicating no data was found or processed.
        osascript -e 'display dialog "No EXIF data found for selected items, or no image files were selected." with title "No Data" buttons {"OK"} default button {"OK"} with icon note'
    fi
    ```

7.  **Save the Quick Action:**
    * Go to `File > Save` (or press `Cmd + S`).
    * Give it a descriptive name like `image-metadata-exiftool` or `Extract Photo Metadata`.

---

## Usage

1.  **Select Image Files:** In Finder, select one or more image files (e.g., `.jpg`, `.png`, `.tif`, etc.).
2.  **Right-Click:** Right-click (or ctrl-click) on any of the selected files.
3.  **Choose "Quick Actions" (or "Services"):** In the contextual menu, hover over or click on "Quick Actions" (or "Services" on older macOS versions).
4.  **Select Your Action:** Click on the name you gave your Quick Action (e.g., "Extract Photo Metadata").

A TextEdit window will open, displaying the combined EXIF data for all selected images!

---

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

---

## Acknowledgements

* A huge thank you to **Phil Harvey** for creating and maintaining the incredible [ExifTool](https://exiftool.org/)!
