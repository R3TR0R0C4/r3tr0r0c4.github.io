---
title: "Some scripts or commands i use from time to time"
date: 2026-02-11
categories: [Linux, Scripts, Commands] 
tags: [Scripts, Commands]
---


## FFMPEG

* Conversion for anime with a standard quality for MKVs from h264 to h265:

  Seen some animes like AoT go from 6Gib/Episode to 1Gib/Episode

  * `ffmpeg -fflags +genpts -i input.mkv -map 0:v:0 -map 0:a:0 -map 0:s? -c:v hevc_nvenc -preset p7 -rc vbr -cq 20 -b:v 0 -c:a copy -c:s copy output.mkv`

  * Script to automate whole folder conversions:

    ```bash
    #!/usr/bin/bash

    mkdir -p output_x265

    for f in *.mkv; do
        ffmpeg -fflags +genpts -i "$f" \
        -map 0:v:0 \
        -map 0:a:0 \
        -map 0:s? \
        -c:v hevc_nvenc \
        -preset p7 \
        -rc vbr \
        -cq 20 \
        -b:v 0 \
        -c:a copy \
        -c:s copy \
        "output_x265/$f"
    done
    ```


## Random Scripts

### diff.sh

This script intakes two strings, compares them and outputs, either "Strings are identical", or the the text on the strings that is the same in green, and the diference in red (If a string is longer than the other, it will show up with a `_`)


```bash
#!/bin/bash

# Function to get input from the user
read -e -p "Enter first string: " str1
read -e -p "Enter second string: " str2

# Check if they are exactly the same
if [[ "$str1" == "$str2" ]]; then
    echo -e "\nStrings are identical:"
    echo "$str1"
else
    echo -e "\nDifferences found (Green = Match, Red = Mismatch):"

    # Define color codes
    GREEN='\033[0;32m'
    RED='\033[0;31m'
    NC='\033[0m' # No Color

    # Get lengths
    len1=${#str1}
    len2=${#str2}
    # Determine the longer length to ensure we check every character
    max_len=$(( len1 > len2 ? len1 : len2 ))

    # Loop through each character index
    for (( i=0; i<max_len; i++ )); do
        char1="${str1:$i:1}"
        char2="${str2:$i:1}"

        if [[ "$char1" == "$char2" ]]; then
            # Characters match
            printf "${GREEN}%s${NC}" "$char1"
        else
            # Characters differ (or one string ended)
            # If char1 is empty (index out of bounds), we print a placeholder or nothing
            printf "${RED}%s${NC}" "${char1:-_}"
        fi
    done

    # Print a newline for the first string result
    echo ""

    # Repeat for the second string to show the comparison from its perspective
    for (( i=0; i<max_len; i++ )); do
        char1="${str1:$i:1}"
        char2="${str2:$i:1}"

        if [[ "$char1" == "$char2" ]]; then
            printf "${GREEN}%s${NC}" "$char2"
        else
            printf "${RED}%s${NC}" "${char2:-_}"
        fi
    done
    echo -e "\n"
fi
```