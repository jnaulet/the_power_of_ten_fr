# La puissance de 10 - Des Règles pour Développer du Code Critique Sûr

*Gerard J. Holzmann, traduction par J.Naulet*

*NASA/JPL Laboratory for Reliable Software*

*PAsadena, CA 91109*

La plupart des projets sérieux de développement logiciel utilisent des lignes directrices. Celles-ci ont pour but de définir quelles sont les règles de base pour le logiciel à écrire : comment il doit être structuré & quelles caractéristiques du langage doivent ou ne doivent pas être utilisées. Etonnament, il y a peu de consensus sur ce qu'est un bon standard de codage. Parmi le grand nombre qui ont été écrits, on discerne peu de motifs récurrents, à l'exception du fait que chaque docuement tend à être plus long que le précedent. Le résultat étant que la plupart des documents existants contiennent bien plus d'une centaine de règles, à la justification parfois discutable. Certaines règles, notamment celles qui tentent de définir l'usage des espaces dans les programmes, peuvent avoir été introduites par simple préférence personnelle; d'autres ont pour but d'éviter certains types d'erreurs improbables & très spécifiques, fruit d'efforts précédents au sein d'une même organisation. Sans surprise, les lignes directrices existantes tendent à n'avoir que peu d'effet sur ce que font les developpeurs quand ils écrivent réellement le code. L'aspect le plus élimnatoire de beaucoup de ces lignes directrices est qu'elle autorisent rarement une vérification complète & automatique de la conformité. La vérification par des outils tiers est importante, étant donné qu'il est souvent impossible de faire une revue de code manuelle des milliers de lignes de code présentes dans les plus grosses applications.


Le bénéfice des lignes directrices existantes est par conséquent souvent mineur, même pour les applications critiques. Un ensemble vérifiable de règles de codage bien choisies peut, néanmoins, rendre les éléments logiciels plus analysables en profondeur, pour des propriétés qui vont au delà de la conformité avec ces mêmes règles. Pour être fficace, cependant, cet ensemble de règles doit être concis, et doit être assez clair pour pouvoir être compris & retenu. Les règles devront être suffisamment précises pour pouvoir être vérifiée mécaniquement. Pour mettre une limite haute quant au nombre de règles pour des lignes directrices efficaces, j'arguerais que nous pouvons obtenir un bénéfice significatif en réduisant le nombre à pas plus de 10. Un ensemble aussi réduit, bien sûr, ne peut tout englober, mais il peut nous donner un ancrage pour atteindre des effets mesurables sur la fiabilitié & la vérifiabilite (?) du logiciel. Pour permettre une vérification forte, les règles sont quelque peu strictes - certains diraient même Draconniennes. Le compromis, ceci dit, doit être clair. Quand c'est vraiment important, en particulier lors du développement de code de sécurité critique, il peut être utile d'aller plus loin & de fixer des limites plus strictes que ce qui est simplement souhaitable. En échange, nous devrions être capables de démontrer de façon plus convaincante que le logiciel critique fonctionne comme prévu.

## Dix Règles pour un Code Critique Sûr
Le choix du langage pour un code critique sécurisé est en soi une considération clé, mais nous n'en débattrons pas beaucoup ici. Dans de nombreuses organisations, JPL inclus, le code critique est écrit en C. De par son long historique, il existe un grande bibliothèque d'outils pour ce langage, incluant de puissants analyseurs de code source, des extracteurs de modèles logiques, des outils de mesure, débogueurs, outils de tests, & un vaste choix de compilateurs stables & au point. Pour cette raison, le C est aussi le sujet principal de le plupart des lignes directrices  qui ont été développées. Pour des raisons raisonnablement pragmatiques, donc, nos règles de codage s'appliquent au C & tentent d'optimiser notre capacité à vérifier plus en profondeur la fiabilité des applications critiques écrites en C.

Les règles suivantes peuvent fournir un bénéfice, en particulier si leur faible nombre signifie que les developpeurs vont reéllement les appliquer. Chaque règle s'accompagne d'un rapide résumé qui explique son inclusion.

1. ***Règle***: Réduire le code à des constructions simples de flux de contrôle - N'utilisez pas d'instructions *goto*, *setjmp* ou *longjmp* ni de *récursion* direct ou indirecte.

***Justification***: Un flux de contrôle plus simple se traduit par de plus grandes capacités de vérification & souvent une amélioration de la clarté du code. L'interdiction de la récursion est peut-être le plus grosse surprise ici. Sans récursion, ceci dit, nous avons la garantie d'un arbre d'appels des fonctions acyclique, qui peut être exploîté par des analyseurs de code, et qui peut directement aider à démontrer que toutes les exécutions qui doivent être couvertes le sont. (Notez que cette règle ne requiert pas que toutes les fonctions n'aient qu'un unique point de retour - bien que cela simplifie souvent le flux de contrôle. Il y a suffisamment de cas, en fait, où un retour d'erreur anticipé est la solution la plus simple)

2. ***Règle***: Toutes les boucles doivent avoir une limite haute fixe. Il doit être ridiculement facile pour un outil d'analyse de prouver statiquement qu'une limite d'itérations prédéfinie ne peut être dépassée. Si la limitation ne peut être prouvée de façon statique, la règle est considérée comme violée.

