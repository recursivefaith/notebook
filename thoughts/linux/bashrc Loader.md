This is my [[thoughts/linux/bashrc|bashrc]] loader. This lets me work with my Linux environments from inside my [[thoughts/gardening/Living Digital Gardens|Living Digital Gardens]]

```bash
# This script extracts and sources Bash commands from a Markdown file.
# It allows you to keep your .bashrc content in a more structured and annotatable format.

# --- Configuration ---
# IMPORTANT: Set the absolute path to your Markdown file containing the .bashrc content.
# This path must be correct for the extractor to work.
export BASHRC_MARKDOWN_FILE="$HOME/github/recursivefaith.github.io/content/thoughts/linux/bashrc.md" # **VERIFY THIS PATH IS 100% ACCURATE**

# Check if the BASHRC_MARKDOWN_FILE variable is set and not empty.
if [ -z "$BASHRC_MARKDOWN_FILE" ]; then
    echo "Error: BASHRC_MARKDOWN_FILE variable is not set. Please define it." >&2
    return 1 # Exit if the path is not set.
fi

# Check if the Markdown file exists before attempting to read it.
if [ ! -f "$BASHRC_MARKDOWN_FILE" ]; then
    echo "Error: Markdown file not found at '$BASHRC_MARKDOWN_FILE'." >&2
    return 1 # Exit if the file does not exist.
fi

# --- Extraction Logic (Using a temporary file for reliable sourcing) ---
# Create a temporary file to hold the extracted Bash code.
# mktemp creates a unique temporary file and prints its name.
TEMP_BASHRC_SNIPPET=$(mktemp)

# Use sed to extract only the content between '```bash' and '```'
# and write it directly to the temporary file.
#   -n: Suppresses automatic printing (important to prevent outputting to terminal).
#   /^```bash$/,/^```$/: Defines a range from the start '```bash' to the end '```'.
#   {...}: Commands to apply within that range.
#   /^```bash$/d: If the current line is '```bash', delete it (don't print).
#   /^```$/d: If the current line is '```', delete it (don't print).
#   p: Print any lines that were not deleted within the range to standard output.
#   > "$TEMP_BASHRC_SNIPPET": Redirects sed's output to our temporary file.
sed -n '/^```bash$/,/^```$/{
    /^```bash$/d;
    /^```$/d;
    p
}' "$BASHRC_MARKDOWN_FILE" > "$TEMP_BASHRC_SNIPPET"

# Check if any content was actually extracted to the temporary file.
if [ ! -s "$TEMP_BASHRC_SNIPPET" ]; then
    echo "Warning: No Bash code found or extracted from '$BASHRC_MARKDOWN_FILE'." >&2
    rm -f "$TEMP_BASHRC_SNIPPET" # Clean up the empty temp file
    return 0 # Exit gracefully if no code was extracted
fi

# Source the temporary file. This executes the extracted commands
# in the current shell context, which is what we need.
source "$TEMP_BASHRC_SNIPPET"

# Remove the temporary file after sourcing.
rm -f "$TEMP_BASHRC_SNIPPET"

# Clean up variables used by this script to avoid cluttering the environment.
unset BASHRC_MARKDOWN_FILE
unset TEMP_BASHRC_SNIPPET
```