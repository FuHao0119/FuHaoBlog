# Fedora 43: Installing and Using Satty for Screenshots

Satty is a powerful screenshot tool that provides interactive selection and editing capabilities. While it's a great utility, you might find that it's not available directly in the official Fedora 43 repositories. This guide will walk you through the process of manually installing Satty using its binary and setting up the necessary dependencies for use in a Niri (Wayland) environment.

## Prerequisites

*   Fedora 43
*   Niri (Wayland) compositor (or another Wayland environment)
*   Basic terminal knowledge

## Installation Steps for Satty

### 1. Download Satty Binary

Since Satty isn't in the Fedora repositories, you'll need to download its pre-compiled binary directly from the official releases page.

Navigate to the [Satty Releases page](https://github.com/witcherylab/satty/releases) on GitHub.

Look for the latest release and download the file named `satty-x86_64-unknown-linux-gnu.tar.gz`.

### 2. Extract and Move the Binary

Once downloaded, open your terminal, navigate to your Downloads directory (or wherever you saved the file), and extract the archive. Then, move the `satty` executable to a directory included in your system's `PATH`, such as `/usr/local/bin/`, so you can run it from anywhere.

```bash
# Navigate to your downloads directory
cd ~/Downloads

# Extract the tar.gz file
tar -xzf satty-*.tar.gz

# Move the executable to /usr/local/bin/
sudo mv satty /usr/local/bin/
```

### 3. Check and Install Dependencies

For Satty to function correctly in a Wayland environment like Niri, it typically relies on two other tools for screen selection and capture: `slurp` and `grim`. These tools are usually available in Fedora's official repositories.

*   **`slurp`**: Used for interactively selecting a region of the screen with your mouse.
*   **`grim`**: Used to capture screen pixels (take the actual screenshot).

Install these dependencies using `dnf`:

```bash
sudo dnf install slurp grim
```

## Using Satty

Once `satty`, `slurp`, and `grim` are installed, you can start using Satty for your screenshot needs.

A typical usage might look like this:

```bash
satty --output-file ~/Pictures/screenshot-$(date +%Y%m%d%H%M%S).png
```

This command will:
1.  Launch `slurp` to allow you to select a screen region.
2.  Use `grim` to capture the selected area.
3.  Open the captured image in Satty for editing.
4.  Save the final image to your `Pictures` directory with a timestamped filename.

Now you have a powerful screenshot and editing tool at your fingertips on Fedora 43 with Niri!
