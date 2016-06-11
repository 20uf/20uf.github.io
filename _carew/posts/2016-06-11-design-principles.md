---
title:  Les simplicités de conception
layout: post
tags:
    - Principes
    - développeur
---

Les simplicités de conception
---------------------

Il est important en tant que développeur de connaître les designs patterns, ils sont la pour résoudre les problèmes récurrents de conception, cependant il est tout aussi primordiale de les utiliser avec parcimonie.
Pour cette raison je tiens à rappeler des principes de simplicités à ne pas oublier. 

#### DRY (Don’t repeat yourself)
Ne vous répétez pas.
A mon sens la répétition démontre un problème de découplage ou de conception, en effet les évolutions deviennent complexes et le risque de bug est augmenté considérablement.
Le respect du principe DRY doit permettre un code réutilisable, facile à maintenir et testable.


#### YAGNI (You ain't gonna need it)
Vous n'en aurez pas besoin.
On peut être dans des cas ou l'on souhaite anticipé une fonctionnalité, on rentre alors une inflation qui peut être contre-productif, difficile à maintenir ou bien on se retrouve avec du code mort. 
Le principe YAGNI est de développer uniquement la fonctionnalité demandée sans surplus.


#### KISS (Keep it simple, stupid)
Ne complique pas les choses. Ou garde ça simple, idiot. 
Ce principe est très proche de celui de YAGNI, une fonctionnalité ne nécessite pas forcement une complexité élevée, si par exemple l'utilisation d'un design pattern n'est pas requis pour le besoin, le principe KISS incite à écrire un code simple et stupide.


