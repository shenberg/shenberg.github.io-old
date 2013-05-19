---
layout: post
category : interesting
tagline: ""
tags : [canvas, chaos]
---
{% include JB/setup %}

The [Chaos Game](http://en.wikipedia.org/wiki/Chaos_game) is a super-simple way to generate fractals. In its most simple form, it's just a 
Mess around with it:
<div>
  <label for="range-in">Ratio:</label>
  <input type="range" id="range-in" min="0" max="1" step="0.01" onchange="rangeout.value=value" />
  <output id="rangeout">0.5</output>
  <label for="range-in">Amout of sides:</label>
  <input type="range" id="ngon-in" min="3" max="16" step="1" onchange="ngonout.value=value" value="3"/>
  <output id="ngonout">3</output>
  <div>
    <button class="btn" id="play">Generate</button>
  </div>
</div>
<canvas id='chaos' width='600' height='600'></canvas>

<script type="text/javascript">
//<![CDATA[|
var canvas = document.getElementById("chaos"),
button = document.getElementById("play");

function generate_ngon(n, width) {
  var points = Array(),
      half_width = width/2;
  for (var i = 0; i < n; i++) {
    points.push(half_width - 1 - Math.sin((i/n)*Math.PI*2)*half_width);
    points.push(half_width - 1 - Math.cos((i/n)*Math.PI*2)*half_width);
  }
  return points;
}

function basic_(){
  var ratio = document.getElementById("range-in").value,
    n = document.getElementById("ngon-in").value,
    points = generate_ngon(n, canvas.width),
    ctx = canvas.getContext('2d'),
    prevX = Math.floor(Math.random()*600),
    prevY = Math.floor(Math.random()*600);
  ctx.clearRect(0,0,600,600);
  ctx.fillStyle = 'rgba(0,0,0,128)';
  // skip the first 100 points
  for (var i = 0; i < 100; i++) {
    var randomIndex = Math.floor(Math.random()*points.length/2) * 2,
        prevX = prevX*ratio + (1-ratio)*points[randomIndex],
        prevY = prevY*ratio + (1-ratio)*points[randomIndex+1];
  }

  for (var i = 0; i < 100000; i++) {
    var randomIndex = Math.floor(Math.random()*points.length/2) * 2,
        prevX = prevX*ratio + (1-ratio)*points[randomIndex],
        prevY = prevY*ratio + (1-ratio)*points[randomIndex+1];
    ctx.fillRect(prevX,prevY,1,1);
  }
}

button.addEventListener('click', do_some_chaos);
do_some_chaos();
//]]>
</script>
