```markdown
# CAT HTB CTF Walkthrough

Welcome to this walkthrough of the **CAT** and Android box challenge from Hack The Box (HTB). This challenge revolves around extracting and analyzing data from a backup file. Let’s dive in!

---

## Tools Required
- **Emulator**
- **ADB (Android Debug Bridge)**

---

## Walkthrough

### Step 1: Provided File
We are given a file named `cat.ab`, which is an Android backup file. This file will serve as the starting point for our investigation.

---

### Step 2: Restoring the Backup File
We can restore the backup contents using the `adb restore` command:

```bash
adb restore cat.ab
```

After running this command, two folders are restored:
- **apps** folder
- **shared** folder

---

### Step 3: Alternative Method to Extract Backup
Instead of restoring the `.ab` file directly, we can extract its contents using a tool called [ABE (Android Backup Extractor)](https://github.com/nelenkov/android-backup-extractor/releases/download/latest/abe-3e9a273.jar).

Use the following command to convert the `.ab` file into a `.tar` archive:

```bash
java -jar abe.jar unpack cat.ab extracted.tar
```

Once the conversion is complete, extract the `.tar` file:

```bash
tar -xf extracted.tar
```

This will reveal the **apps** and **shared** folders.

---

### Step 4: Investigating the Files
#### **Apps Folder**
Upon inspecting the **apps** folder, no relevant data was found.

#### **Shared Folder**
Next, I explored the **shared** folder. Initially, nothing seemed significant. However, after some research and revisiting my approach, I identified a critical detail I had previously overlooked.

#### **Finding the Flag**
Inside the **shared/pictures** folder, there was an image. Upon closer examination of **image 4**, I discovered the flag hidden within the photo.

---

## Conclusion
The challenge required careful attention to detail and persistence. The key was thoroughly inspecting all the extracted files, particularly within the **shared** folder.

---

**Happy hacking!**
```
