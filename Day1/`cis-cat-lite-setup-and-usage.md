
## ☕️ Java Installation & 📊 CAT Lite Security Scan Setup

---

### 🔧 Step 1: Download & Install Java

```bash
sudo apt update
sudo apt-get install openjdk-11-jdk -y
export JAVA_PATH=/usr/lib/jvm/java-11-openjdk-amd64/bin/
java -version
````

---

### 🐱 Step 2: Download CAT Lite (CIS-CAT Assessor)

1. Go to the official [CIS Security website](https://www.cisecurity.org/).
2. Download **CIS-CAT Lite**.
3. Rename the downloaded file to:

   ```
   cat-lite
   ```
4. Upload `cat-lite` to your target machine using **MobaXterm**.

---

### 📦 Step 3: Extract and Run the Assessor

```bash
cp /home/user/cat-lite .
sudo apt install unzip
unzip cis-cat.zip
ls
cd Assessor/
ls
sudo bash Assessor-CLI.sh -i
```

---

### 📁 Step 4: View the Report

* After running the scan, reports will be available under:

  ```
  /home/mahi/Assessor/reports/
  ```
* Use **MobaXterm** to download the report to your local machine.
* Open the report in a browser for review.

---

### 🛡️ Step 5: Fix Security Issues

* Review the failed checks in the report.
* Fix **all security breaches** until you achieve a score of **80% to 85% or higher**.
* Once issues are addressed, re-run the scan:

```bash
sudo bash Assessor-CLI.sh -i
```

Repeat until compliance is met.

```


Let me know if you'd like to include screenshots or automation for the steps.
```
