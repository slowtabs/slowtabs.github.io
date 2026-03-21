---
title: Hello World - Andriod - Escath

---

## Hello World - Andriod - Escathon 2026

> Handout: helloworld.ab

Starting off, I looked online for a way to view .ab files and found this tool on github:

https://github.com/nelenkov/android-backup-extractor/releases/tag/latest

After running the tool on the file, I required a password to decrypt it.
![image](https://hackmd.io/_uploads/SJyicIhLbl.png)

The password was just `hello world`, but I ended up bruting it anyways.

> You can make a hash using `android2john` and brute using rockyou.txt

The backup.tar gave us the andriod RDNS packages. The one that stood out was obviously `com.mcsc.helloworld` 

![image](https://hackmd.io/_uploads/H1wE0LhI-g.png)

There were some database files and an apk inside. I opened the apk files in jadx for static anaylsis. 

![image](https://hackmd.io/_uploads/SJIvZP38-l.png)
![image](https://hackmd.io/_uploads/SyeTfDhIWe.png)

Off the static analysis, I came to the conclusing that `sqlcipher` was used. I also found class `NoteDatabase` in the `MainActivity` class. Which on opening gave us the dbname and password.

![image](https://hackmd.io/_uploads/rJL9zv2Ibx.png)


With info that sqlcipher was imported, I went into the db folder and opened the` .notinhere.db ` using sqlcipher. 

> sqlcipher Usage Steps: (source: Google)

> Set Key: Immediately after opening the database connection, execute PRAGMA key = 'your-passphrase'; to decrypt the database in memory.
>
> Operations: Perform standard SQL queries (SELECT, INSERT, etc.). SQLCipher handles encryption/decryption automatically in the background.

I used the passphrase we got in `NoteDatabase` class and found the flag inside notes db.

![image](https://hackmd.io/_uploads/rJCgrvhUbx.png)

Flag: esch{he1!0o0o0o0o0_w0r!d}

PS: Only android one I solved, but it was super fun. Was my first solve of escathon as well :D

