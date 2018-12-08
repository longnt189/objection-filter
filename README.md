[![Build Status](https://travis-ci.org/tandg-digital/objection-filter.svg?branch=master)](https://travis-ci.org/tandg-digital/objection-filter) [![Coverage Status](https://coveralls.io/repos/github/tandg-digital/objection-filter/badge.svg?branch=master)](https://coveralls.io/github/tandg-digital/objection-filter?branch=master)

# What is objection-filter?
objection-filter is a filtering module for the [objection.js](https://github.com/Vincit/objection.js) ORM. It aims to fulfil some common requirements that occur often during API development:

#### 1. Filtering on nested relations
For example, if you have the models _Customer_ belongsTo _City_ belongsTo _Country_, we can query all _Customers_ where the _Country_ starts with `A`.

#### 2. Eagerly loading data
Eagerly load a bunch of related data in a single query. This is useful for getting a list models e.g. _Customers_ then including all their _Orders_ in the same query.

#### 3. Aggregation and reporting
Creating quick counts and sums on a model can speed up development significantly. An example could be the _numberOfOrders_ for a _Customer_ model.

# Shortcuts

* [Changelog](doc/CHANGELOG.md)
* [Recipes](doc/RECIPES.md)
* [Aggregation](doc/AGGREGATIONS.md)

# Installation

`npm i objection-filter --save`

> objection-filter >= 1.0.0 is fully backwards compatible with older queries, but now supports nested [and/or filtering](#logical-expressions) as well as the new objection.js object notation. The 1.0.0 denotation was used due to these changes and the range of query combinations possible.

# Usage

The filtering library can be applied onto every _findAll_ REST endpoint e.g. `GET /api/{Model}?filter={"name": "longnt189"}`

A typical express route handler with a filter applied:
```js
const { buildFilter } = require('objection-filter');
const { Customer } = require('./models');

app.get('/Customers', function(req, res, next) {
  buildFilter(Customer)
    .build(JSON.parse(req.query.filter))
    .then(customers => res.send(customers))
    .catch(next);
});
```

Available filter properties include:
```js
// GET /api/Customers
{
  // Filtering and eager loading
  "filter": {
      "firstName": "John",
      "profile.isActivated": true,
      "city.country": { "$like": "A" }
  },
  // An objection.js order by expression
  "order": "firstName desc",
  "limit": 10,
  "offset": 10,
  // An array of dot notation fields to select on the root model and eagerly loaded models
  "fields": ["firstName", "lastName", "orders.code", "products.name"]
}
```

> Besides following these instructions, you can use `eager` operators at https://github.com/tandg-digital/objection-filter
# Filter Operators

There are a number of built-in operations that can be applied to columns (custom ones can also be created). These include:

1. **$like** - The SQL _LIKE_ operator, can be used with expressions such as _ab%_ to search for strings that start with _ab_
2. **$gt/$lt/$gte/$lte** - Greater than and Less than operators for numerical fields
3. **=/$equals** - Explicitly specify equality
4. **$in** - Whether the target value is in an array of values
5. **$exists** - Whether a property is not null
6. **$or** - A top level _OR_ conditional operator
7. **CUSTOM: $likeLower** - The SQL _LIKE_ operator, can be used with expressions such as _ab%_ to search for strings that start with _ab_ and _no distinction between upper and lowercase letters_

For any operators not available (eg _ILIKE_, refer to the custom operators section below).

#### Example

An example of operator usage
```json
{
  "filter": {
    "property0": "Exactly Equals",
    "property1": {
      "$equals": 5
    },
    "property2": {
      "$gt": 5
    },
    "property3": {
      "$lt": 10,
      "$gt": 5
    },
    "property4": {
      "$in": [ 1, 2, 3 ]
    },
    "property5": {
      "$exists": false
    },
    "property6": {
      "$or": [
        { "$in": [ 1, 2, 3 ] },
        { "$equals": 100 }
      ]
    }
  }
}
```

#### Custom Operators

If the built in filter operators aren't quite enough, custom operators can be added. A common use case for this may be to add a `lower case LIKE` operator, which may vary in implementation depending on the SQL dialect.

Example:

```js
const options = {
  operators: {
    $ilike: (property, operand, builder) =>
      builder.whereRaw('?? ILIKE ?', [property, operand])
  }
};

buildFilter(Person, null, options)
  .build({
    filter: {
      $where: {
        firstName: { $ilike: 'John' }
      }
    }
  })
```

The `$ilike` operator can now be used as a new operator and will use the custom operator callback specified.
