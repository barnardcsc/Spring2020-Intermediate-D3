# Intermediate Data Visualization with D3
## Computational Science Center
### Barnard College

This workshop is an intermediate level guide to visualizing data with D3 (Data Driven Documents). It is by no means comprehensive, but is mean to go beyond the basics of binding data to svg elements. It is a follow-up to the CSC's [Introduction to D3](https://github.com/Barnard-Computational-Science-Center/intro-d3-fall19) workshop, but anyone with a basic understanding of HTML and javascript would be able to follow it. We assume that you understand method chaining (i.e. .append(), .attr(), .enter()) and basic D3 syntax. 

Our Intro to D3 workshop ends with rectangles bound to data points, in this case showing the median household income (2018) in each of New York's five boroughs. While the rectangles represent the data points, it is a stretch to call this a "graph." In this workshop, we will move from rectangles to an actual interactive chart. 

#### 1 Picking up where we left off: a bar chart using json data
We'll begin with the following code. The rectangles each represent a borough, and their heights represent the median household income in 2018. When you hover over the bars, they change color. 

```html
<html>
    <head>
        <title>D3 Workshop</title>
        <script src="https://d3js.org/d3.v5.min.js"></script>
    </head>
    <body>
        <script type = "text/javascript">
            d3.json("data.json").then(function(data) {
                var svg = d3.select("body").append("svg")
                .attr('width', 960)
                .attr('height', 500);

                var rectangles = svg.selectAll("rect")
                .data(data);
            rectangles.enter()
                .append("rect")
                .attr("x", function(d,i) {
                    return i*30
                })
                .attr("y", function(d,i) {
                    return 500 - d.med_2018/1000;
                })
                .attr("width", 25)
                .attr("height", function(d,i) {
                    return d.med_2018/1000;
                })
                .attr("fill", "lightblue")
                .on("mouseover", handleMouseOver)
                .on("mouseout", handleMouseOut)
            })

            function handleMouseOver(d,i) {
                d3.select(this)
                    .attr("fill", "black")
            }
            function handleMouseOut(d,i) {
                d3.select(this)
                    .attr("fill", "lightblue")
            }
        </script>
    </body>
</html>
```
There are quite a few improvements to be made. A few key needs:
- Axes, scaled to our data
- Axes labels
- Speaking of scaling, it would be good to scale the rectangles automatically, instead of manually dividing them by 1,000 and setting a width of 25, as we do now
- To make things cleaner, we'll separate the javascript into its own script file and reference it as a script the way we do with the d3 library in the header above.

#### 2 Making it look like a bar chart

**Separating Files**
First, let's separate out our javascript into its own file called "d3-script.js". We reference the file in the head of our html file.

```html
<html>
    <head>
        <title>D3 Workshop</title>
        <script src="https://d3js.org/d3.v5.min.js"></script>
        <script src="d3-script.js"></script>
    </head>
    <body>
    </body>
</html>
```

```javascript
d3.json("data.json").then(function(data) {
    var svg = d3.select("body").append("svg")
    .attr('width', 960)
    .attr('height', 500);

    var rectangles = svg.selectAll("rect")
    .data(data);
rectangles.enter()
    .append("rect")
    .attr("x", function(d,i) {
        return i*30
    })
    .attr("y", function(d,i) {
        return 500 - d.med_2018/1000;
    })
    .attr("width", 25)
    .attr("height", function(d,i) {
        return d.med_2018/1000;
    })
    .attr("fill", "lightblue")
    .on("mouseover", handleMouseOver)
    .on("mouseout", handleMouseOut)
})

function handleMouseOver(d,i) {
    d3.select(this)
        .attr("fill", "black")
}
function handleMouseOut(d,i) {
    d3.select(this)
        .attr("fill", "lightblue")
}
```

**Adding Scaled Axes**
From here, we'll remove what's in the d3.json function. We create a new margins array to add padding around the chart, and use those to set the size of the svg element.

.scaleBand() and .scaleLinear are D3 scaling functions. The former is for ordinal scales; we use it for the Boroughs, since they are non-linear. The latter is for linear scales, so we use it for the median household income. We scale them both between 0 and width/height. The y scale has to be from height to 0 because the coordinates begin at the top of the page.

the .domain() functions map the scales to our variables (Borough and med_2018).

Finally, we create a group element "g," append it to the svg element, and append an axisBottom (x axis) and axisLeft (y axis) to it, along with appropriate labels. 