***Justification***: L'absence de récursion & la présence de boucles limitées empêche la jardinage en mémoire. Cette règle, bien sûr, ne s'applique pas aux itérations infinies par essence (ex: dans un scheduler de process). Dans ces cas particuliers, la règle inverse est appliquée: il doit être possible de prouver statiquement que l'itération *ne peut* se terminer.
Une fois d'appliquer cette règle est d'ajouter explicitement une limite haute à toutes les boucles ayant un nombre variable d'itérations (ex: un code qui parcourerait une liste chaînée). Quand la limite haute est dépassée une assertion d'échec est lancée & la fonction contenant l'itération fautive retourne une erreur. (Voir règle 5 sur l'utilisation des assertions).


3. ***Règle***: Ne pas utiliser d'allocation mémoire dynamique après l'initialisation.

***Justification***: Cette règle est courante pour le code critique de sécurité & apparaît dans la plupart des lignes directrices. La raison est simple: les allocateurs mémoire, tels que *malloc* & les ramasse-miettes ont souvent un comprtement non prévisible qui peut impacter la performance de façon significative. Une classe notable d'erreurs de codage émerge d'une mauvaise prise en charge de l'allocation mémoire & des routines de libération: oulbier de libérer la mémoire ou conitnuer à à utiliser la mémoire après qu'elle ait été libérée, tenter d'allouer plus de mémoire qu'il n'y en a de physiquement disponible, dépasser les limites de la mémoire allouée, etc. Forcer toutes les applications à vivre dans une zone mémoire fixe, pré-allouée peut éliminer beaucoup de ces problèmes & rendre la vérification de l'utilisation mémoire plus facile. Notez que la seule façon de demander dynamiquement de la mémoire en l'absence d'allocation mémoire en provenance du tas *(heap)* est d'utiliser la mémoire de la pile. En l'absence de récursion (Règle 1), une limite haute quant à l'utilisation de la pile mémoire peut être déduite statiquement, rendant possible de prouver qu'une application va toujours vivre à l'intérieur de sa zone pré-allouée.

4. ***Règle***: Auncune fonction ne doit être plus longue que ce qui peut être imprimé sur une feuille de papier dans un format standard de référence avec une ligne par définition & une ligne par déclaration. Typiquement, cela signifie pas plus de 60 lignes de code par fonction.

***Justification***: Chaque fonction doit être une unité logique dans le code qui est compréhensible & vérifiable en tant qu'unité. Il est beaucoup plus difficile de comprendre une unité logique qui s'étend sur plusieurs écrans sur un affichage d'ordinateur ou sur de multiples pages une fois imprimée. Les fonctions excessivement longues sont souvent un marqueur d'un code très mal structuré.

5. ***Règle***: La *densité d'assertions* du code doit être en moyenne d'au moins 2 assertions par fonction. Les assertions sont utilisées pour se protéger de conditions anormales qui ne devraient jamais se produire lors d'exécutions en conditions réelles. Les assertions doivent toujours être sans aucun effet de bord & doivent être définies en tant que tests Booléens. Quand une asseertion échoue, une action de sauvetage doit être entreprise, ex, en retournant une erreur à l'appelant de la fonction qui exécute l'assertion échouée. Toute assertion pour laquelle un analyseur statique peut prouver qu'elle ne peut jamais échouer ou jamais réussir viuole cette règle. (Il n'est pas possible de satisfaire cette exigence en ajoutant d'inutiles appels à *"assert(true)"*.)

***Justification***: Les statistiques pour les efforts de codage industriel indiquent que les tests unitaires trouvent souvent au moins une erreur pour 10 à 100 lignes de code écrites. Les chances d'intercepter ces erreurs augmente avec la densité des assertions. L'utilisation des assertions est aussi souvent recommandée en tant qu'élément d'une stratégie de codage fortement défensive. Les assertions peuvent être tuilisées pour vérifier pré- & post- conditions de fonctions, valeurs de paramètres, valeurs de retour des fonctions, & constantes de boucles. Comme les assertions sont sans effet de bord, elles peuvent être désactivées sélectivement pour les tests de performances critiques.
Un usage typique d'assertion serait le suivant:

    if (!c_assert(p >= 0) == true) {
      return ERROR;
    }

avec l'assertion décrite ainsi :

    #define c_assert(e) ((e) ? (true) : \
     tst_debugging(”%s,%d: assertion ’%s’ failed\n”, \
     __FILE__, __LINE__, #e), false)

Dans cette définition, __FILE__ & __LINE__ sont prédéfinies par le préprocesseur de macros & correspondent au nom du fichier & le numéro de ligne de l'assertion échouée. La syntax #e transforme la condiution d'assertion e en chaîne de caractères qui est affichée en tant qu'élement du message d'erreur. Dans un code destiné à un processeur embarqué if n'y a bien sûr pas lieu d'afficher le message d'erreur lui-même - Dans ce cas, l'appel à tst_debugging est trasnformé en no-op, & l'assertion se tranforme en pur test Booléen qui active la gestion d'erreur à partir du comportement anormal.

6. ***Règle***: Les objets & variables doivent être déclarés au niveau de portée le plus faible.

***Justification***: