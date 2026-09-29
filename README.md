# README — Portfolio d'Elsa Borget

Ce document détaille l'ensemble des fonctionnalités du site : d'abord celles d'origine (ciel étoilé, constellations, trou de ver, jeux, etc.), puis les ajouts faits par la suite (photo, couleur dorée, icônes, section CV).

## Sommaire

**Partie 1 — Fonctionnalités d'origine**
1. Le ciel étoilé animé (canvas)
2. Le nom en points lumineux
3. Les apparitions en fondu
4. Les lettres révélées une à une
5. Les constellations de projets
6. Le trou de ver
7. Le bouton magnétique
8. La fusée vers LinkedIn
9. Le mini-jeu Snake
10. Le jeu de mots mêlés
11. Le système solaire des compétences
12. Le son (bips)
13. Le sélecteur de ciel manuel
14. La barre de progression de scroll
15. Le terminal caché

**Partie 2 — Ajouts**
16. Font Awesome (icônes)
17. Photo de profil
18. Couleur dorée
19. Icônes de compétences
20. Section CV

---

# Partie 1 — Fonctionnalités d'origine

## 1. Le ciel étoilé animé (canvas)

**Emplacement :** `<canvas id="sky">` dans le HTML, et tout le bloc JS qui commence par `var cv=document.getElementById('sky')...` jusqu'à la fonction `loop()`.

**Principe :** un `<canvas>` est une zone de dessin que le JS contrôle entièrement, pixel par pixel, contrairement au HTML classique. Le script y dessine :

