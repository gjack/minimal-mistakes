---
title: "TIL: Working with a D3 based chart library"
categories:
  - "JavaScript"
  - "Quick Tip"
  - "Web Development"
---

One of the things I love about my job is that I get to explore different aspects of web development, including all sorts of interesting libraries that otherwise I may have never had the need to use.

This time, I was working with a chart that is generated using [C3.js](http://c3js.org/), a D3-based chart library that is super cool, very well documented and really easy to use.

Using this library, you can quickly generate a beautiful chart such as this:

<img src="/assets/images/posts/D3-image1.png" width="300" class="align-center">

Using simple code that would look something like this:
```javascript
var chart = c3.generate({
    data: {
        columns: [
            ['data1', 30, 200, 100, 400, 150, 250],
            ['data2', 130, 100, 140, 200, 150, 50],
            ['data3', 150, 80, 120, 250, 50, 10]
        ],
        type: 'bar'
    },
    bar: {
        width: {
            ratio: 0.5
        }
    }
});
```

Notice how each one of the columns corresponds to a legend for a group in the bar chart above.  The default behavior for these charts is really cool too; if you click on one of the legends, the corresponding group disappears from view.

<img src="/assets/images/posts/D3-image2.png" width="300" class="align-center">

My assignment was to extract information about what groups remained visible on the chart and then use that information to calculate an average.

As I mentioned before, this library is very well documented, yet it wasn’t really clear what I needed to do in order to obtain the information I needed. After some digging and a bit of trial and error, I realized it was possible to attach a handler to the click event of a legend, provided we make sure to toggle the visibility of the group in the handler. We can add something like this:

```javascript
legend: {
     item: {
       onclick: function(id) {
           chart.toggle(id)
           console.log(chart.data.shown())
       }  
     }    
   }
```
In the code above, chart.data.shown() returns an array of objects representing the elements/groups currently visible in the chart. The id passed to the function corresponds to the first element in each of the columns we passed to the chart as data. Here, I’m simply logging the array of visible elements to the console, but we can do just about anything else we need with this data. We can save it to a variable or, as I did, in the state of the React component that renders this chart. And that’s it! Really simple!

<img src="/assets/images/posts/D3-image3.png" width="300" class="align-center">
