---
title: Writing in BEM using LESS
date: 2017/08/05 19:19:00
tags: javascript, web api
---

This is something I found myself thinking about when using the Block Element Module (BEM) methodology when coding using the preprocessor LESS.

I like this for the follow reasons:

* Less repeating code
* Ability to change the block name and auto cascade to the element descendants
* Less lines and easier to read

#### The Code

{% codeblock lang:css %}
    block {
      &__element {
        &--modifier-alpha {
           color: blue;
        }
        &--modifier--beta {
           color: green;
        }
      }
    }
{% endcodeblock %}

So the compiled elements would look like this:

{% codeblock lang:css %}
    block__element--modifier-alpha {
      color: blue;
    }
    block__element--modifier-beta {
      color: green;
    }
{% endcodeblock %}