- **Les étoiles.** Chacune est un objet `{x, y, z, p}` : `x` et `y` sont sa position (en proportion de l'écran, entre 0 et 1), `z` sa « profondeur » (utilisée pour la taille et la vitesse de parallaxe), `p` une phase aléatoire pour désynchroniser le scintillement.

```js
for(var i=0;i<(innerWidth<700?110:220);i++)
  stars.push({x:Math.random(),y:Math.random(),z:Math.random()*.9+.1,p:Math.random()*6});
```

Moins d'étoiles sont créées sur petit écran (110 contre 220), pour rester léger sur mobile.

- **Le scintillement**, obtenu avec une fonction mathématique périodique :

```js
var a=.4+.6*Math.abs(Math.sin(T*s.z+s.p));
```

`Math.sin` oscille naturellement entre -1 et 1 dans le temps (`T`, qui avance à chaque image) ; `Math.abs` la ramène entre 0 et 1 ; le résultat sert d'opacité (`a`), qui varie donc en douceur et sans se répéter en même temps pour toutes les étoiles (grâce à `s.p`, différent pour chacune).

- **La parallaxe**, l'effet où les étoiles proches bougent plus vite que les lointaines quand la souris bouge ou qu'on scrolle :

```js
var px=((s.x*W-mx*s.z*30)%W+W)%W;
```

Le décalage horizontal dépend de `mx` (position de la souris) multiplié par `s.z` (la profondeur) : plus `z` est grand (étoile « proche »), plus elle se décale. Le `%W` (modulo) fait boucler les étoiles qui sortent d'un côté de l'écran pour réapparaître de l'autre.

- **Les étoiles filantes**, apparues au hasard avec une faible probabilité à chaque image :

```js
if(!shoot && !reduce && Math.random()<.004) shoot={x:...,y:...,l:1};
```

`Math.random()<.004` a environ 0,4 % de chance d'être vraie à chaque image, donc une étoile filante apparaît de temps en temps, jamais deux à la fois (`!shoot` vérifie qu'aucune n'est déjà en cours).

**La boucle d'animation :**

```js
function loop(){
  T+=.016;
  c.clearRect(0,0,W,H);
  // ... dessiner les étoiles, le nom, la traînée, etc.
  requestAnimationFrame(loop);
}
loop();
```

`requestAnimationFrame(loop)` rappelle la fonction `loop` juste avant le prochain rafraîchissement de l'écran (environ 60 fois par seconde). `clearRect` efface tout le canvas à chaque tour, puis tout est redessiné avec des positions légèrement différentes : c'est ce cycle « effacer / redessiner » qui crée le mouvement.

**Le fond selon l'heure réelle :**

```js
function setSky(h){
  var p;
  if(h>=5 && h<9) p=['#2a2358','#e27f6b'];       // aube
  else if(h>=9 && h<17) p=['#15285e','#3d63a8']; // jour
  else if(h>=17 && h<21) p=['#1c1548','#a1476e']; // crépuscule
  else p=['#070b22','#181440'];                   // nuit
  document.documentElement.style.setProperty('--a',p[0]);
  document.documentElement.style.setProperty('--b',p[1]);
}
setSky(new Date().getHours());
```

`new Date().getHours()` donne l'heure actuelle (0 à 23). Selon la tranche horaire, deux couleurs sont choisies et injectées dans les variables CSS `--a` et `--b`, utilisées par le dégradé de fond du `<body>`.

---

## 2. Le nom en points lumineux

**Emplacement :** fonction `sample()`.

**Principe, en 4 étapes :**

1. Un canvas **invisible** (créé uniquement en mémoire, jamais ajouté à la page) reçoit le texte « Elsa Borget » écrit avec `fillText`.
2. `getImageData` lit tous les pixels de ce canvas et renvoie leur transparence.
3. Le script parcourt ces pixels tous les 5 (`for(...;yy+=5)...xx+=5`, pour ne pas en garder trop) et, là où un pixel appartient à une lettre (`alpha>128`), crée une particule dont la **cible** (`tx`, `ty`) est cette position.
4. Chaque particule part d'une position aléatoire sur l'écran et **glisse progressivement** vers sa cible :

```js
t.x += (t.tx - t.x) * t.k;
```

Elle parcourt à chaque image une fraction (`t.k`, environ 2 %) de la distance qui lui reste à parcourir. Ce type de mouvement ralentit naturellement en approchant de la cible, ce qui donne un effet doux plutôt qu'un arrêt brutal.

Les particules sont aussi repoussées par la souris (même logique que la traînée, voir plus bas), ce qui crée l'effet de poussière dorée qui réagit au passage du curseur sur le nom.

---

## 3. Les apparitions en fondu

Déjà détaillé plus haut dans la conversation. En résumé : le CSS définit un état caché (`opacity:0`) et un état visible (`body.on ...{opacity:1}`), relié par une `transition`. Le JS ajoute la classe `on` au `<body>` avec un léger délai (`setTimeout(...,1200)`) après le chargement de la police, ce qui déclenche en cascade l'apparition du nom, du texte, des icônes, etc., chacun avec son propre délai de transition.

---

## 4. Les lettres révélées une à une

Déjà détaillé plus haut. En résumé : le JS découpe chaque titre `.tx` en un `<span>` par lettre, chacun avec un délai croissant (`i*40ms`). Un `IntersectionObserver` ajoute la classe `in` au titre quand il devient visible à l'écran (à 30 % visible), ce qui déclenche en CSS le passage de chaque lettre du flou/invisible au net/visible.

---

## 5. Les constellations de projets

Déjà détaillé plus haut. En résumé : un tableau `SH` contient les coordonnées de 6 points par projet. Une boucle construit le texte HTML de 5 `<line>` (une entre chaque paire de points consécutifs) et 6 `<circle>`, avec la longueur de chaque ligne calculée (`Math.hypot`) et stockée dans la variable CSS `--l`, utilisée pour l'effet de trait qui se dessine (`stroke-dasharray` / `stroke-dashoffset`). Ce texte est injecté dans le SVG vide du projet avec `innerHTML`.

---

## 6. Le trou de ver

Déjà détaillé plus haut. En résumé : au clic sur un projet, la fonction `wormhole()` place un cercle CSS (`clip-path:circle(...)`) à l'endroit exact du clic (variables `--x`/`--y`), le fait grandir jusqu'à couvrir l'écran, puis, une fois l'écran couvert (après 900 ms, la durée de la transition), remplit et affiche la page de détail du projet (titre, description, technos, lien GitHub, jeu éventuel).

---

## 7. Le bouton magnétique

**Emplacement :** bloc JS après le commentaire `// Bouton magnétique`.

```js
var m=document.querySelector('.mag');
addEventListener('mousemove',function(e){
  var r=m.getBoundingClientRect();
  var dx=e.clientX-(r.left+r.width/2), dy=e.clientY-(r.top+r.height/2);
  var d=Math.hypot(dx,dy);
  m.style.transform = d<130 ? 'translate('+dx*.3+'px,'+dy*.3+'px)' : '';
});
```

**Principe :** à chaque mouvement de souris, on calcule la distance (`d`) entre la souris et le centre du bouton (`getBoundingClientRect()` donne sa position et sa taille à l'écran). Si cette distance est inférieure à 130 px, le bouton se déplace de 30 % de l'écart (`dx*.3`, `dy*.3`) vers la souris, ce qui donne l'impression qu'il est « attiré » comme un aimant. Au-delà de 130 px, `transform` est vidé et le bouton reprend sa place normale. La `transition:transform .2s` du CSS rend ce mouvement fluide.

---

## 8. La fusée vers LinkedIn

Déjà détaillé plus haut. En résumé : à l'envoi du formulaire, le JS empêche le rechargement de page (`e.preventDefault()`), relance l'animation CSS `fly` en retirant puis remettant la classe `go` (avec l'astuce `void r.getBoundingClientRect()` pour forcer le navigateur à « voir » le changement), affiche un message, puis redirige vers LinkedIn après 1,8 s.

---

## 9. Le mini-jeu Snake

**Emplacement :** fonction `snake()`, appelée uniquement quand on ouvre le projet correspondant (via `x.g()` dans `wormhole`).

**La grille :** un canvas de 280×280 px, divisé en cases de 20 px, soit une grille de 14×14.

**L'état du jeu :**
```js
var s=[[7,7]], d=[1,0], f=[3,3], sc=0;
```
- `s` : le serpent, une liste de positions `[colonne, ligne]`. Il ne contient qu'une case au départ.
- `d` : la direction actuelle, `[1,0]` signifie « une case vers la droite ».
- `f` : la position de la nourriture.
- `sc` : le score.

**La boucle du jeu :**
```js
var iv=setInterval(function(){
  var h=[s[0][0]+d[0], s[0][1]+d[1]];   // nouvelle position de la tête
  if(h[0]<0 || h[1]<0 || h[0]>13 || h[1]>13 || s.some(p => p[0]==h[0]&&p[1]==h[1])){
    s=[[7,7]]; d=[1,0]; sc=0; return;   // collision : on recommence
  }
  s.unshift(h);                          // on ajoute la nouvelle tête
  if(h[0]==f[0] && h[1]==f[1]){          // la tête est sur la nourriture
    sc++; f=[Math.random()*14|0, Math.random()*14|0];
  } else s.pop();                        // sinon on enlève la queue
  // ... redessiner
}, 130);
```

`setInterval(fonction, 130)` répète l'action toutes les 130 ms : c'est le rythme du jeu. `s.some(...)` vérifie si une case du corps a la même position que la nouvelle tête (collision avec soi-même). `unshift` ajoute en tête de liste, `pop` retire en fin de liste : ensemble, ça fait « avancer » le serpent d'une case. Si la tête touche la nourriture, on ne fait pas `pop`, donc le serpent garde sa dernière case en plus : il grandit.

**Les contrôles :**
```js
function key(e){
  var m={ArrowUp:[0,-1],ArrowDown:[0,1],ArrowLeft:[-1,0],ArrowRight:[1,0]}[e.key];
  if(m && (m[0]+d[0] || m[1]+d[1])){ d=m; e.preventDefault() }
}
```
La condition `m[0]+d[0] || m[1]+d[1]` empêche le demi-tour immédiat : si la nouvelle direction est exactement l'opposée de l'actuelle, leur somme vaut 0 pour les deux axes, et le changement est refusé (sinon le serpent foncerait droit dans sa propre case suivante).

Le tactile est géré séparément (`touchstart`/`touchend`) en comparant la position de début et de fin du glissement pour déduire une direction.

**L'arrêt propre :**
```js
window.stopG = function(){ clearInterval(iv); removeEventListener('keydown', key) };
```
Cette fonction est appelée quand on quitte la page de détail (bouton « Revenir au ciel »), pour arrêter le `setInterval` et retirer l'écoute du clavier. Sans elle, le jeu continuerait de tourner en arrière-plan indéfiniment, même invisible.

---

## 10. Le jeu de mots mêlés

**Emplacement :** fonction `mots()`.

**La grille :** 8×8 lettres aléatoires (`A[Math.random()*26|0]`, un caractère pris au hasard dans l'alphabet), puis certaines lignes sont écrasées par les mots à trouver (`PHP`, `CSS`, `SQL`...), placés à une colonne de départ aléatoire.

**La sélection de deux cases :**
```js
b.onclick = function(){
  if(!a){ a=[i,j]; ... return }         // premier clic : on mémorise la case
  var s=Math.min(a[1],j), e=Math.max(a[1],j);
  var w = a[0]==i ? g[i].slice(s,e+1).join('') : '';  // texte entre les deux cases
  if(W.indexOf(w)>-1 || W.indexOf(w.split('').reverse().join(''))>-1){
    // le mot (ou son inverse) est dans la liste : on colore les cases
  }
  a=null;
};
```

Au premier clic, la case est mémorisée dans `a`. Au second clic, si les deux cases sont sur la même ligne (`a[0]==i`), on extrait les lettres entre elles avec `slice` et on les rassemble en un mot avec `join('')`. Ce mot est comparé à la liste `W`, dans les deux sens (normal et inversé, avec `.reverse()`), pour permettre de sélectionner un mot de droite à gauche.

---

## 11. Le système solaire des compétences

**Emplacement :** bloc JS après le commentaire `// Compétences en système solaire`.

```js
[['JavaScript',80],['PHP',75],['MySQL',70],['React',65],['Cybersécurité',50]].forEach(function(s,i){
  var r=(i+1)*17+8;
  var o=document.createElement('div'); o.className='orb';
  o.style.cssText='width:'+r+'%;height:'+r+'%;animation-duration:'+(14+i*7)+'s;animation-delay:-'+i*3+'s';
  var p=document.createElement('div'); p.className='pl';
  p.style.width=p.style.height=(6+s[1]*.12)+'px';
  o.appendChild(p);
  document.getElementById('sys').appendChild(o);
});
```

Chaque compétence `[nom, niveau]` génère une orbite (`.orb`, un cercle en pointillés qui tourne, défini en CSS avec `@keyframes spin`) et une planète (`.pl`) posée dessus. Trois valeurs dépendent de la position `i` dans la liste ou du niveau `s[1]` :
- **La taille de l'orbite** (`r`) grandit avec `i` : chaque compétence suivante a une orbite plus grande.
- **La vitesse de rotation** (`animation-duration`) augmente aussi avec `i`, pour que les orbites extérieures tournent plus lentement (comme de vraies planètes).
- **La taille de la planète** dépend du niveau de compétence : plus il est élevé, plus la planète est grosse.

`animation-delay` négatif fait démarrer l'animation comme si elle avait déjà commencé depuis un moment, pour que toutes les planètes ne soient pas alignées au chargement.

**Pour modifier vos compétences :** changer les valeurs dans le tableau `[['JavaScript',80], ...]` : le nom et un niveau sur 100.

---

## 12. Le son (bips)

**Emplacement :** bloc JS après le commentaire `// Son optionnel`, et fonction `blip()`.

```js
function blip(f){
  if(!on) return;
  var o=ac.createOscillator(), g=ac.createGain();
  o.frequency.value=f;
  g.gain.setValueAtTime(.05, ac.currentTime);
  g.gain.exponentialRampToValueAtTime(.001, ac.currentTime+.4);
  o.connect(g); g.connect(ac.destination);
  o.start(); o.stop(ac.currentTime+.4);
}
```

Utilise la **Web Audio API**, une fonctionnalité native du navigateur pour générer du son sans fichier audio. Un « oscillateur » (`createOscillator`) génère une note pure à la fréquence `f`. Un « gain » (`createGain`) contrôle son volume, qui démarre à 0,05 et redescend presque à 0 en 0,4 s (`exponentialRampToValueAtTime`), ce qui donne un bip court avec une extinction en douceur plutôt qu'une coupure nette.

`if(!on) return` empêche tout son tant que l'utilisateur n'a pas cliqué sur le bouton « Son » — les navigateurs interdisent de toute façon le son automatique sans interaction préalable.

Cette fonction est appelée à chaque survol d'un projet (`blip(300+n*150)`), avec une fréquence différente selon le projet, pour une petite note distincte à chacun.

---

## 13. Le sélecteur de ciel manuel

**Emplacement :** bouton `#sk` dans le HTML, bloc JS après `// Ciel manuel`.

```js
var modes=[['auto',null],['aube',7],['jour',12],['crépuscule',19],['nuit',23]], mi=0;
document.getElementById('sk').onclick=function(){
  mi=(mi+1)%5;
  var m=modes[mi];
  setSky(m[1]===null ? new Date().getHours() : m[1]);
  this.textContent='Ciel : '+m[0];
};
```

Chaque clic avance d'un cran dans la liste `modes`, en revenant à 0 après le dernier (`(mi+1)%5`, le modulo fait boucler). Selon le mode choisi, `setSky` est appelée soit avec l'heure réelle (mode « auto »), soit avec une heure forcée (7 pour l'aube, 12 pour le jour, etc.), pour prévisualiser le ciel à un autre moment de la journée.

---

## 14. La barre de progression de scroll

**Emplacement :** `<div id="bar">` dans le HTML, dernière ligne du JS.

```js
addEventListener('scroll', function(){
  document.getElementById('bar').style.width =
    scrollY/(document.body.scrollHeight-innerHeight)*100+'%';
}, {passive:true});
```

`scrollY` est la distance déjà parcourue depuis le haut de la page. `document.body.scrollHeight - innerHeight` est la distance totale qu'il est possible de parcourir (hauteur totale de la page moins la hauteur de l'écran). Le rapport des deux, en pourcentage, donne la largeur de la barre, qui se remplit donc progressivement à mesure qu'on scrolle. `{passive:true}` est une option qui indique au navigateur que cette fonction ne bloquera jamais le scroll, ce qui améliore la fluidité.

---

## 15. Le terminal caché

**Emplacement :** `<div id="term">` dans le HTML, fonction `openTerm()` et le bloc `addEventListener('keydown', ...)` final.

**Deux façons de le déclencher :**

```js
buf=(buf+e.key).slice(-4);
if(buf==='sudo') openTerm();
```

À chaque touche pressée, elle est ajoutée à `buf`, dont on ne garde que les 4 derniers caractères (`.slice(-4)`). Si cette suite devient exactement `sudo`, le terminal s'ouvre.

```js
var K=[38,38,40,40,37,39,37,39,66,65], ki=0;
...
ki = e.keyCode===K[ki] ? ki+1 : (e.keyCode===K[0]?1:0);
if(ki===K.length){ ki=0; openTerm() }
```

`K` est la liste des codes clavier du code Konami (↑↑↓↓←→←→BA). `ki` avance d'un cran à chaque bonne touche dans l'ordre ; à la première erreur, il retombe à 0 (ou à 1 si l'erreur est en fait le début d'une nouvelle tentative). Une fois les 10 touches réussies dans l'ordre, le terminal s'ouvre.

**L'effet machine à écrire :**

```js
var t=setInterval(function(){
  term.textContent += L[i++] || '';
  if(i>L.length) clearInterval(t);
}, 25);
```

Le texte `L` est ajouté **un caractère à la fois** toutes les 25 ms, ce qui simule un texte qui s'écrit en direct, comme dans un vrai terminal.

---

# Partie 2 — Ajouts

## 16. Font Awesome (bibliothèque d'icônes)

**Emplacement :** dans le `<head>`, après le lien vers Google Fonts.

```html
<link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.7.2/css/all.min.css">
```

**Rôle :** charge la bibliothèque d'icônes Font Awesome 6.7.2, utilisée pour les petites icônes de compétences et les boutons du CV (`<i class="fa-solid fa-...">`).

**Pour changer une icône :** remplacer le nom après `fa-solid` (ex. `fa-code`, `fa-database`, `fa-download`) par un autre nom trouvé sur [fontawesome.com/icons](https://fontawesome.com/icons) (version gratuite / Solid uniquement).

---

## 17. Photo de profil

**Emplacement :** dans `<section id="top">`, juste avant le `<h1 id="name">`.

```html
<div class="portrait">
  <img src="photo_portfolio.jpg" alt="Portrait d'Elsa Borget">
</div>
```

**Fichier requis :** `photo_portfolio.jpg` doit être dans le même dossier que le fichier HTML. Pour changer de photo, remplacer soit le fichier lui-même (en gardant le même nom), soit la valeur de `src`.

**CSS associé :**

```css
.portrait{
  width:150px;
  aspect-ratio:3/4;
  margin-bottom:1.5rem;
  border-radius:999px 999px 16px 16px;
  overflow:hidden;
  border:2px solid var(--glow);
  box-shadow:0 0 30px rgba(255,207,122,.35);
  opacity:0;
  transition:opacity 2s 2.5s;
}
body.on .portrait{ opacity:1; }
.portrait img{
  width:100%;
  height:100%;
  object-fit:cover;
  object-position:center top;
  display:block;
}
```

**Comportement :**
- Cadre en forme d'arche (`border-radius` différent en haut et en bas), contour doré, légère lueur.
- Apparaît en fondu 2,5 s après le chargement, en même temps que le reste du bloc d'accueil (même mécanique que le `<h1>`, voir la partie « Le nom qui apparaît » discutée plus haut dans la conversation).
- `object-fit:cover` garde les proportions de la photo sans la déformer, en la recadrant si besoin.

**Comportement responsive :**

```css
@media(min-width:1100px){
  #top{ position:relative; }
  .portrait{
    position:absolute;
    right:7vw;
    top:50%;
    transform:translateY(-50%);
    width:clamp(200px,20vw,300px);
    margin-bottom:0;
  }
}
```

À partir de 1100 px de large, la photo passe en position absolue, à droite de l'écran, centrée verticalement. En dessous de cette largeur, elle reste au-dessus du nom, dans le flux normal de la page.

**Pour ajuster :**
- Taille sur mobile : modifier `width:150px`.
- Taille sur grand écran : modifier `clamp(200px,20vw,300px)`.
- Position sur grand écran : modifier `right:7vw`.

---

## 18. Couleur dorée du nom et du texte de présentation

**Emplacement :** dans le `<style>`, à la fin, remplaçant les règles `h1` et `.sub` d'origine.

```css
h1{
  background:linear-gradient(120deg,#f3ecd8 0%,#ffcf7a 45%,#e0a94a 100%);
  -webkit-background-clip:text;
  background-clip:text;
  -webkit-text-fill-color:transparent;
  color:transparent;
  padding-bottom:.1em;
}
```

**Principe :** au lieu d'une couleur unie, le texte du `<h1>` reçoit un **dégradé** (crème → doré clair → doré foncé), puis ce dégradé est « découpé » à la forme des lettres grâce à `background-clip:text`. `color:transparent` rend le texte lui-même invisible pour ne laisser voir que le dégradé à travers.

```css
.sub{
  color:var(--glow);
  text-shadow:0 0 18px rgba(255,207,122,.35);
}
```

Le texte de présentation (« Développeuse fullstack... ») passe en doré uni, avec une légère lueur autour (`text-shadow`, même principe que le `drop-shadow` vu sur les étoiles des constellations).

```css
h2.tx span{ color:var(--glow); }
```

Les titres de section (« Projets », « Mes planètes », etc.) passent aussi en doré, lettre par lettre puisqu'ils sont déjà découpés en `<span>` par le JS existant.

**Pour changer les couleurs :** remplacer les codes hexadécimaux (`#f3ecd8`, `#ffcf7a`, `#e0a94a`) par d'autres teintes, ou ajuster `var(--glow)` directement dans `:root` pour un changement global.

---

## 19. Icônes de compétences (chips)

**Emplacement :** dans `<section id="top">`, juste après `<p class="sub">`.

```html
<div class="chips">
  <span><i class="fa-solid fa-code"></i> Fullstack</span>
  <span><i class="fa-solid fa-database"></i> MySQL</span>
  <span><i class="fa-solid fa-shield-halved"></i> Cybersécurité</span>
</div>
```

**CSS associé :**

```css
.chips{
  display:flex;
  flex-wrap:wrap;
  gap:.6rem;
  margin-top:1.2rem;
  opacity:0;
  transition:opacity 1.5s 3.4s;
}
body.on .chips{ opacity:1; }
.chips span{
  border:1px solid rgba(255,207,122,.5);
  color:var(--glow);
  border-radius:99px;
  padding:.3rem .9rem;
  font-size:.85rem;
}
.chips i{ margin-right:.35rem; }
```

**Comportement :** trois petites étiquettes arrondies avec icône, qui apparaissent en fondu après le texte de présentation (délai de 3,4 s, le plus tardif du bloc d'accueil, pour un effet d'arrivée échelonnée).

**Pour ajouter ou modifier une compétence :** dupliquer une ligne `<span>`, changer le nom de l'icône (`fa-code`, etc.) et le texte affiché.

---

## 20. Section CV

**Emplacement :** nouvelle `<section id="cv">`, insérée juste avant `<section id="contact">`.

```html
<section id="cv">
  <h2 class="tx">Mon CV</h2>
  <p class="cvtxt">Retrouve mon parcours, mes compétences et mes expériences.</p>
  <div class="cvbtns">
    <a class="cvbtn" href="cv_elsa_borget.pdf" download="CV_Elsa_Borget.pdf">
      <i class="fa-solid fa-download"></i> Télécharger mon CV
    </a>
    <a class="cvlink" href="cv_elsa_borget.pdf" target="_blank" rel="noopener">
      <i class="fa-solid fa-eye"></i> Voir en ligne
    </a>
  </div>
</section>
```

**Fichier requis :** `cv_elsa_borget.pdf` doit être dans le même dossier que le fichier HTML.

**Fonctionnement des deux liens :**
- Le premier (`download="CV_Elsa_Borget.pdf"`) force le **téléchargement** du fichier au clic, sous le nom indiqué, plutôt que de l'ouvrir dans le navigateur.
- Le second (`target="_blank"`) **ouvre** le PDF dans un nouvel onglet, pour une lecture directe sans téléchargement. `rel="noopener"` est une précaution de sécurité standard pour les liens qui s'ouvrent dans un nouvel onglet.

**CSS associé :**

```css
.cvtxt{ max-width:44ch; color:var(--mute); }
.cvbtns{
  display:flex;
  flex-wrap:wrap;
  gap:1rem;
  align-items:center;
  margin-top:1rem;
}
.cvbtn{
  background:var(--glow);
  color:#1a1230;
  border-radius:99px;
  padding:.9rem 2rem;
  font:600 1rem "Instrument Sans",sans-serif;
  text-decoration:none;
  transition:transform .2s, box-shadow .2s;
}
.cvbtn:hover{
  transform:translateY(-2px);
  box-shadow:0 0 20px rgba(255,207,122,.5);
}
.cvlink{ color:var(--glow); text-decoration:none; }
.cvlink:hover{ text-decoration:underline; }
.cvbtn:focus-visible,.cvlink:focus-visible{
  outline:2px solid var(--star);
  outline-offset:4px;
}
```

**Point d'attention technique :** le bouton de téléchargement utilise volontairement la classe `.cvbtn` et **non** `.mag` (la classe du bouton magnétique de la section contact). Le script du bouton magnétique cible le premier élément `.mag` trouvé sur la page avec `document.querySelector('.mag')` ; réutiliser cette classe aurait fait « glisser » le bouton du CV vers la souris au lieu du bouton LinkedIn.

**Pour changer le fichier CV :** remplacer `cv_elsa_borget.pdf` par le nom réel du fichier, dans les deux `href`.

---

## Récapitulatif des fichiers à fournir

| Fichier | Rôle | Emplacement attendu |
|---|---|---|
| `photo_portfolio.jpg` | Photo de profil | même dossier que le HTML |
| `cv_elsa_borget.pdf` | CV téléchargeable | même dossier que le HTML |

---

## Principe commun à tous ces ajouts

Comme le reste du site, chaque apparition (photo, chips) suit le principe déjà en place : **un état caché en CSS (`opacity:0`), un état visible (`body.on ...{opacity:1}`), et une transition qui fait le passage en douceur**. Le déclencheur (`body.on`) existe déjà dans le JS d'origine et n'a pas eu besoin d'être modifié : les nouveaux éléments profitent simplement du même interrupteur que le `<h1>`.
