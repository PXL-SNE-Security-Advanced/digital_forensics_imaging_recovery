# **Lab: File Deletion vs Secure Deletion and File Recovery in Windows 11**

---

## **Context (DFIR Perspective)**

In Digital Forensics & Incident Response (DFIR), investigators must understand **how data is deleted** and **when it can still be recovered**. This lab demonstrates the difference between:

* **Normal file deletion** (logical removal of metadata)
* **Secure deletion (shredding)** (intentional overwriting of data)

You will observe how these actions affect **forensic recoverability** using a standard recovery tool.

---

## **Objective**

After completing this lab, students will be able to:

* Create and format a new volume using **Windows Disk Management**
* Store files on a dedicated volume
* Perform normal deletion vs secure deletion
* Attempt file recovery using **Recuva**
* Explain why some files are recoverable and others are not
* Make a forensic copy of a disk with FTK Imager and recover files from the image with autopsy
  
---

## **Requirements**

* Windows 11 system (physical or virtual machine)
* Administrator privileges
* A secondary disk or virtual disk (≈ 1–2 GB)
* Files:

  * `mypicture.jpg`
  * `pxl.jpg`
* Tools:

  * **Recuva** (installed)
  * **File shredder** (Eraser or Sysinternals SDelete)

---

## **Part 1: Create a New Volume Using Disk Management**

### Step 1 — Open Disk Management

1. Right-click **Start** → **Disk Management**

   * or press **Win + R**, type `diskmgmt.msc`, press Enter

### Step 2 — Identify the new disk

* Look for a disk marked as **Unallocated** or **Not Initialized**
* Identify it by **size** (≈ 1–2 GB)
* **Verify carefully** to avoid selecting the system disk

### Step 3 — Initialize the disk (if prompted)

1. Right-click the disk → **Initialize Disk**
2. Choose **GPT**
3. Click **OK**

### Step 4 — Create a New Simple Volume

1. Right-click the **Unallocated** space
2. Select **New Simple Volume**
3. Follow the wizard:

   * Volume size: default
   * Drive letter: e.g. **F:**
   * File system: **NTFS**
   * Volume label: `DFIR_LAB`
   * Enable **Quick Format**
4. Finish the wizard

### Step 5 — Verify

* Open **File Explorer**
* Confirm `DFIR_LAB (F:)` is visible under **This PC**

---

## **Part 2: Copy Files to the New Volume**

### Step 1 — Create a working folder

1. Open `DFIR_LAB (F:)`
2. Create a folder named:

   ```output
   LabFiles
   ```

### Step 2 — Copy files

Copy the following files into `F:\LabFiles`:

* `mypicture.jpg`
* `pxl.jpg`

### Step 3 — Verify

* Confirm both files are present
* Optional: right-click → **Properties** and note file sizes

---

## **Part 3: Normal Deletion vs Secure Deletion**

### Step 1 — Normal deletion of `mypicture.jpg`

1. Right-click `mypicture.jpg`
2. Click **Delete**
3. Open **Recycle Bin**
4. Right-click `mypicture.jpg` → **Delete**
5. Confirm permanent deletion

**Result:** file metadata is removed, data remains on disk

---

### Step 2 — Secure deletion of `pxl.jpg`

Choose **one** of the following methods.

#### Method A — Eraser (GUI)

1. Install and open **Eraser**
2. Navigate to `F:\LabFiles`
3. Right-click `pxl.jpg`
4. Select **Eraser → Erase**
5. Confirm

#### Method B — SDelete (CLI)

1. Open **Windows Terminal (Admin)**
2. Navigate to SDelete folder
3. Execute:

   ```powershell
   sdelete64.exe -p 1 "F:\LabFiles\pxl.jpg"
   ```

**Result:** file data is overwritten and destroyed

---

### Step 3 — Verify deletion

* Ensure `F:\LabFiles` is empty

---

## **Part 4: File Recovery Using Recuva**

### Step 1 — Launch Recuva

1. Start **Recuva**
2. Use **Wizard mode**

### Step 2 — Configure scan

* File type: **Pictures** (or All Files)
* Location: **Specific location** → `F:\LabFiles`
* Enable **Deep Scan**

Start the scan.

---

## **Part 5: Analyze and Recover Files**

### Step 1 — Inspect results

Look for:

* `mypicture.jpg`
* `pxl.jpg`

Color indicators:

* Green: excellent chance
* Orange: poor
* Red: unrecoverable

### Expected results

* `mypicture.jpg`: **recoverable**
* `pxl.jpg`: **not recoverable or corrupted**

---

### Step 2 — Recover `mypicture.jpg`

1. Select `mypicture.jpg`
2. Click **Recover**
3. Choose a recovery folder on **another drive** (e.g. Desktop)
4. Confirm

### Step 3 — Verify recovery

* Open the recovered image
* Note whether the **original filename** was preserved

---

## **Part 6: Forensic Disk Imaging with FTK Imager**

### Step 1 — Create a forensic image

1. Open **FTK Imager**
2. Go to **File → Create Disk Image**
3. Select **Physical Drive** and choose the disk containing `DFIR_LAB`
4. Click **Next**
5. Select **Image Destination** as **E01** format
6. Choose a destination folder on another drive
7. Click **Finish** to create the image
8. Wait for the imaging process to complete
9. Close FTK Imager

### Step 2 — Analyze the forensic image with Autopsy

1. Open **Autopsy**
2. Create a new case
3. Add the E01 image created in FTK Imager as a data source
4. Start the analysis
5. Navigate to the file system of the image
6. Look for `mypicture.jpg` and `pxl.jpg` in the `DFIR_LAB` folder
7. Attempt to recover both files using Autopsy's file recovery features
8. Verify the integrity of the recovered files
9. Document your findings
10. Close Autopsy
11. Delete the case if desired to free up space
12. Remove the E01 image from your system if no longer needed

---

## **Part 7: DFIR Analysis Questions**

Answer the following:

1. Why was `mypicture.jpg` recoverable after deletion?
2. Why was `pxl.jpg` not recoverable after secure deletion?
3. What is het difference between `del` and `shift+del` in Windows? Can files be recovered in both cases?
4. Did `mypicture.jpg` retain its original filename? Why or why not?
5. Why must recovered files be saved to a different drive?
6. How would SSDs and TRIM affect this experiment?

---

## **Key DFIR Takeaway**

> Deletion does not equal destruction.
>
> Only secure deletion techniques significantly reduce forensic recoverability — and even then, results depend on storage technology.
