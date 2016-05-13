---
layout: post
title: recursion exercise
---

A simple problem involving recusrion. I'm curious as to how people go about tackling this. There are several approaches one could take. The one I plumped for turned out to involve quite a bit of explaining to baffled colleagues.

We're supplied with a JSON file representing the route structure of a website. Some of the nodes have content in them (flagged up by `content: true`). We want to generate a list of all valid paths to content, by crawling the JSON recursively.

For instance, given the following route structure:

```json
{
  "content": true,
  "animal": {
    "bird": {"content": true},
    "mammal": {
      "cat": {"content": true},
      "dog": {"content": true}
    },
  },
  "vegetable": {
    "flower": {
      "daisy": {"content": true}
    },
    "tree": {
      "oak": {}
    }
  },
  "mineral": {
    "content": true,
    "rock": {}
  }
}
```

`getPaths()` should return:

```json
[
  "",
  "/animal/bird",
  "/animal/mammal/cat",
  "/animal/mammal/dog",
  "/vegetable/flower/daisy",
  "/mineral",
]
```

How would you go about coding `getPaths()`? My answer, fwiw, is [here](https://gist.github.com/bat020/3274f1c296e55b3ab92c23faadb7f9e3).
