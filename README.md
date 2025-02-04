# Repositoris Privats  

Podem utilitzar el nostre servidor per allotjar repositoris privats i fins i tot podem utilitzar el control de versions per publicar les actualitzacions dels nostres projectes.  

---

## 🔹 Crear un repositori privat en el nostre servidor  

Primer de tot, hem d’accedir al nostre servidor:  

```sh
$ ssh videoclub@m08.daw
```
Un cop haguem accedit, cal crear una carpeta on col·locar els repositoris:

```sh

$ cd
$ mkdir repositori
$ cd repositori
$ mkdir videoclub.git  # Acabar la carpeta amb .git és una convenció.
$ cd videoclub.git
```
Ara inicialitzarem un repositori GIT en aquesta carpeta. Com que no utilitzarem la carpeta per treballar, crearem un repositori del tipus bare. Aquests repositoris no tenen directori de treball i només s'hi pot interactuar amb git push o git pull.

sh
Copiar
Editar
$ git init --bare
Ja hem creat un repositori privat en el nostre servidor.

🔹 Connectar el nostre repositori local al repositori privat
Per vincular el nostre repositori local amb el repositori privat del servidor, haurem de definir un remot.

Primer, obtenim el path fins al nostre repositori en el servidor:

sh
Copiar
Editar
$ pwd
/var/www/vhost/videoclub.daw/repositoris/videoclub.git
Ara afegirem un remot en el nostre repositori local:

sh
Copiar
Editar
(màquina local) $ git remote add web videoclub@m08.daw:/var/www/vhost/videoclub.daw/repositori/videoclub.git
Per comprovar si hem configurat bé el remot:

sh
Copiar
Editar
(màquina local) $ git push web main
🔹 Publicar utilitzant Git
Podem utilitzar el nostre repositori remot i els hooks de Git per automatitzar el procés de publicació.

Aquest mètode és útil per projectes petits o per projectes que requereixin un control precís del procés de publicació.

Aquesta tècnica és més fiable que eines com FTP o SCP, ja que:

Garanteix que tots els fitxers modificats siguin publicats.
Permet executar processos abans i després de la publicació (còpies de seguretat, aturar serveis, actualitzar dependències...).
1️⃣ Crear un Hook post-receive
Accedim al directori del repositori:

sh
Copiar
Editar
$ cd /var/www/vhost/videoclub.daw/repositoris/videoclub.git
$ cd hooks
A la carpeta hooks hi ha scripts que Git executarà en moments clau. Crearem l’script post-receive:

sh
Copiar
Editar
$ touch post-receive
$ chmod 700 post-receive
Editem el fitxer post-receive i afegim la següent configuració bàsica:

sh
Copiar
Editar
#!/bin/sh
git --work-tree=/var/www/vhost/videoclub.daw/www checkout main -f
Aquesta versió publica automàticament els canvis de la branca main després d’un git push.

2️⃣ Detectar amb quina branca s’ha fet push
L’script post-receive rep per entrada estàndard (stdin) els paràmetres del push.

sh
Copiar
Editar
#!/bin/sh
read oldrev newrev ref
echo "oldrev $oldrev newrev $newrev ref $ref"
Provem fent push d’una branca de proves:

sh
Copiar
Editar
(màquina local) $ git checkout -b prova
(màquina local) $ git push web prova
Sortida esperada:

sh
Copiar
Editar
remote: oldrev 420e6f... newrev 6ee89c... ref refs/heads/prova
Ara podem ajustar l’script perquè actuï segons la branca amb què s’ha fet push:

sh
Copiar
Editar
#!/bin/sh
prod="$HOME/www"
dev="$HOME/www"
while read oldrev newrev ref
do
  branch=`echo $ref | cut -d/ -f3`
  if [ "$branch" = 'main' ]; then
    workdir=$prod
    if [ ! -d "$workdir" ]; then
       echo "Directori de treball $workdir no existeix. Abortant."
       exit 1
    fi
    echo 'Canvis publicats a producció.'
  else
    workdir=$dev
    echo 'Canvis publicats a desenvolupament.'
  fi
   git --work-tree=$workdir checkout -f $branch
done
🔹 Altres configuracions útils
✅ Canviar la versió de PHP
Hosting → Servidor → PHP

✅ Afegir dependències amb Composer
Per instal·lar el composer i les dependències:

sh
Copiar
Editar
php ../composer.phar install
✅ Redirigir la pàgina
El projecte ha d'estar en una carpeta diferent de www. Després, s’ha de demanar a Dinahosting que facin un enllaç simbòlic de www a <nomcarpeta>/public.

✅ Activar Let’s Encrypt
🔗 Guia oficial

✅ Treballar amb NPM
El nostre hosting no permet instal·lar npm. La solució és:

1️⃣ Generar els arxius .js per producció en local.
2️⃣ Enviar el bundle.js al servidor.

Forçar l’afegit d’un fitxer ignorat per .gitignore:

sh
Copiar
Editar
git add -f path/del/fitxer/bundle.js
git commit -m "Incloc bundle.js malgrat el .gitignore"
git push nomremot nombranca
