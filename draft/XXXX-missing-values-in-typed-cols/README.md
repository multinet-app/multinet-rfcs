---
RFC: XXXX
Author: Jack Wilburn
Created: 2020-10-26
Last-Modified: 2020-10-26
---

# RFC XXXX: Missing Values In Typed Columns

As we move to having typed data in Multinet, it is important to handle missing data elegantly, even with typing. Currently, if a column has any amount of missing data, we assume that the data is a "label" type. Since the "label" type is essentially a catch-all for data that we can't provide a meaningful type for, it's important that if there is a semantically meaningful type, we provide it to visualization applications. This must, also, be true in the case where we have mostly complete data with just a few missing rows. Without this information, the client applications cannot adequately choose a visualization for these columns.

There will likely be some client- and server-side considerations for handling missing data. This will include correctly parsing data and when reading it into the ArangoDB types (or we may choose to leave them as strings in the db). Types that we define will be referenced as *semantic types*, and native ArangoDB types will be referenced as *Arango types*.

## Motivation

We now have a motivation to supply data schema information to visualization applications, UpSet. Upset requires set membership information and other data types to generate its visualization. We can also utilize this type information in the other visualization applications that support multinet (MultiLink and MultiMatrix).

Since we now need to pass semantic typing into these applications, knowing what the types are, even if there is missing data, is essential. As such, we'll need to modify our typing logic to handle missing data. This stretches from the frontend, where we provide type suggestions, to the backend, where we store the type information and disseminate it to the vis applications.

## Proposals

Multinet should support typing of columns with missing entries, rather than just assuming such columns are `label` type. There are at least 3 places that we should support this paradigm:

1. `multinet-client` on data preview/type suggestion.
2. `multinet-server` on data upload and data fetching.
3. `multinetjs` on data upload and data fetching.

For the type suggester, there is a code chunk in the [Reference Implementation](#reference-implementation) section that suggests a way that we can check for the type of a column, even when data is missing.

For the other repos, `multinet-server` and `multinetjs`, the considerations are slightly different. We'll need to think about how to store and serve data with missing values, and how to disseminate the type information given that there are missing values. Additionally, an open question is, should we tell client applications to expect some missing data in typed columns? This could help when rendering visualizations.

## Backwards Compatibility

There will be no added backwards incompatibilities from this change. There may be some knock-on consequences for apps that can utilize types in the case that there was data uploaded with the old type definitions, but the impact will be minimal.

## Reference Implementation

The client already has logic to calculate the type of the columns but does not allow for missing values as it stands. This implementation should just be an extension of the existing logic.

Jake gave a suggestion for how we might calculate if a column is actually the correct type with some missing values:

```
const number = entry.number + entry.empty === entry.total;
```
