# Syntax ideas

Common operations take the active form of the verb:

    selects()     
    summarises()   (aka summarizes)
    transforms()   (aka mutates)
    reorders()     (aka arranges)
    filters()      (aka subsets)

    applies()
    combines()

(`filters` is more obviously a verb than `subsets`)

Specifying grouping is more like an adverb:

    groupwise()

## Combining operations

Individual operations are combined with `+` to form a compound operation:

    op <- counts(by_var(cod, hod)) +
      filters(is.na(hod)) +
      counts(by_var(cod, hod)) +
      groupwise(by_var(cod), transforms(prop = freq / sum(freq)) +
      arranges(desc(freq))

When combined with a data source, the operations can be performed:

    source(deaths) + op

This gives a class hierarchy:

    op -> subset_op | filter_op | group_op | transform_op | order_op
    ops -> list of ops
    complete_ops -> ops + source

An complete operation has one data source and a list of operations. A partial operation just has a list of operations.

With `+` defined as:

    op + op -> ops
    ops + op -> ops
    complete_ops + op -> complete_ops

    op + source -> complete_ops
    ops + source -> complete_ops
    complete_ops + source -> complete_ops

## Grouping

In `groupwise` (an adverb), you can specify group as:

    by_var(mycol)
    by_(mycol) # perhaps an abbrevation

    x <- "mycol"
    by_name(x)

    by_rows()

You can also provide your own function that takes a data frame as input and returns a single vector where each group has a common value.

## Arbitrary operations.

Functions that don't return a data frame use `applies`:

    baseball +
      groupwise(by_(id), does(lm, formula = g ~ year))

The compliment, `combines`, collapses the list back into a `data.frame`

    baseball +
      groupwise(by_(id), applies(lm, formula = g ~ year)) +
      combines(coef)


## Planning/compilation

While operations are expressed in a fixed order, it might be more efficient to rearrange them.  This will be especially important for external data sources, since we can typically only do one pass in the external application before we get results back into R. 

    arrange   + arrange   -> MERGE
    arrange   + mutate    ->
    arrange   + subset    -> WHERE + ORDER BY
    arrange   + summarise -> 
    mutate    + arrange   ->
    mutate    + mutate    -> MERGE
    mutate    + subset    -> SELECT + WHERE
    mutate    + summarise -> END
    subset    + arrange   ->
    subset    + mutate    ->
    subset    + subset    -> MERGE
    subset    + summarise ->
    summarise + arrange   ->
    summarise + mutate    -> INTERPOLATE?
    summarise + subset    -> SELECT + HAVING / WHERE
    summarise + summarise -> MERGE (implicit AND)

    ops <- format(c("subset", "summarise", "mutate", "arrange"))
    cat(with(expand.grid(ops, ops), paste(Var1, "+", Var2, collapse = "\n")))

Need way to describe pattern matching to simplify. And standard method for running. Differs between data.frame (just apply each method in sequence, always avaialble as fallback), data.table, and sql.

## Some examples

    # Equivalent to:
    source(deaths) +
      filters(is.na(hod)) +
      by(var(cod, hod), summarises(freq = count()) +
      arrange(desc(freq)) +
      by(var(cod), transforming(prop = freq / sum(freq)) 

    source(deaths) +
      counting(by_var(cod, hod)) +
      filters(is.na(hod)) +
      groups(by_var(cod), 
        groups(by_var(hod), summarises(freq = count()) +
        transforming(prop = freq / sum(freq)) +
      arranging(desc(freq))

    baseball <- source(baseball)
    baseball + filters(year < 1900)
    baseball + by(id, filters(all(year < 1900)))
    baseball + by(id, filters(length(id) > 10)))

    baseball + 
      by(id, transforming(cyear = year - min(year) + 1)) +
      subset(cyear == 1)

    baseball + 
      by(id, {
        transforming(cyear = year - min(year) + 1)) + 
        filters(cyear == 1)
      })
