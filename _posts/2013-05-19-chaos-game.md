---
layout: post
category : interesting
tagline: ""
---
{% include JB/setup %}
<!-- Hack -->

The [Chaos Game](http://en.wikipedia.org/wiki/Chaos_game) is a super-simple way to generate fractals. In its most simple form, just create a regular triangle, square (, 4-gon, 5-gon, ..., n-gon), choose a random point in it and a ratio smaller than 1. Then, repeatedly take the point at _ratio_ between the current point and a randomly selected vertex of the polygon.

Throw away the first few points and you may get a fractal. 

Seeing as it's that simple, I whipped up a quick canvas implementation:
<!-- more -->
<form>
  <div class="clearfix">
    <label for="range-in">Ratio:</label>
    <div class="input">
      <input type="range" id="range-in" min="0" max="1" step="0.01" onchange="rangeout.value=value" />
      <output id="rangeout">0.5</output>
    </div>
  </div>
  <div class="clearfix">
    <label for="ngon-in">Amout of sides:</label>
    <div class="input">
      <input type="range" id="ngon-in" min="3" max="16" step="1" onchange="ngonout.value=value" value="3"/>
      <output id="ngonout">3</output>
    </div>
  </div>
  <div class="clearfix">
    <label for="points-in">Number of points to draw:</label>
    <div class="input">
      <input type="range" id="points-in" min="5000" max="500000" step="4950" onchange="pointsout.value=value" value="20000"/>
      <output id="pointsout">20000</output>
    </div>
  </div>
  <div class="clearfix">
    <button class="btn" type="button" id="play" style="float:left">Generate</button> 
    <div class="input">
      <span>Share: </span><input type="text" class="input-xlarge" style="margin-bottom: 0" id="share-link" onclick="this.select();" />
    </div>
  </div>
</form>
<canvas id='chaos' width='600' height='600'>
</canvas>

<script type="text/javascript">
//<![CDATA[|
var canvas = document.getElementById("chaos"),
    button = document.getElementById("play");

function params_from_hash() {
  if (location.hash) {
    var valueStrings = location.hash.split('#')[1].split(','),
        ratio = parseFloat(valueStrings[0]),
        n = parseInt(valueStrings[1]),
        points = parseInt(valueStrings[2]);

    if ((ratio >= 0 ) && (ratio <= 1)) {
      document.getElementById("range-in").value = ratio;
      document.getElementById("rangeout").value = ratio;
    }
    if ((points >= 5000 ) && (points <= 500000)) {
      document.getElementById("points-in").value = points;
      document.getElementById("pointsout").value = points;
    }
    if ((n >= 3 ) && (n <= 16)) {
      document.getElementById("ngon-in").value = n;
      document.getElementById("ngonout").value = n;
    }
  }
}

function generate_ngon(n, width, height) {
  var points = Array(),
      half_width = width/2,
      half_height = height/2;
  for (var i = 0; i < n; i++) {
    points.push(half_width - 1 - Math.sin((i/n)*Math.PI*2)*half_width);
    points.push(half_height - 1 - Math.cos((i/n)*Math.PI*2)*half_height);
  }
  return points;
}


function do_some_chaos() {
  var ratio = document.getElementById("range-in").value,
      n = document.getElementById("ngon-in").value,
      point_count = document.getElementById("points-in").value,
      points = generate_ngon(n, canvas.width, canvas.height),
      ctx = canvas.getContext('2d'),
      prev_x = Math.floor(Math.random()*canvas.width), // random initial point in the canvas
      prev_y = Math.floor(Math.random()*canvas.height); // strictly speaking, it should be inside the polygon, but this is good enough in practice

  window.location.hash = "#" + ratio + ',' + n + ',' + point_count;
  document.getElementById('share-link').value = window.location;

  ctx.clearRect(0, 0, canvas.width, canvas.height);
  ctx.fillStyle = 'rgba(0,0,0,128)';

  var start_time = +new Date();
  
  // skip the first 100 points
  for (var i = 0; i < 100; i++) {
    var randomIndex = Math.floor(Math.random()*points.length/2) * 2,
        prev_x = prev_x*ratio + (1-ratio)*points[randomIndex],
        prev_y = prev_y*ratio + (1-ratio)*points[randomIndex+1];
  }

  for (var i = 0; i < point_count - 100; i++) {
    var randomIndex = Math.floor(Math.random()*points.length/2) * 2,
        prev_x = prev_x*ratio + (1-ratio)*points[randomIndex],
        prev_y = prev_y*ratio + (1-ratio)*points[randomIndex+1];
    ctx.fillRect(prev_x,prev_y,1,1);
  }

  console.log("Took " + (((new Date()) - start_time) / 1000) + " seconds to run");
}

button.addEventListener('click', do_some_chaos, false);

params_from_hash();
do_some_chaos();
//]]>
</script>

My friend Imri wrote [a short post](http://www.algorithm.co.il/blogs/programming/fractals-in-10-minutes-no-5-sierpinski-chaos-game/) on using the chaos game to generate the Sierpinski triangle a good five years ago (before canvas was a thing, really). It's a bit pleasing and worrying at the same time: past me still liked interesting things, but I had no recollection of the matter and it was only five years ago that I had read his post.

If this were the only thing that falls under the purview of the chaos game, it would be cute. The next part, generalizing to IFS, is where things start getting really fun.