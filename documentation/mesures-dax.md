# Mesures DAX du tableau de bord

Ce document présente les principales mesures DAX utilisées dans le tableau de bord Power BI consacré au suivi des coûts, des durées, des livrables et des alertes projet.

Les mesures sont regroupées dans la table `_Mesures`.

## Mesures de base

### Coût prévu

Calcule le montant total prévu pour les projets ou les phases sélectionnés.

```DAX
Coût prévu =
SUM(Projects_plans[Planned_Cost])
```

### Coût réel

Calcule le montant total réellement dépensé.

```DAX
Coût réel =
SUM(Actual_Costs[Actual_Cost])
```

### Durée prévue

Calcule la durée totale initialement prévue.

```DAX
Durée prévue =
SUM(Projects_plans[Planned_Duration])
```

### Durée réelle

Calcule la durée totale réellement observée.

```DAX
Durée réelle =
SUM(Actual_Duration[Actual_Duration])
```

### Livrables prévus

Calcule le nombre total de livrables prévus.

```DAX
Livrables prévus =
SUM(Projects_plans[Planned_Delivrable])
```

### Livrables réels

Calcule le nombre total de livrables effectivement réalisés.

```DAX
Livrables réels =
SUM(Actual_Delivrable[Actual_Deliverables])
```

## Mesures d’écart

### Écart coût %

Compare le coût réel au coût prévu.

Une valeur positive indique un dépassement budgétaire. Par exemple, une valeur de `0,20` correspond à un dépassement de 20 %.

```DAX
Écart coût % =
DIVIDE(
    [Coût réel] - [Coût prévu],
    [Coût prévu]
)
```

### Écart durée %

Compare la durée réelle à la durée prévue.

Une valeur positive indique un retard. Une valeur négative signifie que la durée réelle est inférieure à la durée prévue.

```DAX
Écart durée % =
DIVIDE(
    [Durée réelle] - [Durée prévue],
    [Durée prévue]
)
```

### Écart livrables %

Mesure la proportion de livrables prévus qui n’ont pas été réalisés.

Une valeur positive indique un manque de livrables par rapport à l’objectif prévu.

```DAX
Écart livrables % =
DIVIDE(
    [Livrables prévus] - [Livrables réels],
    [Livrables prévus]
)
```

## Mesures d’alerte

Le seuil d’alerte retenu est de **15 %**.

### Alerte coût

Renvoie `1` lorsque le dépassement de coût atteint ou dépasse 15 %, sinon `0`.

```DAX
Alerte coût =
IF(
    [Écart coût %] >= 0.15,
    1,
    0
)
```

### Alerte durée

Renvoie `1` lorsque le dépassement de durée atteint ou dépasse 15 %, sinon `0`.

```DAX
Alerte durée =
IF(
    [Écart durée %] >= 0.15,
    1,
    0
)
```

### Alerte livrables

Renvoie `1` lorsque le manque de livrables atteint ou dépasse 15 %, sinon `0`.

```DAX
Alerte livrables =
IF(
    [Écart livrables %] >= 0.15,
    1,
    0
)
```

### Alerte globale

Renvoie `1` lorsqu’au moins une alerte est déclenchée sur le coût, la durée ou les livrables.

```DAX
Alerte globale =
IF(
    [Alerte coût] = 1
        || [Alerte durée] = 1
        || [Alerte livrables] = 1,
    1,
    0
)
```

## Indicateurs de suivi

### Nombre de projets

Calcule le nombre distinct de projets présents dans le contexte de filtre.

```DAX
Nombre de projets =
DISTINCTCOUNT(Projects_plans[Project ID])
```

### Nombre de phases en alerte

Additionne les alertes globales sur les lignes de la table de planification.

```DAX
Nombre de phases en alerte =
SUMX(
    Projects_plans,
    [Alerte globale]
)
```

Cette mesure suppose que la granularité de `Projects_plans` correspond bien à une ligne par phase à comptabiliser.

### Nombre de projets en alerte

Compte les projets distincts pour lesquels l’alerte globale vaut `1`.

```DAX
Nombre de projets en alerte =
COALESCE(
    COUNTROWS(
        FILTER(
            VALUES(Projects_plans[Project ID]),
            [Alerte globale] = 1
        )
    ),
    0
)
```

### Taux de projets en alerte

Calcule la proportion de projets présentant au moins une alerte.

```DAX
Taux de projets en alerte =
COALESCE(
    DIVIDE(
        [Nombre de projets en alerte],
        [Nombre de projets]
    ),
    0
)
```

### Nombre d’alertes durée

Additionne les alertes liées aux durées dans le contexte sélectionné.

```DAX
Nombre alertes durée =
SUMX(
    Projects_plans,
    [Alerte durée]
)
```

### Nombre d’alertes livrables

Additionne les alertes liées aux livrables.

```DAX
Nombre alertes livrables =
SUMX(
    Projects_plans,
    [Alerte livrables]
)
```

### Nombre d’alertes coût

Additionne les alertes liées aux coûts.

```DAX
Nombre alertes coût =
SUMX(
    Projects_plans,
    [Alerte coût]
)
```

### Alerte projet total

Compte les projets distincts en alerte à partir de la table de localisation des projets.

```DAX
Alerte projet total =
SUMX(
    VALUES(Projects_Locations[Project ID]),
    IF(
        [Alerte globale] = 1,
        1,
        0
    )
)
```

## Logique générale du système d’alerte

Le tableau de bord applique la logique suivante :

1. calcul des valeurs prévues et réelles ;
2. calcul des écarts en pourcentage ;
3. déclenchement d’une alerte lorsqu’un écart atteint 15 % ;
4. création d’une alerte globale lorsqu’au moins un indicateur est en alerte ;
5. agrégation des alertes par projet, phase, région, pays ou type de projet.

## Requête DAX de contrôle

La requête suivante permet d’afficher simultanément les résultats des principales mesures dans la vue Requête DAX de Power BI.

Elle sert à vérifier les mesures, mais ne fait pas partie des mesures enregistrées dans le modèle.

```DAX
EVALUATE
    SUMMARIZECOLUMNS(
        "Coût prévu", [Coût prévu],
        "Coût réel", [Coût réel],
        "Durée prévue", [Durée prévue],
        "Durée réelle", [Durée réelle],
        "Livrables prévus", [Livrables prévus],
        "Livrables réels", [Livrables réels],
        "Écart coût %", [Écart coût %],
        "Écart durée %", [Écart durée %],
        "Écart livrables %", [Écart livrables %],
        "Alerte coût", [Alerte coût],
        "Alerte durée", [Alerte durée],
        "Alerte livrables", [Alerte livrables],
        "Alerte globale", [Alerte globale],
        "Nombre de projets", [Nombre de projets],
        "Nombre de phases en alerte", [Nombre de phases en alerte],
        "Nombre de projets en alerte", [Nombre de projets en alerte],
        "Taux de projets en alerte", [Taux de projets en alerte],
        "Nombre alertes durée", [Nombre alertes durée],
        "Nombre alertes livrables", [Nombre alertes livrables],
        "Nombre alertes coût", [Nombre alertes coût],
        "Alerte projet total", [Alerte projet total]
    )
```