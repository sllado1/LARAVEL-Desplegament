# Repositoris Privats  

Podem utilitzar el nostre servidor per allotjar repositoris privats i fins i tot podem utilitzar el control de versions per publicar les actualitzacions dels nostres projectes.  

---

## Crear un repositori privat en el nostre servidor  

Primer de tot, hem d’accedir al nostre servidor:  

```sh
$ ssh totmaquinacendrassos@totmaquinacendrassos.cat
```
Un cop haguem accedit, cal crear una carpeta on col·locar els repositoris:

```sh

$ cd
$ mkdir repositoris
$ cd repositoris
$ mkdir projecte.git  # Acabar la carpeta amb .git és una convenció.
$ cd projecte.git
```
Ara inicialitzarem un repositori GIT en aquesta carpeta. Com que no utilitzarem la carpeta per treballar, crearem un repositori del tipus bare. Aquests repositoris no tenen directori de treball i només s'hi pot interactuar amb git push o git pull.

```sh
$ git init --bare
```

Ja hem creat un repositori privat en el nostre servidor.

## Connectar el nostre repositori local al repositori privat
Per vincular el nostre repositori local amb el repositori privat del servidor, haurem de definir un remot.

Primer, obtenim el path fins al nostre repositori dins del servidor (Això ho farem al servidor):

```sh
$ pwd
```
** /home/totmaquinacendrassos/laravel/repositoris/projecte.git **

Ara afegirem un remot en el nostre repositori local:

```sh
git remote add web ssh://totmaquinacendrassos@totmaquinacendrassos.cat/home/totmaquinacendrassos/laravel/repositoris/projecte.git
```
Per comprovar si hem configurat bé el remot:

```sh
git remote -v
```

## Publicar utilitzant Git
Podem utilitzar el nostre repositori remot i els hooks de Git per automatitzar el procés de publicació.
Aquest mètode és útil per projectes petits o per projectes que requereixin un control precís del procés de publicació.
Aquesta tècnica és més fiable que eines com FTP o SCP, ja que:

- Garanteix que tots els fitxers modificats siguin publicats.
- Permet executar processos abans i després de la publicació (còpies de seguretat, aturar serveis, actualitzar dependències...).

### 1️ - Crear un Hook post-receive
Accedim al directori del repositori:

```sh
$ cd /laravel/repositoris/projecte.git
$ cd hooks
```
A la carpeta hooks hi ha scripts que Git executarà en moments clau. Crearem l’script **post-receive**:

```sh

$ touch post-receive
$ chmod 700 post-receive
```

Editem el fitxer post-receive i afegim la següent configuració bàsica:

```sh
#!/bin/sh
prod="$HOME/www"
while read oldrev newrev ref
do
  branch=`echo $ref | cut -d/ -f3`
  #Si la branca es main es fa el desplegament
  if [ "$branch" = 'main' ]; then
    workdir=$prod
    if [ ! -d "$workdir" ]; then
       echo "Directori de treball $workdir no existeix. Abortant."
       exit 1
    fi
    echo 'Canvis publicats a producció.'
  fi
   git --work-tree=$workdir checkout -f $branch
done
#Aquí es poden posar totes les instruccions necessàries 
```
## Altres configuracions útils
### Canviar la versió de PHP

Hosting → Servidor → PHP

