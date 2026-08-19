# [TECH] Consultation des stocks d'articles

**Type** : Story technique
**Composant** : `sportStore` — domaine Article
**Epic** : Gestion des stocks (à créer si absent)

## Contexte

Le catalogue d'articles ne porte aujourd'hui aucune notion de quantité disponible. On introduit un stock par article, consultable via un nouveau service, et on relie son cycle de vie à celui de l'article (création, consultation, suppression).

## Description fonctionnelle

- Chaque article possède un stock (quantité entière ≥ 0, jamais négatif).
- La création d'un article initialise son stock à 0.
- Le remplacement d'un article existant (`PUT` sur un nom déjà connu) **ne modifie pas** le stock en place.
- La consultation d'un article (`GET /store/articles/{name}`) décrémente son stock d'une unité, sans jamais passer sous 0.
- Si le stock est déjà à 0 au moment de la consultation, une erreur métier `OutOfStockException` est levée (`400 Bad Request`).
- La suppression d'un article (`DELETE /store/articles/{name}`) supprime également la ligne de stock associée.
- Un nouveau service permet de consulter le stock de chaque article.

## Exemple d'API — nouvelle route

**Requête**

```
GET /store/articles/stocks
```

**Réponse `200 OK`**

```json
[
  {
    "articleId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "name": "Soccer Ball",
    "quantity": 12
  },
  {
    "articleId": "f0e1d2c3-b4a5-6978-89ab-cdef01234567",
    "name": "Running Shoes",
    "quantity": 0
  }
]
```

**Réponse `500 Internal Server Error`**

```json
{
  "error": "Storage failure while reading stocks"
}
```

## Exemple d'impact sur une route existante

**Requête**

```
GET /store/articles/Soccer%20Ball
```

**Réponse `200 OK`** — le stock est décrémenté d'une unité après cette lecture (12 → 11)

```json
{
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "name": "Soccer Ball",
  "category": "Team Sports",
  "price": 29.99
}
```

**Requête** (même article, stock déjà à 0)

```
GET /store/articles/Running%20Shoes
```

**Réponse `400 Bad Request`** — levée par `OutOfStockException`

```json
{
  "error": "Article out of stock: Running Shoes"
}
```

