---
layout: post
title:  "Welcome to Jekyll!"
date:   2018-01-11 23:24:57 +0200
categories: jekyll update
---
Hello there.

Unit testing is really important as it helps identify errors at the really early stage of development and to save our time in a long time perspective.

And for now Jest is treated as most suitable tool for React application testing. Let's take a look at few simple cases to be tested.
Here's really simple component. It's accepts single number as prop, renders it, and increases it whenever the '+' button is clicked.

{% highlight ruby %}
import React, { Component } from 'react';

export default class Counter extends Component {
  constructor({number}) {
    super();

    this.state = {
      number
    };
  }

  increaseNumber() {
    this.setState({
      number: ++this.state.number
    })
  }

  render() {
    const {
      number
    } = this.state;

    return (
      <div>
        {number}
        <button onClick={this.increaseNumber.bind(this)}>+</button>
      </div>
    )
  } 
}
{% endhighlight %}

Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/
