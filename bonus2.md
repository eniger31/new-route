# Bonus URLs

Une fois les routes mises en place, les ressources statiques du site sont _cassées_. Si ce n'est pas le cas, essayer avec une autre page que la page d'accueil.  

## Pourquoi ? :mag_right:

- aller dans DevTools de Chrome, onglet Network
- de nombreuses ressources sont en 404
- cliquer dessus, regarder l'URL accédée, analyser

<details><summary>Aide & réponse</summary>

Pour l'URL d'exemple `http://localhost/S05/S05-projet-oShop/public/catalog/category/12`,  
on constate que l'URL pour `style.css` est `http://localhost/S05/S05-projet-oShop/public/catalog/category/assets/css/styles.css`.  
L'URL devrait être `http://localhost/S05/S05-projet-oShop/public/assets/css/styles.css`,  
il y a donc `catalog/category/` en trop. **Pourquoi ?**

Voici le code HTML déterminant l'URL de ce fichier `styles.css` :

```html
<link rel="stylesheet" href="assets/css/styles.css">
```

L'URL est **relative**. C'est-à-dire qu'elle dépend de l'URL dans laquelle on se trouve actuellement.  
Le "point de départ" est le "dossier" dans lequel on se trouve actuellement, **aux yeux du navigateur** !

URL : `http://localhost/S05/S05-projet-oShop/public/catalog/category/12`  
=> "dossier" actuel => `http://localhost/S05/S05-projet-oShop/public/catalog/category/`  
=> on ajoute l'URL relative (`assets/css/styles.css`) => `http://localhost/S05/S05-projet-oShop/public/catalog/category/assets/css/styles.css`  
=> l'URL finale est normale

Comme les URLs sont réécrites (`.htaccess` & _RewriteEngine_), on ne peut pas savoir dans quel "dossier" se trouve la page courante.  
=> on doit définir les URLs pour qu'elles soient fonctionnelles, quelque soit le "dossier" courant  
=> on définit des **URLs absolues**
  
</details>

## On corrige :pencil2:

### URL du dossier public :see_no_evil:

- on veut récupérer dynamiquement l'URL jusqu'au dossier "public" : `http://localhost/S05/S05-projet-oShop/public`
- mais l'URL `/S05/S05-projet-oShop/public` fonctionne tout aussi bien :wink:
- la variable superglobale `$_SERVER` (un tableau) contient pas mal de données intéressantes
- `print_r` ou `var_dump` ou `dump` pour regarder si une des valeurs contient ce qu'on cherche

<details><summary>Réponse</summary>

`$_SERVER['BASE_URI']` contient bien `/S05/S05-projet-oShop/public` quelque soit la page sur laquelle on se trouve.

Pour être précis, c'est le fichier `.htaccess` qui crée cette information et l'envoie à Apache (qui transmet à PHP).

```
# dynamically setup base URI
RewriteCond %{REQUEST_URI}::$1 ^(/.+)/(.*)::\2$
RewriteRule ^(.*) - [E=BASE_URI:%1]
```
  
</details>

### Correction des URLs :hammer:

- dans le `MainController`, dans la méthode `show`
  - créer une variable `$absoluteURL` contenant cette _URL du dossier public_ (vu plus haut)
  - cette variable sera disponible dans tous les fichiers inclus par cette méthode => les _Views_
- faire de la même dans la méthode `show` du `CatalogController`
- dans la _View_ `header.tpl.php`,
  - utiliser cette variable `$absoluteURL` pour rendre toutes les URLs absolues
  - par exemples, pour "styles.css" : `<link rel="stylesheet" href="<?= $absoluteURL ?>/assets/css/styles.css">`
- faire de même pour la _View_ `footer.tpl.php`
- faire de même pour toutes les autres _Views_
