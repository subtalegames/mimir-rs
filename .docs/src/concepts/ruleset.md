# Ruleset

Rulesets are collections of rules, represented in Rust as `Vec<Rule<FactKey, Outcome>>`.

```rs
struct Ruleset<FactKey, Outcome> {
    rules: Vec<Rule<FactKey, Outcome>>,
}
```

::: tip
Check out the [ruleset storage section](/guide/performance#ruleset-storage) on the performance page for further details on how Mímir represents rulesets in Rust to improve performance when evaluating queries against them.
:::

## Evaluating against queries

Just like rules, rulesets can be evaluated against queries to determine if they are true given the current game world's state:

```rs
let mut rule = Rule::new("You killed 5 enemies!");
rule.require("enemies_killed", Requirement::EqualTo(5.));

let mut more_specific_rule = Rule::new(
    "You killed 5 enemies and opened 2 doors!"
);
more_specific_rule.require("enemies_killed", Requirement::EqualTo(5.));
more_specific_rule.require("doors_opened", Requirement::gt(2.));

let ruleset = Ruleset::from(vec![rule, more_specific_rule]);

let mut query = Query::new();
query.fact("enemies_killed", 2.5 + 1.5 + 1.);

assert_eq!(
    ruleset.evaluate_all(&query)[0].outcome,
    "You killed 5 enemies!"
);

let mut more_specific_query = Query::new();
more_specific_query.fact("enemies_killed", 2.5 + 1.5 + 1.);
more_specific_query.fact("doors_opened", 10.);

assert_eq!(
    ruleset.evaluate_all(&more_specific_query)[0].outcome,
    "You killed 5 enemies and opened 2 doors!"
);
```

In the above example, we define a ruleset with two rules. Both rules require that 5 enemies have been killed, but one rule is more specific (also requiring that more than 2 doors have been opened).

The first query evaluates to the simpler rule, because the query does not satisfy the doors opened requirement. However, the second query evaluates to the more complex rule because the query *does* satistfy the doors opened requirement.

::: info
In the second query, although the simpler rule is satisfied, Mímir does not evaluate it as true because it's less specific (i.e. contains fewer requirements).
:::