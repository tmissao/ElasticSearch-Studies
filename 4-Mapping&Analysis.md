# Mapping & Analysis

## Analysis
---
Before the document is saved on Elastic, it is analysed to defined the best way to persist the data and how to search it.

The process of analysis of document is called `Analyzer` and can divided into 3 parts:

- `Character Filters` - Adds, removes or changes characters ( Can have zero or more).
```
# *html_strip* - removes html elements
Input: "I&apos;m in a <em>good</em> mood&nbsp; and I <strong>love</strong> açaí!"

Output: "I'm in a good mood - and I love açaí"
```
- `Tokenizer` - Splits a String into tokens. (Just 1 is allowed)
```
Input: "I REALLY like beer!"

Output: ["I", "REALLY", "like", "beer"]
```
- `Token Filters` - Receives tokens from tokenizer then add, remove or modify tokens (Can have zero or more token filters)
```
# LowerCase Token Filter
Input: ["I", "REALLY", "like", "beer"]

Output: ["i", "really", "like", "beer"]
```

### Using the API
```bash
POST /_analyze
{
  "analyzer": "standard",
  "text": "2 guys walk into    a bar, but the third... DUCKS! :--)"  
}
```


## Utils
---

- [Analyzer APis](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-analyze.html)