``` javascript 
d3.json("data.json").then(function(data) {

    var margin = {
            top:50,
            right:50,
            bottom:50,
            left:50},
        width = 900,
        height = 500;

    var svg = d3.select("body").append("svg")
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom);

    var x = d3.scaleBand()
        .rangeRound([0, width])
        .padding(0.1);

    var y = d3.scaleLinear()
        .rangeRound([height, 0]);

    x.domain(data.map(function(d,i) {
        return d.Borough;
    }))

    y.domain([0,d3.max(data, function (d,i) {
        return d.med_2018;
    })]);

    var g = svg.append("g")
                .attr("transform", "translate(" + margin.left + "," + margin.top + ")");
    g.append("g")
        .attr("transform", "translate(0," + height + ")")
        .call(d3.axisBottom(x))
    g.append("g")
        .call(d3.axisLeft(y))
        .append("text")
        .attr("fill", "black")
        .attr("transform", "rotate(-90)")
        .attr("y", 6)
        .attr("dy", "0.71em")
        .attr("text-anchor", "end")
        .text("Median Household Income ($)");
})
```

**Adding Rectangles Back In, Scaled**

```javascript
d3.json("data.json").then(function(data) {

    var margin = {
            top:50,
            right:50,
            bottom:50,
            left:50},
        width = 900,
        height = 500;

    var svg = d3.select("body").append("svg")
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom);

    var x = d3.scaleBand()
        .rangeRound([0, width])
        .padding(0.1);

    var y = d3.scaleLinear()
        .rangeRound([height, 0]);

    x.domain(data.map(function(d,i) {
        return d.Borough;
    }))

    y.domain([0,d3.max(data, function (d,i) {
        return d.med_2018;
    })]);

    var g = svg.append("g")
                .attr("transform", "translate(" + margin.left + "," + margin.top + ")");
    g.append("g")
        .attr("transform", "translate(0," + height + ")")
        .call(d3.axisBottom(x))
    g.append("g")
        .call(d3.axisLeft(y))
        .append("text")
        .attr("fill", "black")
        .attr("transform", "rotate(-90)")
        .attr("y", 6)
        .attr("dy", "0.71em")
        .attr("text-anchor", "end")
        .text("Median Household Income ($)");

    g.selectAll("bar")
        .data(data)
        .enter().append("rect")
        .attr("class", "bar")
        .attr("x", function (d,i) {
            return x(d.Borough);
        })
        .attr("y", function(d,i) {
            return y(d.med_2018);
        })
        .attr("width", x.bandwidth())
        .attr("height", function (d,i) {
            return height - y(d.med_2018);
        })
        .attr("fill", "lightblue")
})
```

#### 3 Mouse events and tooltips
In the introductory workshop we covered basic mouse events like mouseover and mouseout to change the color of the bars. We'll add those back into this chart, and use a new event, mousemove, to add an html tooltip to the bars. This will take the form of a box that follows the mouse displaying the exact value of each rectangle when hovered. To create the tooltip, we append a div of class "tooltip" to the svg, which we then style in the style tag in our html file. To make the tooltips appear, we only need to make them visible.

```html
<html>
    <head>
        <title>D3 Workshop</title>
        <script src="https://d3js.org/d3.v5.min.js"></script>
        <script src="d3-script.js"></script>
    </head>
    <style>
        .tooltip{
            position: absolute;
            text-align: center;
            width: 70px;
            height: 22px;
            padding: 6px;
            font: 10px sans-serif;
            background: white;
            opacity: 0.9;
            pointer-events: none;
        }
    </style>
    <body>
    </body>
</html>
```

```javascript
d3.json("data.json").then(function(data) {

    var margin = {
            top:50,
            right:50,
            bottom:50,
            left:50},
        width = 900,
        height = 500;

    var svg = d3.select("body").append("svg")
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom);

    var tooltip = d3.select("body").append("tooltip") // appending new variable
        .attr("class", "tooltip")
        .style("display", "none");

    var x = d3.scaleBand()
        .rangeRound([0, width])
        .padding(0.1);

    var y = d3.scaleLinear()
        .rangeRound([height, 0]);

    x.domain(data.map(function(d,i) {
        return d.Borough;
    }))

    y.domain([0,d3.max(data, function (d,i) {
        return d.med_2018;
    })]);

    var g = svg.append("g")
                .attr("transform", "translate(" + margin.left + "," + margin.top + ")");
    g.append("g")
        .attr("transform", "translate(0," + height + ")")
        .call(d3.axisBottom(x))
    g.append("g")
        .call(d3.axisLeft(y))
        .append("text")
        .attr("fill", "black")
        .attr("transform", "rotate(-90)")
        .attr("y", 6)
        .attr("dy", "0.71em")
        .attr("text-anchor", "end")
        .text("Median Household Income ($)");

    g.selectAll("bar")
        .data(data)
        .enter().append("rect")
        .attr("class", "bar")
        .attr("x", function (d,i) {
            return x(d.Borough);
        })
        .attr("y", function(d,i) {
            return y(d.med_2018);
        })
        .attr("width", x.bandwidth())
        .attr("height", function (d,i) {
            return height - y(d.med_2018);
        })
        .attr("fill", "lightblue")
        .on("mouseover", handleMouseOver)
        .on("mouseout", handleMouseOut)
        .on("mousemove", handleMouseMove)

    function handleMouseOver(d,i) {
        tooltip.style("display", "inline");
        d3.select(this)
            .attr("fill", "darkblue")
    }
    function handleMouseOut(d,i) {
        tooltip.style("display", "none")
        d3.select(this)
            .attr("fill", "lightblue")
    }
    function handleMouseMove(d,i) {
        tooltip.text(d.Borough + ": $" + d3.format(",")(d.med_2018))
            .style("left", d3.event.pageX)
            .style("top", d3.event.pageY);
    }
    
})
```

