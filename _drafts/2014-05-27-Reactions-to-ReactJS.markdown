---
layout: post
title:  First reactions to facebook's ReactJS.
date:   2014-05-27
categories: learning programming js
comments: true
---

```html
<!DOCTYPE html>
<html>
  <head>
    <script src="build/react.js"></script>
    <script src="build/JSXTransformer.js"></script>
  </head>
  <body>
    <div id="example"></div>
    <script type="text/jsx">
      /** @jsx React.DOM */
      React.renderComponent(
        <h1>Hello, world!</h1>,
        document.getElementById('example')
      );
    </script>
  </body>
</html>
```
