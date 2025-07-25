# Questions techniques / orientées développeur·euses (a11y dev)

- **Qu’est-ce que le DOM d’accessibilité (*accessibility tree*) ?**

Le DOM d’accessibilité, ou *Accessibility Tree*, est une représentation parallèle du DOM standard, utilisée par les technologies d’assistance (comme les lecteurs d’écran) pour comprendre et interagir avec le contenu d'une page web. C’est ce tree, et non le DOM visuel, qui est lu par les lecteurs d’écran.

Il contient uniquement les informations pertinentes pour l’accessibilité, extraites ou dérivées du DOM principal.

Chaque élément de ce DOM inclut des propriétés comme :

-- Nom accessible (*accessible name*)

-- Rôle (*button, heading, link, etc.*)

-- État (activé, désactivé, coché, etc.)

-- Valeur (dans le cas des champs)

-- Relations (par exemple : étiquettes liées)

Attention, tous les éléments HTML ne sont pas automatiquement inclus dans l’*accessibility tree*.

Par exemple, un élément avec ``aria-hidden="true"`` ou ``display: none`` sera exclu. Une ``div`` sans rôle ni texte ni interaction ne sera généralement pas exposée.

-----

- **Quelle est la différence entre une ``div`` et un ``button`` en termes d’accessibilité ?**

La balise ``div`` est un élément générique, sans sémantique, fonction ni interactivité native. Il n’est pas focusable par défaut.

La balise ``button`` a une sémantique claire : c’est un élément cliquable. Elle est nativement focusable au clavier (via la touche ``Tab``) et déclenche des événements clavier (``Enter``, ``Espace``) automatiquement. Elle est annoncée comme un "bouton" par les lecteurs d’écran.

Il ne faut donc pas utiliser une ``div`` à la place d'un ``button`` !

-----

- **À quoi sert l’attribut ``aria-label`` ? Quand le préfère-t-on à ``aria-labelledby`` ?**

``aria-label`` est un attribut qui permet de donner un nom accessible personnalisé à un élément. Il ne repose pas sur un contenu visible du DOM. Il est utile quand l'élément ne contient pas de texte ou que le texte visible n'est pas suffisant pour les technologies d’assistance.

```html

<!-- Dans cet exemple, le bouton n’a aucun texte visible, donc aria-label fournit un nom lisible par les lecteurs d’écran. -->
<button aria-label="Fermer">
  <svg>...</svg>
</button>

```

``aria-labelledby`` est un attribut qui permet d’associer un ou plusieurs éléments existants du DOM comme nom accessible. Il est préféré quand le contenu visible peut (ou doit) servir de nom à l’élément.

```html

<!-- Dans cet exemple, le lecteur d’écran annoncera « section : Paramètres ». -->
<h2 id="section-title">Paramètres</h2>
<section aria-labelledby="section-title">
  ...
</section>

```

Attention à ne jamais combiner ``aria-label`` et ``aria-labelledby`` sur le même élément. Si les deux sont présents, ``aria-labelledby`` l’emporte.

Si le texte est visuellement visible et pertinent, il est préférable d'utiliser plutôt ``aria-labelledby``.
``aria-label`` sera plutôt utilisé pour des contenus visuellement ambigus ou absents.

-----

- **Que permet l’attribut ``role`` en HTML ?**

L’attribut ``role`` permet de définir explicitement le rôle d’un élément HTML, c’est-à-dire la fonction qu’il joue dans l’interface (ex. : bouton, dialogue, alerte, etc.).

Cet attribut fait partie de la spécification ``WAI-ARIA`` (*Web Accessibility Initiative – Accessible Rich Internet Applications*), qui permet de rendre accessibles les composants riches ou personnalisés aux technologies d’assistance (lecteurs d’écran, etc.).

Cela permet de fournir une sémantique claire pour les éléments non sémantiques (``div``, ``span``, ``li``, etc.) et de permettre aux technologies d’assistance de comprendre la nature d’un élément, même s’il est "*custom*".

Il permet aussi de créer des composants dynamiques accessibles (onglets, *sliders*, modales, etc.).

Attention à ne pas ajouter de role inutilement sur des éléments qui ont déjà un rôle implicite. Par exemple : pas besoin de ``role="button"`` sur une balise ``<button>``. Et garder en tête que le rôle seul ne rend pas un élément interactif : il faut souvent ajouter ``tabindex``, ``keydown``, etc.

-----

- **Quelle est la bonne pratique pour les boutons "Lire plus" ?**

Les boutons ou liens "Lire plus" sont souvent utilisés pour accéder à du contenu complémentaire (ex. : un article, une fiche produit, etc.).