#### 4 Radio buttons
At this point, your chart effectively communicates information and incorporates one of the first elements of interactivity that users look for. From here, you can use other html elements, such as buttons, menus, sliders, and text entry to manipulate your chart. 

Since our dataset has median household income for four different points in time, we'll use radio buttons to change the chart to display data from those years. There are many different ways to use radio buttons with d3 and no real comprehensive guide online. It's necessary to note here that this method is not necessarily the "best," but it was the most straightforward for this situation. 

First, we'll add the radio buttons as a form element in our html. The value is the name of the variable in the dataset. We use the word "checked" in the 2018 button to have it be automatically checked off. 

```html
<html>
    <head>
        <title>D3 Workshop</title>
        <script src="https://d3js.org/d3.v5.min.js"></script>
        <script src="d3-script.js"></script>
        <form>
            <label><input type = "radio" name = "year" value = "med_1990">1990</label>
            <label><input type = "radio" name = "year" value = "med_2000">2000</label>
            <label><input type = "radio" name = "year" value = "med_2010">2010</label>
            <label><input type = "radio" name = "year" value = "med_2018" checked>2018</label>
        </form>
    </head>
    <style>
        .tooltip{
            position: absolute;
            text-align: center;
            width: 70px;
            height: 22px;
            padding: 6px;
            font: 10px sans-serif;
            background: white;
            opacity: 0.9;
            pointer-events: none;
        }
    </style>
    <body>
    </body>
</html>
```
**Access information from the buttons**
Right now, the buttons can be pressed but nothing changes. The next step is to use the buttons to change the data appended to our rectangles in d3.

First, we need to create a variable in our function called "field" that indicates which field to visualize. We will initialize it as med_2018 but change it when the buttons are pressed. Next, we create a handler for when the "input" is changed, and tell it to reference a function called changeFunction, which we haven't yet defined. Finally, define changeFunction, use it to select the value of the input and display it in your console. Check that this is working in your browser.

``` javascript
d3.json("data.json").then(function(data) {

    var margin = {
            top:50,
            right:50,
            bottom:50,
            left:50},
        width = 900,
        height = 500;

    var svg = d3.select("body").append("svg")
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom);

    var tooltip = d3.select("body").append("tooltip")
        .attr("class", "tooltip")
        .style("display", "none");

    var field = "med_2018"; // new variable "field"

    var x = d3.scaleBand()
        .rangeRound([0, width])
        .padding(0.1);

    var y = d3.scaleLinear()
        .rangeRound([height, 0]);

    x.domain(data.map(function(d,i) {
        return d.Borough;
    }))

    y.domain([0,d3.max(data, function (d,i) {
        return d.med_2018;
    })]);

    var g = svg.append("g")
                .attr("transform", "translate(" + margin.left + "," + margin.top + ")");
    g.append("g")
        .attr("transform", "translate(0," + height + ")")
        .call(d3.axisBottom(x))
    g.append("g")
        .call(d3.axisLeft(y))
        .append("text")
        .attr("fill", "black")
        .attr("transform", "rotate(-90)")
        .attr("y", 6)
        .attr("dy", "0.71em")
        .attr("text-anchor", "end")
        .text("Median Household Income ($)");

    g.selectAll("bar")
        .data(data)
        .enter().append("rect")
        .attr("class", "bar")
        .attr("x", function (d,i) {
            return x(d.Borough);
        })
        .attr("y", function(d,i) {
            return y(d.med_2018);
        })
        .attr("width", x.bandwidth())
        .attr("height", function (d,i) {
            return height - y(d.med_2018);
        })
        .attr("fill", "lightblue")
        .on("mouseover", handleMouseOver)
        .on("mouseout", handleMouseOut)
        .on("mousemove", handleMouseMove)

    d3.selectAll("input").on("change", changeFunction);

    function handleMouseOver(d,i) {
        tooltip.style("display", "inline");
        d3.select(this)
            .attr("fill", "darkblue")
    }
    function handleMouseOut(d,i) {
        tooltip.style("display", "none")
        d3.select(this)
            .attr("fill", "lightblue")
    }
    function handleMouseMove(d,i) {
        tooltip.text(d.Borough + ": $" + d3.format(",")(d.med_2018))
            .style("left", d3.event.pageX)
            .style("top", d3.event.pageY);
    }
    function changeFunction(d,i) {
        field = this.value;
        console.log(field)
    }
})
```

