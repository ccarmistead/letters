## Letter-Writing in the Anthropocene

# Welcome

[subheader here - or introductory intro]

## Intro

[text]

## Other


## Upload a letter


### Other

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

<!DOCTYPE html>
<meta charset="utf-8">
<style>
#toggle {
    position: absolute;
    right: 15px; top: 15px;
    z-index: 100;
}

.map path {
    fill: #eee;
    stroke: #fff;
    stroke-width: 2;
}

.links line {
    stroke: #679;
    stroke-opacity: 0.25;
}

.nodes circle {
    fill: #679;
    stroke: #235;
    stroke-width: 2;
}
</style>
<body>
<button id="toggle">Toggle Map</button>
<script src="//d3js.org/d3.v4.min.js"></script>
<script>
var positioning = 'map'

var width = 960
var height = 600

var projection = d3.geoAlbersUsa()
    .scale([width * 1.25])
    .translate([width / 2, height / 2])

var path = d3.geoPath().projection(projection)

var linkForce = d3.forceLink()
    .id(function (d) { return d.id })
    .distance(40)

var simulation = d3.forceSimulation()
    .force('link', linkForce)
    .force('charge', d3.forceManyBody().strength(-160))
    .force('center', d3.forceCenter(width / 2, height / 2))
    .stop()

var drag = d3.drag()
    .on('start', dragStarted)
    .on('drag', dragged)
    .on('end', dragEnded)

d3.queue()
    .defer(d3.json, 'data.json')
    .defer(d3.json, 'continental-us.json')
    .awaitAll(initialize)

function initialize(error, results) {
    if (error) { throw error }

    var graph = results[0]
    var features = results[1].features

    simulation.nodes(graph.nodes)
        .on('tick', ticked)

    simulation.force('link').links(graph.links)

    var svg = d3.select('body')
        .append('svg')
        .attr('width', width)
        .attr('height', height)

    var map = svg.append('g')
        .attr('class', 'map')
        .selectAll('path')
        .data(features)
        .enter().append('path')
        .attr('d', path)

    var links = svg.append('g')
        .attr('class', 'links')
        .selectAll('line')
        .data(graph.links)
        .enter().append('line')
        .attr('stroke-width', function (d) { return d.count / 4 })

    var nodes = svg.append('g')
        .attr('class', 'nodes')
        .selectAll('circle')
        .data(graph.nodes)
        .enter().append('circle')
        .attr('r', 5)
        .call(drag)

    nodes.append('title')
        .text(function (d) { return d.faa })

    fixed(true)
    d3.select('#toggle').on('click', toggle)

    function fixed(immediate) {
        graph.nodes.forEach(function (d) {
            var pos = projection([d.lon, d.lat])
            d.x = pos[0]
            d.y = pos[1]
        })

        var t = d3.transition()
            .duration(immediate ? 0 : 600)
            .ease(d3.easeElastic.period(0.5))

        update(links.transition(t), nodes.transition(t))
    }

    function ticked() {
        update(links, nodes)
    }

    function update(links, nodes) {
        links
            .attr('x1', function (d) { return d.source.x })
            .attr('y1', function (d) { return d.source.y })
            .attr('x2', function (d) { return d.target.x })
            .attr('y2', function (d) { return d.target.y })

        nodes
            .attr('cx', function (d) { return d.x })
            .attr('cy', function (d) { return d.y })
    }

    function toggle() {
        if (positioning === 'map') {
            positioning = 'sim'
            map.attr('opacity', 0.25)
            simulation.alpha(1).restart()
        } else {
            positioning = 'map'
            map.attr('opacity', 1)
            simulation.stop()
            fixed()
        }
    }
}

function dragStarted(d) {
    if (positioning === 'map') { return }
    simulation.alphaTarget(0.3).restart()
    d.fx = d.x
    d.fy = d.y
}

function dragged(d) {
    if (positioning === 'map') { return }
    d.fx = d3.event.x
    d.fy = d3.event.y
}

function dragEnded(d) {
    if (positioning === 'map') { return }
    simulation.alphaTarget(0)
    d.fx = null
    d.fy = null
}

</script>
</body>