Cependant, si plusieurs boutons "Lire plus" sont présents sur une page sans contexte explicite, les technologies d’assistance ne peuvent pas les distinguer, car elles n'annoncent que "Lire plus", sans indication sur ce que l’on va lire. L'utilisateur ne sait pas s'il va lire un article sur du HTML, du fromage, ou un vélo.

La bonne pratique veut que l'on remplace complètement le texte du lien. Cela améliore l'expérience des utilisateurs de lecteurs d’écran, en plus du SEO.

```html

<a href="/article/html-accessible">Lire l’article sur l’accessibilité HTML</a>

```

-----

- **Pourquoi faut-il éviter les ``tabindex`` positifs ?**

L'attribut global ``tabindex`` permet aux dev de rendre les éléments HTML focalisables, d'autoriser ou d'empêcher qu'ils soient focalisables (généralement avec la touche ``Tab``, d'où le nom) et de déterminer leur ordre relatif pour la navigation séquentielle du focus.

Il accepte un entier comme valeur, avec des résultats différents selon la valeur :

-- Une valeur négative (la valeur négative exacte importe peu, généralement ``tabindex="-1"``) signifie que l'élément n'est pas accessible par la navigation au clavier.

-- ``tabindex="0"`` signifie que l'élément doit être accessible par la navigation au clavier, après toute valeur tabindex positive. L'ordre de navigation de ces éléments est défini par leur ordre dans le document source.

-- Une valeur positive signifie que l'élément doit être accessible par la navigation au clavier, son ordre étant défini par la valeur du nombre. Ainsi, ``tabindex="4"`` est accessible avant ``tabindex="5"`` et ``tabindex="0"``, mais après ``tabindex="3"``. Si plusieurs éléments partagent la même valeur tabindex positive, leur ordre relatif suit leur position dans le document source.

Il est recommandé d'utiliser uniquement les valeurs 0 et -1 comme tabindex. Une valeur positive complique la navigation et l'utilisation du contenu des pages par les utilisateurs utilisant le clavier ou des technologies d'assistance. Il faut rédiger plutôt le code en respectant un ordre logique pour les éléments.

-----

- **Quelle est la différence entre ``aria-hidden="true"`` et ``display: none`` ?**

Les deux servent à masquer un contenu, mais n’ont pas du tout le même effet, ni le même public cible.

``aria-hidden="true"`` est utilisé pour cacher un contenu (et tous ses éléments enfants) non interactif du DOM d’accessibilité (*accessibility tree*). L’élément reste visible à l’écran. Il est utile pour ignorer du contenu non, purement décoratif ou dupliqué pour les utilisateurs d’assistance.

*Note : Attention, ``aria-hidden="true"`` ne doit pas être utilisé sur des éléments qui peuvent être focus.*

``display: none`` cache l’élément visuellement et dans le DOM d’accessibilité. L’élément est donc invisible à toutes et tous, y compris aux lecteurs d’écran et à la navigation clavier. (L’élément est totalement ignoré par les technologies d’assistance.)

```html

<div style="display: none">Je suis invisible pour tout le monde</div>
<div aria-hidden="true">Je suis visible mais ignoré par le lecteur d’écran</div>
<button aria-hidden="true">Cliquez ici</button> <!-- Mauvais car ce bouton reste focusable au clavier, mais il est ignoré par le lecteur d’écran. -->

```

-----

- **Comment gérer les focus lors d’une ouverture de modal ?**

Lorsqu’une modale s’ouvre, il faut gérer le focus pour garantir une expérience accessible, notamment pour les utilisateurs naviguant au clavier (``Tab``, ``Shift+Tab``) et les utilisateurs de lecteurs d’écran.

Pour (bien) gérer les focus lors d’une ouverture de modal, il faut donner le focus à un élément pertinent (le premier élément interactif généralement) dans la modale dès son ouverture.

```javascript

document.querySelector('.modal input')?.focus();

```

Il faut faire attention au *focus trap*. Pendant que la modale est ouverte, le focus ne doit pas sortir de celle-ci (ni aller derrière). On doit intercepter ``Tab`` et ``Shift+Tab`` pour boucler le focus sur les éléments interactifs de la modale.

```javascript

const trapFocus = (modalElement) => {
  const focusableSelectors = 'a, button, input, textarea, select, [tabindex]:not([tabindex="-1"])';
  const focusableElements = [...modalElement.querySelectorAll(focusableSelectors)];
  const first = focusableElements[0];
  const last = focusableElements[focusableElements.length - 1];

  modalElement.addEventListener('keydown', (e) => {
    if (e.key === 'Tab') {
      if (e.shiftKey && document.activeElement === first) {
        e.preventDefault();
        last.focus();
      } else if (!e.shiftKey && document.activeElement === last) {
        e.preventDefault();
        first.focus();
      }
    }
  });
};

```

Lorsque la modale se ferme, il faut restaurer le focus sur l’élément qui l’a ouverte.

```javascript

const previouslyFocused = document.activeElement;

closeModal();
previouslyFocused?.focus();

```

Il faut également ajouter les bons rôles et attributs ARIA à la modale pour une bonne expérience utilisateur.

```html

<!-- 
 
role="dialog" : indique qu’il s’agit d’une boîte de dialogue

aria-modal="true" : précise que l’utilisateur ne peut interagir qu’avec la modale

aria-labelledby : relie le titre de la modale pour donner un nom explicite (ici "Connexion")

 -->
<div role="dialog" aria-modal="true" aria-labelledby="modal-title" class="modal">
  <h2 id="modal-title">Connexion</h2>
  <form>…</form>
</div>

```

-----

- **Comment rendre un composant SPA accessible au clavier ?**

Dans une *Single Page Application* (SPA), l’interaction avec les composants doit être entièrement accessible au clavier, car l’utilisateur peut naviguer sans rechargement de page, mais toujours avec des attentes d’accessibilité similaires à une navigation classique.

Tous les éléments interactifs doivent donc être accessibles au clavier. Pour ça, il faut :

-- Utiliser des éléments HTML natifs quand c’est possible (``<button>``, ``<a>``, ``<input>``) car ils sont accessibles par défaut.

-- Gérer les événements clavier manuellement pour les éléments non natifs (composant custom).

-- Utiliser ``tabindex`` à bon escient. (Éviter les valeurs positives de type ``tabindex="1"``, etc.)

-- Gérer le focus lors de changements de vues (via *React Router*, *Vue Router* par exemple). Le focus doit être automatiquement déplacé sur un élément logique.

```javascript

// Le focus se fera sur le titre principal (de niveau 1)
document.querySelector('h1')?.focus();

```

-----

- **Quels attributs ARIA peuvent rendre un composant "accordion" accessible ?**

Pour rendre un composant accordéon accessible, il est essentiel d’utiliser les attributs ARIA de manière appropriée afin que les utilisateurs de lecteurs d’écran ou de navigation clavier puissent comprendre la structure, les états d’ouverture et interagir correctement avec les sections.

Chaque bouton qui déclenche l’ouverture ou la fermeture d’une section doit utiliser l’attribut aria-expanded, qui indique dynamiquement si la section est ouverte (``true``) ou fermée (``false``). On lui ajoute également un attribut ``aria-controls``, qui référence l’ID de la section qu’il contrôle, permettant ainsi d’établir une relation claire entre le déclencheur et le contenu affiché.

De son côté, la section de contenu associée doit avoir un ``role="region"``, ce qui la rend identifiable comme une région importante pour la navigation. On lui ajoute aussi ``aria-labelledby``, qui fait référence à l’ID du bouton qui en est le titre, afin que le lecteur d’écran annonce ce contenu dans son contexte.

Enfin, pour masquer ou afficher dynamiquement le contenu, il est courant d’utiliser l’attribut ``hidden`` ou ``aria-hidden="true"`` (selon les besoins), mais attention : ce contenu doit être retiré du focus si non visible, et il ne faut jamais masquer un élément focusable avec ``aria-hidden``, au risque de rendre l'expérience incohérente pour les utilisateurs de technologies d’assistance.

En complément de ces attributs, le bouton doit être utilisable au clavier : il doit pouvoir recevoir le focus (par défaut c’est le cas d’un ``<button>``) et être activé via les touches ``Entrée`` et ``Espace``. On évite d’utiliser un élément neutre comme une ``<div>`` pour jouer le rôle de bouton.

Grâce à cette combinaison d’attributs (``aria-expanded``, ``aria-controls``, ``aria-labelledby``, ``role="region"``, etc.), on garantit que l’accordéon est non seulement fonctionnel visuellement, mais aussi compréhensible et navigable pour tous les utilisateurs, quelles que soient leurs capacités.

-----

- **Comment structurer correctement une page HTML pour un bon usage avec lecteur d’écran ?**

Pour qu’une page soit bien interprétée par un lecteur d’écran, il est essentiel de respecter une structure sémantique claire et logique en HTML. Cela signifie utiliser des balises HTML selon leur signification, pas seulement leur apparence (qui peut de toute façon être modifiée grâce au CSS). Le cœur de cette bonne structure repose sur trois piliers : la hiérarchie des titres, la navigation cohérente, et les rôles sémantiques explicites.

La page doit contenir un seul élément ``<h1>`` qui correspond au titre principal de la page. Ensuite, les titres doivent suivre une hiérarchie logique (``<h2>`` pour les sections principales, ``<h3>`` pour les sous-sections, etc.), sans sauter de niveau. Cette hiérarchie permet aux utilisateurs de lecteur d’écran de naviguer facilement par titres.

Il est également crucial d’utiliser les landmarks HTML5 comme ``<header>``, ``<main>``, ``<nav>``, ``<footer>``, ``<aside>``, et ``<section>``. Ces éléments fournissent des repères de navigation aux technologies d’assistance. Le contenu principal de la page doit être encapsulé dans un ``<main>``, et le menu de navigation dans une ``<nav>``. Cela permet, par exemple, à un utilisateur de sauter directement au contenu principal sans avoir à écouter tout l’en-tête ou les menus à chaque fois.

Tous les boutons, liens, champs de formulaire et autres composants interactifs doivent être clairement identifiés grâce à leurs balises natives (``<button>``, ``<a>``, ``<input>``, etc.) ou, en cas d’usage personnalisé, par des attributs ARIA comme ``role``, ``aria-label``, ``aria-labelledby`` ou ``aria-describedby``. Il est aussi important de ne pas masquer du contenu utile au lecteur d’écran avec des styles ou des attributs comme ``aria-hidden="true"`` sans raison.

Enfin, l’ordre de tabulation doit être logique et naturel. Les éléments interactifs doivent apparaître dans l’ordre de lecture du document, sans avoir recours à des tabindex positifs. Cela garantit que la navigation au clavier et la lecture via synthèse vocale soient fluides.

-----

- **Quels éléments HTML sont naturellement accessibles ?**

Certains éléments HTML sont dits "nativement accessibles", car ils possèdent déjà un rôle sémantique, une prise en charge du focus et des comportements attendus compatibles avec les technologies d’assistance comme les lecteurs d’écran ou la navigation clavier. Ces éléments n’ont donc pas besoin d’attributs ARIA ou de scripts supplémentaires pour être compris.

Parmi eux, on retrouve les titres (``<h1>`` à ``<h6>``), qui permettent de structurer le contenu de manière hiérarchique et sont utilisés par les lecteurs d’écran pour proposer une navigation rapide. Les liens (``<a href="...">``), les boutons (``<button>``), les champs de formulaire (``<input>``, ``<textarea>``, ``<select>``, etc.) sont également accessibles par défaut : ils peuvent recevoir le focus, être activés au clavier (Entrée ou Espace), et annoncent automatiquement leur rôle et leur état (ex : coché, sélectionné, invalide...).

De plus, les listes (``<ul>``, ``<ol>``, ``<li>``), les tableaux (``<table>``, ``<th>``, ``<caption>``), les balises de navigation (``<nav>``, ``<main>``, ``<aside>``, ``<header>``, ``<footer>``, etc.) et les éléments de structure de contenu (``<section>``, ``<article>``, ``<figure>``, etc.) ont également une signification sémantique directement exploitable par les lecteurs d’écran.

-----

- **Pourquoi faut-il préférer les balises sémantiques aux éléments génériques avec ARIA ?**

L’intérêt d’utiliser ces éléments natifs (balises sémantiques) plutôt que des éléments génériques comme ``<div>`` ou ``<span>`` avec une configuration ARIA (``role="button"``, ``role="navigation"``, etc.), est d’offrir une interopérabilité maximale, c’est-à-dire la capacité de plusieurs systèmes (navigateurs, lecteurs d’écran, technologies d’assistance) à fonctionner ensemble correctement, sans conflit, tout en améliorant la maintenabilité et la fiabilité du code.

En utilisant les balises appropriées, on bénéficie automatiquement du comportement attendu (focus, interaction clavier, rôle, état…), ce qui évite de devoir recréer manuellement ces fonctionnalités avec JavaScript et des attributs ARIA. Cela réduit donc le risque d’erreurs, rend le code plus maintenable, plus compréhensible, et améliore l’accessibilité par défaut.

-----

- **Comment tester le focus d’un site avec le clavier uniquement ?**

Pour tester le focus clavier d’un site, on utilise principalement la touche ``Tab`` pour parcourir tous les éléments interactifs d'une page (liens, boutons, champs de formulaire, etc.) dans l'ordre défini par le DOM. Chaque pression sur Tab déplace le focus visible vers l’élément suivant, tandis que ``Shift + Tab`` permet de revenir en arrière. Ce test permet de vérifier que tous les éléments importants sont accessibles au clavier, que le focus est visible (souvent avec un contour ou un changement de style), et que l’ordre de navigation est logique et cohérent.

-----

- **Qu’est-ce que ``aria-live`` et dans quel cas l’utiliser ?**

L'attribut ``aria-live`` est utilisé pour indiquer qu’une zone de contenu va être mise à jour dynamiquement, sans que la page entière ne soit rechargée. Il informe les technologies d’assistance, comme les lecteurs d’écran, que les changements dans cette région doivent être annoncés automatiquement à l’utilisateur. Cet attribut est particulièrement utile dans les cas où du contenu important s’actualise en temps réel, par exemple pour afficher des notifications, des messages d’erreur, des mises à jour de statut, ou tout autre contenu dynamique qui nécessite l’attention immédiate de l’utilisateur sans qu’il ait à naviguer manuellement jusqu’à cette zone.

-----

- **Comment rendre un tableau accessible ?**

Pour rendre un tableau accessible, il faut structurer correctement son code HTML en utilisant les balises sémantiques dédiées comme ``<table>``, ``<thead>``, ``<tbody>``, ``<tfoot>``, ``<tr>``, ``<th>`` et ``<td>``.

Les cellules d’en-tête (``<th>``) doivent être bien définies et associées aux données correspondantes via les attributs scope (par exemple ``scope="col"`` pour une colonne, ``scope="row"`` pour une ligne) afin que les lecteurs d’écran puissent comprendre la relation entre les en-têtes et les cellules de données.

Il est également recommandé d’ajouter un titre au tableau avec une balise ``<caption>`` pour fournir un contexte clair.

Enfin, il faut éviter de fusionner trop de cellules (attributs ``colspan`` ou ``rowspan``) qui peuvent compliquer la lecture, et s’assurer que le tableau reste clair et bien organisé pour tous les utilisateurs, y compris ceux qui utilisent des technologies d’assistance.

-----

- **Quelles erreurs courantes en Javascript nuisent à l’accessibilité ?**

Un bon développement Javascript accessible implique de gérer correctement le focus, de fournir des notifications adaptées aux technologies d’assistance, de garantir l’accès clavier complet, et d’utiliser les attributs ARIA de façon cohérente et justifiée.

**Gestion incorrecte du focus** : ne pas gérer ou déplacer le focus correctement lors des interactions dynamiques (modales, dialogues, menus déroulants), ce qui peut perdre ou bloquer l’utilisateur au clavier ou avec un lecteur d’écran.

**Modification du contenu sans informer les technologies d’assistance** : mettre à jour du contenu dynamique sans utiliser des attributs comme ``aria-live``, ce qui empêche les lecteurs d’écran d’annoncer les changements.

**Événements uniquement sur la souris** : ne pas gérer les interactions clavier (ex : activation d’un bouton uniquement via un ``onclick`` sans équivalent ``onkeydown`` ou gestion du ``Enter/Space``), excluant ainsi les utilisateurs qui naviguent au clavier.

**Utilisation abusive ou incorrecte des rôles ARIA** : surcharger les éléments natifs avec des rôles inadaptés ou contradictoires, ou oublier de mettre à jour les états ARIA (``aria-expanded``, ``aria-checked``, etc.) lors des changements d’état.

**Création d’éléments interactifs non focalisables** : par exemple, des boutons ou liens personnalisés sans attributs tabindex ou sans gestion du clavier, rendant ces éléments inaccessibles aux utilisateurs de clavier.

**Animations ou effets visuels excessifs ou non désactivables** : qui peuvent provoquer des troubles chez certains utilisateurs (migraines, crises d’épilepsie).

-----

- **Quels outils permettent de tester l’accessibilité ?**

Pour tester l’accessibilité d’un site ou d’une application, plusieurs outils sont disponibles, parmi lesquels :

Les extensions de navigateur comme axe ``DevTools``, ``Lighthouse`` (intégré dans Chrome), ``WAVE``, ou ``Accessibility Insights``. Ces outils analysent automatiquement les pages web pour détecter des problèmes d’accessibilité courants et fournir des rapports détaillés.

Les lecteurs d’écran comme ``NVDA`` (Windows), ``VoiceOver`` (Mac/iOS), ou ``JAWS`` qui permettent de tester l’expérience utilisateur pour les personnes malvoyantes ou aveugles.

Les validateurs en ligne tels que le ``Validateur RGAA officiel``, ou des outils basés sur les WCAG comme ``AChecker``.

Les outils de simulation permettant de simuler différents types de déficiences visuelles, auditives ou motrices, afin d’évaluer la lisibilité et l’usage.

Les tests manuels essentiels pour compléter les tests automatiques, en vérifiant notamment la navigation au clavier, la compréhension des contenus, ou la présence de descriptions alternatives.

-----

- **Comment rendre un formulaire accessible ?**

Pour rendre un formulaire accessible, il est essentiel d’associer chaque champ à un label clair et explicite, afin que les technologies d’assistance puissent identifier facilement le rôle de chaque ``input``. Il faut aussi fournir des indications précises sur les données attendues, comme le type (texte, nombre, date), les formats acceptés, ou les contraintes (taille minimale/maximale), souvent via des attributs HTML (``type``, ``min``, ``max``) ou des ``placeholders`` bien utilisés, tout en évitant de s’appuyer uniquement sur ces derniers pour transmettre l’information.

En cas d’erreur lors de la saisie, il faut avertir l’utilisateur non seulement par des signaux visuels (couleur, icônes) mais aussi par des messages textuels clairs et accessibles, pour que tous les utilisateurs, y compris ceux utilisant un lecteur d’écran, comprennent la nature du problème et comment le corriger. Enfin, la navigation au clavier doit être fluide, avec un ordre logique des champs, et les erreurs doivent être facilement repérables lors du parcours au clavier.

-----

- **Comment gérer le focus dans les composants dynamiques (tabs, accordéons, modales) ?**

Pour gérer le focus dans les composants dynamiques comme les tabs, accordéons ou modales, il faut s’assurer que le focus se déplace de manière logique et prévisible pour les utilisateurs, notamment ceux qui naviguent au clavier ou utilisent des technologies d’assistance.

Quand un composant dynamique s’ouvre ou devient actif (par exemple une modale qui s’affiche), le focus doit être automatiquement déplacé vers le premier élément interactif de ce composant, comme un bouton de fermeture ou un champ de saisie, afin que l’utilisateur sache où commencer son interaction.

Dans le cas d’un accordéon ou d’onglets (tabs), le focus doit se déplacer entre les titres ou les contrôles qui déclenchent l’ouverture ou la fermeture des panneaux, avec une navigation au clavier cohérente (par exemple, flèches pour naviguer entre les onglets, tabulation pour accéder au contenu).

Enfin, lors de la fermeture d’une modale ou d’un panneau, il est important de renvoyer le focus vers l’élément qui a déclenché son ouverture, pour éviter que l’utilisateur ne perde son repère dans la page.

-----

- **Peut-on faire un site accessible en utilisant uniquement React / Vue ?**

Oui, il est tout à fait possible de faire un site accessible en utilisant uniquement React ou Vue, mais cela demande une attention particulière. Ces frameworks facilitent la création d’interfaces dynamiques, mais l’accessibilité ne dépend pas uniquement de la technologie utilisée, elle repose surtout sur la manière dont le développeur structure le code HTML, utilise les attributs ARIA, gère le focus, les interactions clavier, et respecte les bonnes pratiques d’accessibilité.

React et Vue permettent d’intégrer facilement des rôles ARIA, des labels, et de contrôler le DOM de manière fine, ce qui est un avantage. Cependant, si on génère des composants non sémantiques sans se soucier de l’accessibilité, ou si on oublie de gérer correctement le focus ou les interactions clavier, le site sera difficilement accessible.

En résumé, React et Vue ne garantissent pas l’accessibilité par défaut, mais ils peuvent être utilisés pour créer des sites accessibles à condition de bien appliquer les bonnes pratiques et de tester régulièrement l’accessibilité.

-----

- **Quelle est la différence entre label explicite et implicite ?**

La différence entre un label explicite et un label implicite réside dans la manière dont le label est associé à un champ de formulaire.

Un label explicite est un élément ``<label>`` qui est directement lié à un champ de formulaire grâce à l’attribut for qui référence l’id de ce champ. Par exemple :

```html

<label for="email">Adresse email</label>
<input type="email" id="email" />

```

Ici, le label est explicitement associé à l’input via l’attribut ``for``.

Un label implicite, quant à lui, consiste à envelopper directement l’élément de formulaire à l’intérieur de la balise ``<label>``, sans utiliser l’attribut ``for``. Par exemple :

```html

<label>
  Adresse email
  <input type="email" />
</label>

```

Dans ce cas, le label est implicitement lié à l’input parce que l’input est contenu à l’intérieur du ``<label>``.

Les deux méthodes sont valides et permettent aux technologies d’assistance de comprendre quel texte sert de label pour quel champ. L’important est qu’un champ ait toujours un label associé, explicite ou implicite.

-----

- **Comment rendre un composant *drag and drop* accessible ?**

Pour rendre un composant *drag and drop* accessible, il faut permettre à tous les utilisateurs, y compris ceux qui utilisent uniquement le clavier ou des technologies d’assistance, de manipuler les éléments déplacés. Voici les bonnes pratiques principales :

-- **Assurer la navigation clavier complète** : les éléments déplaçables doivent pouvoir être sélectionnés, déplacés, et déposés via le clavier (par exemple avec les touches ``Tab`` pour la navigation, et des combinaisons comme ``Espace`` ou ``Entrée`` pour saisir et lâcher un élément, ou encore les flèches pour le déplacer).

-- **Utiliser les rôles et attributs ARIA adaptés** : par exemple, ``role="listbox"`` pour la zone contenant les éléments, ``role="option"`` pour chaque élément déplaçable, et des attributs comme ``aria-grabbed="true|false"`` pour indiquer si un élément est actuellement saisi.

-- **Informer l’utilisateur des actions** : utiliser des annonces ARIA live (``aria-live``) pour communiquer les changements d’état, comme quand un élément est pris ou déplacé, ou quand il est déposé avec succès.

-- **Préserver la structure sémantique** : maintenir une structure HTML claire et logique pour que les lecteurs d’écran comprennent le contexte des éléments.

-- **Proposer des alternatives non-glisser-déposer** : permettre aussi la manipulation via des contrôles clavier classiques pour ceux qui ne peuvent pas utiliser la souris ou les gestes tactiles.

-----

- **Comment gérer les notifications non bloquantes pour qu'elles soient accessibles ?**

Pour gérer des notifications non bloquantes accessibles (comme des messages d’information, succès ou erreur qui apparaissent sans interrompre l’utilisateur), il faut suivre quelques bonnes pratiques essentielles :

Utiliser l’attribut ``aria-live`` sur la zone contenant la notification, avec une valeur appropriée (``polite`` pour les messages non urgents, ou ``assertive`` pour les messages importants). Cela permet aux lecteurs d’écran d’annoncer automatiquement le contenu qui apparaît.

Ne pas utiliser de focus automatique sur la notification, pour éviter de déranger la navigation ou la saisie en cours.

Veiller à ce que la notification soit visible visuellement et perceptible (bon contraste, taille lisible).

Proposer un moyen clair de fermer ou d’ignorer la notification pour que l’utilisateur puisse reprendre son activité facilement.

S’assurer que les notifications ne s’accumulent pas ou ne disparaissent pas trop rapidement, afin que tous les utilisateurs aient le temps de les percevoir.

Ainsi, une notification non bloquante accessible informe sans interrompre, est annoncée par les technologies d’assistance grâce à ``aria-live``, tout en laissant l’utilisateur libre de continuer son interaction.

-----

- **Qu’est-ce qu’un *skip link* ?**

Un ``skip link`` (ou lien de saut) est un lien invisible ou discret placé en tout début de page qui permet aux utilisateurs, notamment ceux naviguant au clavier ou avec un lecteur d’écran, de passer directement au contenu principal, en sautant les éléments répétitifs comme les menus de navigation.

Cela améliore grandement l’accessibilité en facilitant la navigation rapide et en évitant de devoir tabuler à travers de nombreux liens ou éléments avant d’atteindre l’information recherchée.

Typiquement, un ``skip link`` devient visible lorsqu’il reçoit le focus clavier, offrant ainsi une navigation plus fluide et efficace pour les personnes utilisant uniquement le clavier.

-----

- **Comment rendre un composant de type "carousel" accessible ?**

Pour rendre un carousel (diaporama d’images ou de contenus) accessible, il faut prendre en compte plusieurs aspects liés à la navigation, l’interaction au clavier, l’annonce de contenu par les lecteurs d’écran, et la maîtrise du mouvement.

Tout d'abord avoir une structure HTML sémantique. Utiliser des balises sémantiques comme ``<section>``, ``<figure>``, ``<button>``, etc. Et regrouper les éléments du carousel dans un conteneur avec un rôle explicite.

Ensuite, ajouter des boutons précédent / suivant avec des aria-label explicites pour avoir des contrôles de navigation accessibles. (Attention à ne fournir des boutons pour naviguer manuellement entre les diapositives que si le contenu est important.)

Tous les contrôles (flèches, indicateurs de position, pause/reprise) doivent être accessibles avec ``Tab``, ``Enter`` et ``Espace``.

Il faut également annoncer l’état actuel de la diapositive. Attention à ne pas utiliser ``aria-live`` de façon abusive. Annoncer uniquement les changements si l'utilisateur interagit.

On évite l'autoplay non contrôlable. Si le carousel est en mouvement automatique, il faut proposer un bouton pause avec un aria-label clair. Et de manière générale, limiter les animations.

```html

<div role="region" aria-label="Diaporama de témoignages" aria-roledescription="carousel">
  <div id="slide1" role="group" aria-roledescription="slide" aria-label="1 sur 3">
    <img src="image1.jpg" alt="Portrait de Jean, client satisfait">
  </div>
  <div id="slide2" role="group" aria-roledescription="slide" aria-label="2 sur 3" hidden>
    <img src="image2.jpg" alt="Portrait de Claire, cliente fidèle">
  </div>
  <div id="slide3" role="group" aria-roledescription="slide" aria-label="3 sur 3" hidden>
    <img src="image3.jpg" alt="Portrait de Paul, utilisateur depuis 5 ans">
  </div>

  <!-- Navigation -->
  <button onclick="prevSlide()" aria-label="Diapositive précédente">‹</button>
  <button onclick="nextSlide()" aria-label="Diapositive suivante">›</button>
  <button onclick="toggleAutoPlay()" id="pauseBtn" aria-label="Mettre en pause le diaporama">⏸️ Pause</button>
</div>

```

```css

<style>
  .carousel [hidden] {
    display: none;
  }
  .skip-link {
    position: absolute;
    top: -40px;
    left: 0;
    background: #000;
    color: #fff;
    padding: 8px;
    z-index: 100;
  }
  .skip-link:focus {
    top: 0;
  }
</style>

```

```javascript

<script>
  const slides = document.querySelectorAll('[role="group"]');
  let index = 0;
  let autoplay = null;

  function showSlide(i) {
    slides.forEach((slide, idx) => {
      slide.hidden = idx !== i;
    });
    index = i;
  }

  function nextSlide() {
    showSlide((index + 1) % slides.length);
  }

  function prevSlide() {
    showSlide((index - 1 + slides.length) % slides.length);
  }

  function toggleAutoPlay() {
    const btn = document.getElementById('pauseBtn');
    if (autoplay) {
      clearInterval(autoplay);
      autoplay = null;
      btn.innerText = '⏵';
      btn.setAttribute('aria-label', 'Reprendre le diaporama');
    } else {
      autoplay = setInterval(nextSlide, 5000);
      btn.innerText = '⏸️ Pause';
      btn.setAttribute('aria-label', 'Mettre en pause le diaporama');
    }
  }

  // Démarrer le diapo auto (optionnel)
  autoplay = setInterval(nextSlide, 5000);
</script>

```

-----

- **Que faire quand une animation peut perturber certains utilisateurs ?**

Quand une animation peut perturber certains utilisateurs, notamment ceux souffrant de troubles d'épilepsie photosensible ou de troubles cognitifs, il est essentiel de réduire ou supprimer l'effet visuel, ou de donner à l'utilisateur le contrôle.

Pour ça, il est impératif de respecter la préférence utilisateur ``prefers-reduced-motion``. Les systèmes d'exploitation modernes (Windows, macOS, iOS, Android) permettent à l'utilisateur d’indiquer qu’il préfère réduire les animations. En CSS, on peut détecter ce réglage :

```css

@media (prefers-reduced-motion: reduce) {
  * {
    animation: none;
    transition: none;
    scroll-behavior: auto;
  }
}

```

Il faut éviter les animations rapides, clignotantes ou trop mouvementées. Exit les clignotements rapides (plus de 3 flashs par seconde), les effets de zoom ou de parallaxe violents et les transitions automatiques sans interaction.

On donne le contrôle à l’utilisateur. Par exemple via un bouton "Pause" pour les animations automatiques (carousel, bannières animées, transitions).

On évite que des animations ne se déclenchent automatiquement sans que l’utilisateur les ait demandées ou sans qu’il soit préparé à les recevoir.

Et on utilise des animations subtiles et fonctionnelles. C'est-à-dire celles utiles à la compréhension d’un changement d’état (par exemple : validation de formulaire, déploiement d’un menu) qui sont généralement mieux tolérées que celles purement décoratives.

-----

- **Quels sont les avantages d’utiliser des bibliothèques accessibles (ex. Radix UI, Reach UI) ?**

Utiliser des bibliothèques accessibles comme Radix UI, Reach UI, ou Headless UI présente plusieurs avantages majeurs en matière d'accessibilité, de maintenabilité et d'efficacité de développement.

Ces bibliothèques sont conçues en suivant les normes WAI-ARIA, WCAG et les bonnes pratiques d’accessibilité dès le départ. Les composants comme ``Dialog``, ``Tabs``, ``Accordion``, ``Tooltip``, ``Menu``, etc., gèrent automatiquement le focus, les rôles ARIA, le clavier, les régions live, etc.

Il n'y a pas à réimplémenter manuellement la logique de focus, de navigation clavier ou les attributs ARIA.  Cela réduit le risque d’oublis, d’incohérences ou de bugs liés à l’accessibilité.

La plupart de ces bibliothèques sont "headless", c’est-à-dire sans styles CSS imposés, ce qui permet d’utiliser ses propres styles/UI, tout en bénéficiant d’un comportement accessible. On gardes le contrôle total du design sans sacrifier l’accessibilité.

Elles sont conçues pour être interopérables avec les frameworks modernes (React, Next.js, etc.), testées avec les lecteurs d’écran (NVDA, VoiceOver, JAWS…), et maintenues activement par des équipes spécialisées.
