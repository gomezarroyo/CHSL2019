(this code will not work, its only (mostly formatted notes)

# BASIC PROGRAMMING TIPS

```
ls /  #takes you the highest level directory
ls -lh #function
touch #command to make a file	

mv filename Directoryname/ . #to either move a file or rename
cp #copy file
cp -R #if you wanna copy directory
```
**BE CAREFUL about removing
```
rm -i #remove all 
rm -ir #R is to keep going within the directory

#Accessing txt file

nano
cat 
less 

echo 
#will help you add text to text file 
echo “add text” > filename. 

#use double dash to APPEND rather than OVERWRITE
```

COUNTING CHARACTERS 

```
Wc 
Wc -l #will show you size of the file in bytes
```

MATCHING LINES/FINDING IN FILES

```grep```

**Other basic grep functions

ignore case when matching (```-i```)
only match whole words (```-w```) if NOT it will do pattern recognition
show lines that don’t match a pattern (```-v```)
Use wildcard characters and other patterns to allow for alternatives (*, ., and [])

**Other Useful Commands

```scp``` for transferring files over SSH
```wget``` for downloading files from URL
```watch``` for monitoring
```xargs``` for complex inputs
```ps``` for process status
```top``` for running processes
```kill``` for stopping processes

**Other cool tips

Cut out the 3rd column of a tab-delimited text file and sort it to only show unique lines (i.e. remove duplicates):
 ```cut -f 3 file.txt | sort -u```

Count how many lines in a file contain the words ‘cat’ or ‘bat’ (-c option of grep counts lines):
 ```grep -c '[bc]at' file.txt```

Turn lower-case text into upper-case (using tr command to ‘transliterate’):
 ```cat file.txt | tr 'a-z' 'A-Z'```



