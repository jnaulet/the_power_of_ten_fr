# La puissance de 10 - Des Règles pour Développer du Code Critique Sûr

*Gerard J. Holzmann, traduction par J.Naulet

NASA/JPL Laboratory for Reliable Software

PAsadena, CA 91109*

La plupart des projets sérieux de développement logiciel utilisent des lignes directrices. Celles-ci ont pour but de définir quelles sont les règles de base pour le logiciel à écrire : comment il doit être structuré & quelles caractéristiques du langage doivent ou ne doivent pas être utilisées. Etonnament, il y a peu de consensus sur ce qu'est un bon standard de codage. Parmi le grand nombre qui ont été écrits, on discerne peu de motifs récurrents, à l'exception du fait que chaque docuement tend à être plus long que le précedent. Le résultat étant que la plupart des documents existants contiennent bien plus d'une centaine de règles, à la justification parfois discutable. Certaines règles, notamment celles qui tentent de définir l'usage des espaces dans les programmes, peuvent avoir été introduites par simple préférence personnelle; d'autres ont pour but d'éviter certains types d'erreurs improbables & très spécifiques, fruit d'efforts précédents au sein d'une même organisation. Sans surprise, les lignes directrices existantes tendent à n'avoir que peu d'effet sur ce que font les developpeurs quand ils écrivent réellement le code. L'aspect le plus élimnatoire de beaucoup de ces lignes directrices est qu'elle autorisent rarement une vérification complète & automatique de la conformité. La vérification par des outils tiers est importante, étant donné qu'il est souvent impossible de faire une revue de code manuelle des milliers de lignes de code présentes dans les plus grosses applications.


Le bénéfice des lignes directrices existantes est par conséquent souvent mineur, même pour les applications critiques. Un ensemble vérifiable de règles de codage bien choisies peut, néanmoins, rendre les éléments logiciels plus analysables en profondeur, pour des propriétés qui vont au delà de la conformité avec ces mêmes règles. Pour être fficace, cependant, cet ensemble de règles doit être concis, et doit être assez clair pour pouvoir être compris & retenu. Les règles devront être suffisamment précises pour pouvoir être vérifiée mécaniquement. Pour mettre une limite haute quant au nombre de règles pour des lignes directrices efficaces, j'arguerais que nous pouvons obtenir un bénéfice significatif en réduisant le nombre à pas plus de 10. Un ensemble aussi réduit, bien sûr, ne peut tout englober, mais il peut nous donner un ancrage pour atteindre des effets mesurables sur la fiabilitié & la vérifiabilite (?) du logiciel. Pour permettre une vérification forte, les règles sont quelque peu strictes - certains diraient même Draconniennes. Le compromis, ceci dit, doit être clair. Quand c'est vraiment important, en particulier lors du développement de code de sécurité critique, il peut être utile d'aller plus loin & de fixer des limites plus strictes que ce qui est simplement souhaitable. En échange, nous devrions être capables de démontrer de façon plus convaincante que le logiciel critique fonctionne comme prévu.

## Dix Règles pour un Code Critique Sûr
Le choix du langage pour un code critique sécurisé est en soi une considération clé, mais nous n'en débattrons pas beaucoup ici. Dans de nombreuses organisations, JPL inclus, le code critique est écrit en C. De par son long historique, il existe un grande bibliothèque d'outils pour ce langage, incluant de puissants analyseurs de code source, des extracteurs de modèles logiques, des outils de mesure, debogueurs, outils de tests, & un vaste choix de compilateurs stables & au point. Pour cette raison, le C est aussi le sujet principal de le plupart des lignes directrices  qui ont été développées. Pour des raisons raisonnablement pragmatiques, donc, nos règles de codage s'appliquent au C & tentent d'optimiser notre capacité à vérifier plus en profondeur la fiabilité des applications critiques écrites en C.

Les règles suivantes peuvent fournir un bénéfice, en particulier si leur faible nombre signifie que les developpeurs vont reéllement les appliquer. Chause règle s'accompagne d'un rapide résumé qui explique son inclusion.

1. ***Règle***: Réduire le code à des constructions simples de flux de contrôle - N'utilisez pas d'instructions *goto*, *setjmp* ou *longjmp* ni de *récursion* direct ou indirecte.

***Justification***: Un flux de contrôle plus simple se traduit par de plus grandes capacités de vérification & souvent une amélioration de la clarté du code. L'interdiction de la récursion est peut-être le plus grosse surprise ici. Sans récursion, ceci dit, nous avons la garantie d'un arbre d'appels des fonctions acyclique, qui peut être exploîté par des analyseurs de code, et qui peut directement aider à démontrer que toutes les exécutions qui doivent être délimintées le sont. (Notez que cette règle ne requiert pas que toutes les fonctions n'aient qu'un unique point de retour - bien que cela simplifie souvent le flux de contrôle. Il y a suffisamment de cas, en fait, où un retour d'erreur anticipé est la solution la plus simple)

