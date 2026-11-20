# Administració de WordPress amb WP-CLI · PorçonsTech

**Sprint 2** · Continuació del WordPress implantat al Sprint 1 (Apache + PHP + MariaDB/MySQL sobre Ubuntu, AWS).
**Autor:** Daniel

En aquest Sprint s'administra el portal corporatiu de PorçonsTech des del terminal amb **WP-CLI**: plugins, temes, actualitzacions, configuració del lloc i verificació final.

## E1 · WP-CLI instal·lat i operatiu

Descàrrega del `.phar` oficial, permisos d'execució i instal·lació com a comandament `wp`:

```bash
cd \~
curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar
chmod +x wp-cli.phar
sudo mv wp-cli.phar /usr/local/bin/wp
wp --info
```

!\[WP-CLI instal·lat](imatges/e1-wp-info.png)

**Què és WP-CLI i quin avantatge té?**
WP-CLI (WordPress Command Line Interface) és una eina per administrar WordPress des del terminal. A diferència de `wp-admin`, permet treballar de forma remota per SSH, és més ràpida i es pot automatitzar amb scripts, cosa que facilita les tasques repetitives en servidors.

### Permisos per treballar amb WP-CLI

Es va assignar l'usuari actual com a propietari de `/var/www/html` mantenint `www-data` com a grup (directoris 775, fitxers 664) i es va comprovar que WP-CLI llig la instal·lació:

```bash
sudo chown -R $USER:www-data /var/www/html
sudo find /var/www/html -type d -exec chmod 775 {} \\;
sudo find /var/www/html -type f -exec chmod 664 {} \\;
cd /var/www/html
wp core version
```

!\[Permisos i versió del core](imatges/permisos-wp-core-version.png)

\---

## E2 · Gestió de plugins amb WP-CLI

Es va treballar amb el plugin **wps-hide-login**: es va configurar la ruta d'accés al panell, es va comprovar al navegador, i es va desactivar i tornar a activar.

```bash
wp option update whl\_page "acces-porconstech"
wp option get whl\_page
wp plugin deactivate wps-hide-login
wp plugin list
wp plugin activate wps-hide-login
```

**Configuració de la nova ruta**

!\[Configuració del plugin](imatges/e2-plugin-config.png)

**`wp plugin list` amb el plugin desactivat**

!\[Llista de plugins](imatges/e2-plugin-list.png)

**Comprovació al navegador:** el login ja no està a `/wp-admin`, ara respon a `/acces-porconstech/`

!\[Login a la nova URL](imatges/e2-login-nueva-url.png)

**Quina operació s'ha fet i com es comprova?**
Es va configurar l'opció `whl\_page` del plugin, i es va desactivar i reactivar. El canvi es va comprovar amb `wp option get whl\_page`, amb `wp plugin list` (columna `status`) i obrint `/acces-porconstech/` al navegador, on apareix el formulari de login.

\---

## E3 · Gestió de temes amb WP-CLI

Es va instal·lar i activar un tema de prova (`twentytwentytwo`), es va comprovar el canvi visual i després es va tornar al tema original `porconstech-child` i s'eliminà el de prova.

```bash
wp theme install twentytwentytwo
wp theme activate twentytwentytwo
wp theme list
# ...
wp theme activate porconstech-child
wp theme delete twentytwentytwo
```

!\[Llista de temes](imatges/e3-theme-list.png)

**Portal amb el tema de prova actiu**

!\[Portal amb el tema nou](imatges/e3-portal-tema.png)

**Com sé quin tema està actiu?**
Amb `wp theme list`, la columna `status` mostra `active` per al tema en ús (i `parent` per al tema pare d'un tema fill). També es pot consultar amb `wp option get stylesheet` i `wp option get template`.

\---

## E4 · Actualitzacions de WordPress

Es va comprovar i gestionar l'estat d'actualitzacions del core, dels plugins i dels temes.

```bash
wp core check-update
wp core update
wp plugin update --all
wp theme update --all
```

**Core: comprovació**

!\[Comprovació del core](imatges/e4-check-update-core.png)

**Core: actualització**

!\[Actualització del core](imatges/e4-core-update.png)

**Plugins**

!\[Actualització de plugins](imatges/e4-plugin-update.png)

**Temes**

!\[Actualització de temes](imatges/e4-theme-update.png)

**Per què és important mantindre-ho actualitzat?**
Les actualitzacions corregeixen errors i vulnerabilitats de seguretat, i aporten millores de rendiment i compatibilitat. Un core, plugin o tema desactualitzat és una de les vies d'atac més habituals contra WordPress.

\---

## E5 · Configuració realitzada amb WP-CLI

S'han fet tres configuracions des del terminal: els **enllaços permanents**, l'**idioma** i les **URL del lloc**.

### Enllaços permanents

```bash
wp rewrite structure '/%postname%/' --hard
wp option get permalink\_structure
```

!\[Enllaços permanents](imatges/e5-permalinks.png)

**Comprovació:** una entrada és accessible amb la nova estructura d'URL (`/hola-mundo/`)

!\[Entrada amb URL nova](imatges/e5-entrada-url.png)

### Idioma

```bash
wp language core list
wp language core install en\_US --activate
wp site switch-language es\_ES
```

!\[Idioma en anglés](imatges/e5-idioma-en.png)

**Comprovació al login amb l'idioma canviat**

!\[Login en anglés](imatges/e5-panel-ingles.png)

Després es va tornar a l'idioma original (`es\_ES`).

**Quina configuració s'ha modificat i com es comprova?**
S'han configurat els enllaços permanents amb `wp rewrite structure` i es va comprovar amb `wp option get permalink\_structure` i obrint una entrada amb la URL nova. També es va canviar l'idioma amb `wp language core install --activate` i es va comprovar al panell de login.

\---

## E6 · Portal final verificat

Després de totes les operacions, es va comprovar que el portal continua operatiu:

```bash
wp core version
wp plugin list
wp theme list
curl -I http://localhost/
```

!\[Verificació al terminal](imatges/e6-verificacion.png)

!\[Portal final](imatges/e6-portal-final.png)

**Quines comprovacions s'han fet?**
Es va revisar la versió del core, que el plugin `wps-hide-login` està actiu, que el tema actiu és `porconstech-child`, que el servidor respon `HTTP/1.1 200 OK` i que la portada i el login carreguen correctament al navegador.

\---

## Incidències resoltes durant la pràctica

|Problema|Causa|Solució|
|-|-|-|
|`curl` descarregava un fitxer de 14 bytes (`404: Not Found`)|URL escrita amb `gh.pages` en lloc de `gh-pages`|Esborrar el fitxer i repetir la descàrrega amb la URL correcta|
|`mariadb.service not found`|El servei de la base de dades es diu `mysql`|Comprovar l'estat amb `systemctl is-active mysql`|
|`/acces-porconstech` donava 404 d'Apache|Enllaços permanents sense configurar i bloc de WordPress buit a `.htaccess`|`wp rewrite structure '/%postname%/' --hard` i regles de reescriptura a `.htaccess`|
|El login no carregava (`ERR\_CONNECTION\_TIMED\_OUT`)|La IP pública de la instància havia canviat i WordPress apuntava a l'antiga|`wp option update siteurl` i `wp option update home` amb la IP actual|

\---

## Tasques finals · PorçonsTech

**1. Administració amb WP-CLI.** Dos avantatges: es pot administrar el servidor de forma remota per SSH sense necessitat del panell web, i les tasques es poden automatitzar i repetir amb scripts.

**2. Plugins.** Procés: instal·lar (`wp plugin install NOM`), activar (`wp plugin activate NOM`), comprovar que funciona (`wp plugin list` i prova al navegador), actualitzar (`wp plugin update NOM`) i, si ja no cal, desactivar i eliminar (`wp plugin deactivate NOM` i `wp plugin delete NOM`).

**3. Usuaris i permisos.**

* Administradora del sistema: **Administrador**, perquè gestiona tot el lloc.
* Persona que revisa i publica continguts: **Editor**, perquè pot publicar i editar entrades de tothom sense gestionar el sistema.
* Persona que només publica les seues entrades: **Autor**, perquè gestiona únicament les seues pròpies entrades. Així s'aplica el principi de mínim privilegi.

**4. Actualitzacions.** Cal mantindre actualitzats WordPress, els plugins i els temes per corregir vulnerabilitats, errors i incompatibilitats. Abans d'una actualització important es comprova que hi ha una còpia de seguretat recent (fitxers i base de dades), la compatibilitat de plugins i tema amb la nova versió i, si és possible, es prova en un entorn de proves.

**5. Backup i exportació.**

* **Còpia de seguretat completa** (fitxers + base de dades): recuperar el lloc sencer després d'una avaria del servidor.
* **Exportació WXR/XML**: traslladar entrades i pàgines a una altra instal·lació WordPress.
* **CSV/JSON**: analitzar dades en un full de càlcul (CSV) o consumir-les des d'una aplicació (JSON).

**6. Sindicació.** Un canal RSS és un document XML que WordPress genera automàticament amb les últimes entrades (`/feed/`). Una altra aplicació o lloc web pot consultar-lo periòdicament i mostrar les novetats de PorçonsTech sense copiar-les manualment.

**7. Anàlisi d'accessos.** El fitxer `/var/log/apache2/access.log` registra la IP del client, la data i hora, el mètode HTTP, el recurs sol·licitat i el codi de resposta. Ajuda l'administrador a detectar errors (404, 500), veure quins recursos es consulten més, identificar accessos sospitosos (per exemple, intents contra `/wp-admin`) i investigar problemes del servidor.

