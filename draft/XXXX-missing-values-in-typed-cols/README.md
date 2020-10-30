---
RFC: XXXX
Author: Jack Wilburn
Created: 2020-10-26
Last-Modified: 2020-10-26
---

# RFC XXXX: Missing Values In Typed Columns

As we move to having typed data in MultiNet, it is important to handle missing data elegantly, even with typing. Currently, if a column has any amount of missing data, we assume that the data is a "label" type. Since the "label" type is essentially a catch-all for data that we can't provide a meaningful type for, it's important that if there is a semantically meaningful type, we provide it to visualization applications. This must, also, be true in the case where we have mostly complete data with just a few missing rows. Without this information, the client applications cannot adequately choose a visualization for these columns.

There will likely be some client- and server-side considerations for handling missing data. This will include correctly parsing data and when reading it into the ArangoDB types (or we may choose to leave them as strings in the db). Types that we define will be referenced as *semantic types*, and native ArangoDB types will be referenced as *Arango types*.

## Motivation

In general, we want to support typed columns for a variety of purposes, including visualization, comparison, grouping, etc. In the real world, datasets almost always have missing entries, and, as such, MultiNet will need to accommodate column typing with missing data. 

## Proposals

MultiNet should support typing of columns with missing entries, rather than just assuming such columns are `label` type. There are at least 3 places that we should support this paradigm:

1. `multinet-client` on data preview/type suggestion.
2. `multinet-server` on data upload and data fetching.
3. `multinetjs` on data upload and data fetching.

For the type suggester, there is a code chunk in the [Reference Implementation](#reference-implementation) section that suggests a way that we can check for the type of a column, even when data is missing.

For the other repos, `multinet-server` and `multinetjs`, the considerations are slightly different. We'll need to think about how to store and serve data with missing values, and how to disseminate the type information given that there are missing values. Additionally, an open question is, should we tell client applications to expect some missing data in typed columns? This could help when rendering visualizations.

## Backwards Compatibility

There is a potential for some backwards incompatibilities due to the types of the data being changed. Currently, missing data is represented as an empty string; whereas, when we have typed data, the returned value will be `null`. This could impact some of our data processing in the client/vis apps.

## Implementation Strategies

The client already has logic to calculate the type of the columns but does not allow for missing values as it stands. This implementation should just be an extension of the existing logic.

Jake gave a suggestion for how we might calculate if a column is actually the correct type with some missing values:

```
const number = entry.number + entry.empty === entry.total;
```

When a column has `entry.empty > 0` we could use this to create a flag that is sent with API calls. This would alert whoever is receiving the data that there are missing values in the column. This is of debatable utility, and is something that could be calculated on the client