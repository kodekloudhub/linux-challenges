# Linux Challenge 4

Some of our apps generate some raw data and store the same in `/home/bob/preserved` directory. We want to clean and manipulate some data and then want to create an `archive` of that data.

`Note`: The validation will verify the final processed data so some of the tests might fail till all data is processed as asked.



All the tasks require you to be root, so the first step is to become root

```bash
sudo -i
```

# Individual Steps

### Find

<details>
<summary>Find the "hidden" files in "/home/bob/preserved" directory and copy them in "/opt/appdata/hidden/" directory (create the destination directory if doesn't exist).</summary>

```bash
mkdir -p /opt/appdata/hidden
find /home/bob/preserved -type f -name ".*" -exec cp "{}" /opt/appdata/hidden/ \;
```

</details>

<details>
<summary>Find the "non-hidden" files in "/home/bob/preserved" directory and copy them in "/opt/appdata/files/" directory (create the destination directory if doesn't exist).</summary>

```bash
mkdir -p /opt/appdata/files
find /home/bob/preserved -type f -not -name ".*" -exec cp "{}" /opt/appdata/files/ \;
```

</details>

<details>
<summary>Find and delete the files in "/opt/appdata" directory that contain a word ending with the letter "t" (case sensitive).</summary>

```bash
rm -f $(find /opt/appdata/ -type f -exec grep -l 't\>' "{}"  \; )
```

</details>

### Replace

<details>
<summary>Change all the occurrences of the word "yes" to "no" in all files present under "/opt/appdata/" directory.</summary>

```bash
find /opt/appdata -type f -name "*" -exec sed -i 's/\byes\b/no/g' "{}" \;
```

</details>

<details>
<summary>Change all the occurrences of the word "raw" to "processed" in all files present under "/opt/appdata/" directory. It must be a "case-insensitive" replacement, means all words must be replaced like "raw , Raw , RAW" etc.</summary>

```bash
find /opt/appdata -type f -name "*" -exec sed -i 's/\braw\b/processed/ig' "{}" \;
```

</details>

### appdata.tar.gz

<details>
<summary>Create a "tar.gz" archive of "/opt/appdata" directory and save the archive to this file: "/opt/appdata.tar.gz"</summary>

```bash
cd /opt
# /opt/appdata contains the final processed data
tar -zcf appdata.tar.gz appdata
```

</details>

### Permissions

<details>
<summary>Add the "sticky bit" special permission on "/opt/appdata" directory (keep the other permissions as it is).</summary>

```bash
chmod +t /opt/appdata
```

</details>

<details>
<summary>Make "bob" the "user" and the "group" owner of "/opt/appdata.tar.gz" file.</summary>

```bash
chown bob:bob /opt/appdata.tar.gz
```

</details>

<details>
<summary>The "user/group" owner should have "read only" permissions on "/opt/appdata.tar.gz" file and "others" should not have any permissions.</summary>

```bash
chmod 440 /opt/appdata.tar.gz
```

</details>

### Softlink

<details>
<summary>
Create a "softlink" called "/home/bob/appdata.tar.gz" of "/opt/appdata.tar.gz" file.</summary>

```bash
ln -s /opt/appdata.tar.gz /home/bob/appdata.tar.gz
```

</details>

### filter.sh and filtered.txt

<details>
<summary>Create a script called "/home/bob/filter.sh".<br>The script should filter the lines from "/opt/appdata.tar.gz" file which contain the word "processed", and save the filtered output in "/home/bob/filtered.txt" file. It must "overwrite" the existing contents of "/home/bob/filtered.txt" file.</summary>

```bash
vi /home/bob/filter.sh
```

Add the following lines and save it.

```bash
#!/bin/bash

tar -xzOf /opt/appdata.tar.gz | grep processed > /home/bob/filtered.txt
```

Make executable, and run it

```bash
chmod +x /home/bob/filter.sh
/home/bob/filter.sh
```
</details>

# Automate the entire lab in a single script!

Pretty much everything done above, in the same order. We automate the creation of `filter.sh` by redirecting a [heredoc](https://linuxize.com/post/bash-heredoc/) with `cat`

<details>
<summary>Single Script Automation</summary>

First, become root

```bash
sudo -i
```

Then


```bash
{
# Paste this entire script to the command prompt.
# When it completes, press the check button.

#################################
#
# Find
#
#################################

echo "Find"
mkdir -p /opt/appdata/hidden
mkdir -p /opt/appdata/files
# Hidden files
find /home/bob/preserved -type f -name ".*" -exec cp "{}" /opt/appdata/hidden/ \;
# non-hidden files
find /home/bob/preserved -type f -not -name ".*" -exec cp "{}" /opt/appdata/files/ \;
# delete files with words ending in 't'
rm -f $(find /opt/appdata/ -type f  -exec grep -l 't\>' "{}"  \; )

#################################
#
# Replace:
#
#################################

echo "Replace"
# Change all the occurrences of the word "yes" to "no"
find /opt/appdata -type f -name "*" -exec sed -i 's/\byes\b/no/g' "{}" \;
# Change all the occurrences of the word "raw" to "processed"
find /opt/appdata -type f -name "*" -exec sed -i 's/\braw\b/processed/ig' "{}" \;

#################################
#
# appdata.tar.gz
#
#################################

echo "appdata.tar.gz"
# Create a "tar.gz" archive of "/opt/appdata" directory and save the archive to this file: "/opt/appdata.tar.gz"
cd /opt
tar -zcf appdata.tar.gz appdata

#################################
#
# Permissions
#
#################################

echo "Permissions"
# Sticky bit
chmod +t /opt/appdata
# Make bob owner
chown bob:bob /opt/appdata.tar.gz
# Set read-only
chmod 440 /opt/appdata.tar.gz

#################################
#
# Softlink
#
#################################

echo "Softlink"
ln -s /opt/appdata.tar.gz /home/bob/appdata.tar.gz

#################################
#
# Filter.sh 
#
#################################

echo "Filter"
cat <<'EOF' > /home/bob/filter.sh
#!/bin/bash

tar -xzOf /opt/appdata.tar.gz | grep processed > /home/bob/filtered.txt
EOF

chmod +x /home/bob/filter.sh

#################################
#
# Filtered.txt
#
#################################

echo "Filtered.txt"
# Execute our script
/home/bob/filter.sh

echo "Complete!"
}
```
</details>