**Make bars respond to buttons**
Finally, we make the heights of the bars dependent on the button selection. To do this, first replace all of the d.med_2018 items with d[field] when you create the bars. Next, in the changeFunction, we will rescale the axis and data to the new value of 'field'. Don't forget to replace d.med_2018 with d[field] in your handleMouseMove function, or your tooltips won't change with the graph!

Finally, adding a duration to the transition in changeFunction adds a nice lag for the user to see your bars change. 

Final html and javascript files:

```html
<html>
    <head>
        <title>D3 Workshop</title>
        <script src="https://d3js.org/d3.v5.min.js"></script>
        <script src="d3-script.js"></script>
        <form>
            <label><input type = "radio" name = "year" value = "med_1990">1990</label>
            <label><input type = "radio" name = "year" value = "med_2000">2000</label>
            <label><input type = "radio" name = "year" value = "med_2010">2010</label>
            <label><input type = "radio" name = "year" value = "med_2018" checked>2018</label>
        </form>
    </head>
    <style>
        .tooltip{
            position: absolute;
            text-align: center;
            width: 70px;
            height: 22px;
            padding: 6px;
            font: 10px sans-serif;
            background: white;
            opacity: 0.9;
            pointer-events: none;

        }
    </style>
    <body>
    </body>
</html>
```

```javascript
d3.json("data.json").then(function(data) {

    var margin = {
            top:50,
            right:50,
            bottom:50,
            left:50},
        width = 900,
        height = 500;

    var svg = d3.select("body").append("svg")
        .attr('width', width + margin.left + margin.right)
        .attr('height', height + margin.top + margin.bottom);

    var tooltip = d3.select("body").append("tooltip")
        .attr("class", "tooltip")
        .style("display", "none");

    var field = "med_2018";

    var x = d3.scaleBand()
        .rangeRound([0, width])
        .padding(0.1);

    var y = d3.scaleLinear()
        .rangeRound([height, 0]);

    x.domain(data.map(function(d,i) {
        return d.Borough;
    }))

    y.domain([0,d3.max(data, function (d,i) {
        return d[field];
    })]);

    var g = svg.append("g")
                .attr("transform", "translate(" + margin.left + "," + margin.top + ")");
    g.append("g")
        .attr("transform", "translate(0," + height + ")")
        .call(d3.axisBottom(x))
    g.append("g")
        .call(d3.axisLeft(y))
        .append("text")
        .attr("fill", "black")
        .attr("transform", "rotate(-90)")
        .attr("y", 6)
        .attr("dy", "0.71em")
        .attr("text-anchor", "end")
        .text("Median Household Income ($)");

    g.selectAll("bar")
        .data(data)
        .enter().append("rect")
        .attr("class", "bar")
        .attr("x", function (d,i) {
            return x(d.Borough);
        })
        .attr("y", function(d,i) {
            return y(d[field]);
        })
        .attr("width", x.bandwidth())
        .attr("height", function (d,i) {
            return height - y(d[field]);
        })
        .attr("fill", "lightblue")
        .on("mouseover", handleMouseOver)
        .on("mouseout", handleMouseOut)
        .on("mousemove", handleMouseMove)

    d3.selectAll("input").on("change", changeFunction);

    function handleMouseOver(d,i) {
        tooltip.style("display", "inline");
        d3.select(this)
            .attr("fill", "darkblue")
    }
    function handleMouseOut(d,i) {
        tooltip.style("display", "none")
        d3.select(this)
            .attr("fill", "lightblue")
    }
    function handleMouseMove(d,i) {
        tooltip.text(d.Borough + ": $" + d3.format(",")(d[field]))
            .style("left", d3.event.pageX)
            .style("top", d3.event.pageY);
    }
    function changeFunction(d,i) {
        field = this.value;
        console.log(field)
        y.domain([0,d3.max(data, function (d,i) {
            return d[field];
        })]);
        g.transition()
            .selectAll(".bar")
            .attr("y", function(d,i) {
                return y(d[field])
            })
            .attr("height", function(d,i) {
                return height - y(d[field])
            })
    }
})
```
