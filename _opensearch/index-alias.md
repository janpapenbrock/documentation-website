---
layout: default
title: Index aliases
nav_order: 12
---

# Index aliases

An alias is a virtual index name that can point to one or more indexes.

If your data is spread across multiple indexes, rather than keeping track of which indexes to query, you can create an alias and query it instead.

For example, if you’re storing logs into indexes based on the month and you frequently query the logs for the previous two months, you can create a `last_2_months` alias and update the indexes it points to each month.

Because you can change the indexes an alias points to at any time, referring to indexes using aliases in your applications allows you to reindex your data without any downtime.

---

#### Table of contents
1. TOC
{:toc}


---

## Create aliases

To create an alias, use a POST request:

```json
POST _aliases
```

Use the `actions` method to specify the list of actions that you want to perform. This command creates an alias named `alias1` and adds `index-1` to this alias:

```json
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "index-1",
        "alias": "alias1"
      }
    }
  ]
}
```

You should see the following response:

```json
{
   "acknowledged": true
}
```

If this request fails, make sure the index that you're adding to the alias already exists.

To check if `alias1` refers to `index-1`, run the following command:

```json
GET alias1
```

## Add or remove indexes

You can perform multiple actions in the same `_aliases` operation.
For example, the following command removes `index-1` and adds `index-2` to `alias1`:

```json
POST _aliases
{
  "actions": [
    {
      "remove": {
        "index": "index-1",
        "alias": "alias1"
      }
    },
    {
      "add": {
        "index": "index-2",
        "alias": "alias1"
      }
    }
  ]
}
```

The `add` and `remove` actions occur atomically, which means that at no point will `alias1` point to both `index-1` and `index-2`.

You can also add indexes based on an index pattern:

```json
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "index*",
        "alias": "alias1"
      }
    }
  ]
}
```

## Manage aliases

To list the mapping of aliases to indexes, run the following command:

```json
GET _cat/aliases?v
```

#### Sample response

```json
alias     index   filter    routing.index   routing.search
alias1    index-1   *             -                 -
```

To check which indexes an alias points to, run the following command:

```json
GET _alias/alias1
```

#### Sample response

```json
{
  "index-2": {
    "aliases": {
      "alias1": {}
    }
  }
}
```

Conversely, to find which alias points to a specific index, run the following command:

```json
GET /index-2/_alias/*
```

To check if an alias exists, run the following command:

```json
HEAD /alias1/_alias/
```

## Add aliases at index creation

You can add an index to an alias as you create the index:

```json
PUT index-1
{
  "aliases": {
    "alias1": {}
  }
}
```

## Create filtered aliases

You can create a filtered alias to access a subset of documents or fields from the underlying indexes.

This command adds only a specific timestamp field to `alias1`:

```json
POST _aliases
{
  "actions": [
    {
      "add": {
        "index": "index-1",
        "alias": "alias1",
        "filter": {
          "term": {
            "timestamp": "1574641891142"
          }
        }
      }
    }
  ]
}
```

## Index alias options

You can specify the options shown in the following table.

Option | Valid values | Description | Required
:--- | :--- | :---
`index` | String | The name of the index that the alias points to. | Yes
`alias` | String | The name of the alias. | No
`filter` | Object | Add a filter to the alias. | No
`routing` | String | Limit search to an associated shard value. You can specify `search_routing` and `index_routing` independently. | No
`is_write_index` | String | Specify the index that accepts any write operations to the alias. If this value is not specified, then no write operations are allowed. | No
