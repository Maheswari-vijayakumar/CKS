
### Download Java 

```
sudo apt update
sudo apt-get install openjdk-11-jdk -y && export JAVA_PATH=/usr/lib/jvm/java-11-openjdk-amd64/bin/
java -version
```

### Download cat lite
```
https://www.cisecurity.org/
```

=> rename to cat-lite
=> Then, upload to mobaxteam 

```
cp /home/user/cat-lite .
sudo apt install unzip
unzip cis-cat.zip
ls
cd Assessor/
ls
sudo bash Assessor-CLI.sh -i

```

=> then, report will be generated, you can get it under report folder `/home/mahi/Assessor/reports/` from mobaxstream, then download and review in the browser
=> based on report you have to fix the all security breeches till you get 80 to 85% (once fixed the failed items, should run the command `sudo bash Assessor-CLI.sh -i`