### Afegir dependències amb Composer
Com instal·lar [composer al servidor](https://dinahosting.com/ayuda/que-es-y-como-instalo-composer-en-mi-hosting/).
Per instal·lar les dependències del projecte:

```sh
php ../composer.phar install
```

### Redirigir la pàgina

El projecte s'ha de posar a la carpeta **www** i en aquest s'ha de posar un fitxer **.htaccess** amb el contingut següent:

```sh
RewriteEngine On
RewriteCond %{REQUEST_URI} !^/public/
RewriteRule ^(.*)$ /public/$1 [L]
```
Els fitxers .htaccess faciliten el canvi de configuració de l'Apache a nivell de carpeta. Aquests fitxers només s'han d'utilitzar quan no es té accés a la configuració principal. Per tant, és el seu ús és comu en proveïdors de hosting, on a cada servidor hi ha varis dominis allotjats, i no es té accés a la configuració.

__Què fa aquestes línies de codi a un fitxer .htaccess__

Aquesta regla .htaccess redirigeix totes les peticions que no van a la carpeta **/public/** cap a aquesta carpeta.
- `RewriteEngine On`: Activa el motor de reescriptura d'Apache.
- `RewriteCond %{REQUEST_URI} !^/public/`: Aquesta condició comprova que la URL sol·licitada no comenci amb /public/.
     - `RewriteCond`: Aquesta directiva s'utilitza per definir una condició que ha de complir-se perquè s'apliqui una regla de reescriptura posterior (en aquest cas, la regla RewriteRule que vèiem abans). Només quan la condició és certa, es processarà la regla associada.
     - `%{REQUEST_URI}` és una variable que conté la part de la URL que segueix al domini. Per exemple, si l'usuari accedeix a `http://www.exemple.com/usuaris`, el valor de `%{REQUEST_URI}` serà `/usuaris`.
     - `!^/public/`: El /public/ és la part de la URL que busquem. Si la URI comença amb /public/, aquesta condició no s'aplicaria.
          -`!`: Aquest símbol és un negatiu. En una expressió regular, significa "no coincideix".
          -`^/public/`: Això és una expressió regular que vol dir "comença amb /public/".
- `RewriteRule ^(.*)$ /public/$1 [L]`: Redirecciona a la mateixa ruta, però afegint public al davant.
     - `^(.*)$` és una expressió regular que captura tota l'URL (sense considerar el domini, només el camí de l'URL).
     - Si l'usuari accedeix a /abc/def, $1 serà abc/def, i la redirecció serà a /public/abc/def.
     - `[L]`: Aquest és un "flag" que indica que si aquesta regla es compleix, no s'hauran d'executar més regles de reescriptura. Si aquesta regla redirigeix la petició, el servidor no processarà cap altra regla .htaccess posterior, fins i tot si hi ha més regles definides.
*Exemple*
Si l'usuari accedeix a la ruta `/usuaris`  serà redirigit a `/public/usuaris`  

### Activar Let’s Encrypt
[Guia oficial](https://dinahosting.com/ayuda/como-activo-lets-encrypt-en-mi-hosting/)

### Treballar amb NPM

#### Instal·lar Node 20 
:warning: Perquè el servidor pugui tenir instal·lat npm cal que s'hagi fet una migració al hosting Professional Passenger. En altres paraules, s'ha d'haver fet una migració a un servidor Passenger. 
```sh
curl -fsSL https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.4/install.sh | bash
source ~/.bashrc
nvm install 20
nvm use 20
```
Comprova la versió node
```sh
node -v
```
## Desplegament d'una aplicació Laravel
```sh
# Instal·la o actualitza les dependències. Si no s'han posat dependències noves no cal.
composer install --no-interaction --prefer-dist --optimize-autoloader --no-dev
# Crear el fitxer .env
# APP_DEBUG = false   Per no exposar informació sensible.

# Migracions de la base de dades
php artisan migrate --force
#  Aplica només les migracions pendents.
#  No demana confirmació i s'executa directament.
#  Segur en producció si les migracions estan correctament preparades.

# Neteja de caches
php artisan cache:clear

# Neteja els tokens expirats de la base de dades
php artisan auth:clear-resets

# Neteja la caché de les rutes
php artisan route:clear

# Neteja la caché de les configuracions. Molt important si s'ha canviat el fitxer .env
php artisan config:clear

# Instal·la les dependències de NodeJs. Si no s'han posat depències noves no cal.
npm install

# Compila el VUE
npm run build

# Combina els fitxers de configuració en un de sol
php artisan config:cache
## ULL! Si executeu l'ordre config:cache durant el vostre procés de desplegament, haureu d'assegurar-vos que només esteu cridant la funció env. Un cop s'hagi guardat la configuració a la memòria cau,
## el fitxer .env no es carregarà i totes les crides a la funció env per a les variables .env tornaran nul·les.

# Emmagatzema events
php artisan event:cache

# Millora l'eficiència de les rutes
php artisan route:cache

# Enllaç simbòlic a storage
php artisan storage:link

```





