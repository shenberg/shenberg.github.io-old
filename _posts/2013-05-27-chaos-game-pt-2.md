---
layout: post
category : interesting
tagline: ""
tags : [canvas, chaos]
---
{% include JB/setup %}

In the [previous post]({% post_url 2013-05-19-chaos-game %}), the original, simplest form of the [Chaos Game](http://en.wikipedia.org/wiki/Chaos_game) was implemented on a canvas. Like all things fractal, it's a very simple process that yields interesting results. Let's dive deeper into a family of such processes.

[Iterated Function Systems](http://en.wikipedia.org/wiki/Iterated_function_system), or IFSs, are a set of functions ($$ S = \left\{f_1,f_2,...\right\} $$) that operate on points on a plane and have one important property: if we take two points and feed them both through a function, the results will be closer to one another ($$ a < 1, \left\| p_1 - p_2 \right\| \geq a \cdot \left\| f_i(p_1) - f_i(p_2) \right\| $$). A function with this property is *contractive*.

To get our fractal, all we do is start with some initial point $$ p $$ and define the set of points in the fractal to be all possible sequences of applying functions from our IFS. So, $$ p $$ is a point in the set, so is $$ f_1(p) $$, and $$ f_1(f_2(f_1(...f_n(f_2(p))...))) $$ is a member too, and so on for every combination of functions. It's here that the fact all these functions are *contractive* is important - this is what guarantees that values don't explode to infinity under repeated function applications.



Look at pt\*ratio + (1 - ratio)\*random_vertex as an affine function, mention contraction.

<div>
  <label for="points-in">Number of points to draw:</label>
  <input type="range" id="points-in" min="5000" max="300000" step="2950" onchange="pointsout.value=value" value="20000"/>
  <output id="pointsout">20000</output>
  <br>
  <button onclick="add_square()">Add Square
  </button>
  <button onclick="delete_selected()">Delete Selected Square</button>
  <span style='position: relative; display: inline-block; margin-top: 1em; border: 1px solid #444'>
    <canvas id="ifs-renderer" width="500" height="500" style="position: absolute">
    </canvas>
    <canvas id="design-overlay" width="500" height="500">
    </canvas>
  </span>
</div>
<script src="/js/fabric.js">
</script>
<script src="/js/gl-matrix.js">
</script>
<script>
//<![CDATA[|

var ifs_canvas = document.getElementById('ifs-renderer'),
    design_canvas; // initialized only after MathJAX is loaded because interactions break otherwise

var rects = [
  new fabric.Rect({width: 250, height: 250, top:125, left:250, fill: 'rgba(0,0,0,0.4)'}),
  new fabric.Rect({width: 250, height: 250, top:375, left:125, fill: 'rgba(0,0,0,0.4)'}),
  new fabric.Rect({width: 250, height: 250, top:375, left:375, fill: 'rgba(0,0,0,0.4)'}),
];

var presets = {
  sierpinski: [
    {top:125,left:250,scale_x:1,scale_y:1},
    {top:375,left:125,scale_x:1,scale_y:1},
    {top:375,left:375,scale_x:1,scale_y:1}
    ]
}

function add_square() {
  design_canvas.add(new fabric.Rect({width: 250, height: 250, top:250, left:250, fill: 'rgba(0,0,0,0.4)'}));
  do_ifs();
}

function delete_selected() {
  var selected = design_canvas.getActiveObject();
  if (selected) {
    selected.remove();
    do_ifs();
  }
}

function bring_to_front(e) {
  e.target.bringToFront();
}
// actual IFS code

function do_ifs(e) {
  //TODO: redraw
  var matrices = [],
      width = design_canvas.width,
      height = design_canvas.height;

  design_canvas.forEachObject(function(obj) {
    var x = obj.left,
        y = obj.top,
        matrix = mat2d.create();
    // change coordinate system from [0..width),[0..height) to (-0.5,0.5)^2
    // and represent the transform as a matrix
    mat2d.translate(matrix, matrix, [x/width - 0.5, y/width - 0.5]);
    // the divide by 2 is because the Rects are implicitly scaled down by a factor of 2
    // (their size is 250px/dimension, canvas is 500px/dim)
    // note: I'm increasing scale by a bit to let you manipulate smaller objects in the UI
    mat2d.scale(matrix, matrix, [obj.scaleX * 0.5, obj.scaleY * 0.5]);
    mat2d.rotate(matrix, matrix, -obj.angle*Math.PI/180);
    matrices.push(matrix);
  });
  console.log(matrices);
  chaos_ifs(matrices);
}
// use the chaos game to approximate a solution to the IFS
function chaos_ifs(matrices) {
  var point_count = document.getElementById("points-in").value,
      width = ifs_canvas.width,
      height = ifs_canvas.height,
      ctx = ifs_canvas.getContext('2d'),
      xk = vec2.fromValues(Math.random() - 0.5, Math.random() - 0.5);

  ctx.clearRect(0, 0, ifs_canvas.width, ifs_canvas.height);
  ctx.fillStyle = 'rgba(0,0,0,128)';
  var start_time = new Date().getTime();
  // skip first 10 points
  for (var i = 0; i < point_count - 100; i++) {
    var randomMatrix = matrices[(Math.random()*matrices.length) | 0];
    // x[k+1] = randomMatrix*x[k]
    xk = vec2.transformMat2d(xk, xk, randomMatrix);
  }
  for (var i = 0; i < point_count - 100; i++) {
    var randomMatrix = matrices[(Math.random()*matrices.length) | 0];
    // x[k+1] = randomMatrix*x[k]
    xk = vec2.transformMat2d(xk, xk, randomMatrix);
    // draw with some upscaling of coordinates
    ctx.fillRect((xk[0]*2+0.5)*width, (xk[1]*2+0.5)*height,1,1);
  }
  var total = (new Date().getTime()) - start_time;
  console.log("drawing time (secs): " + total/1000);
}

function params_from_hash(){

}

function on_mathjax_load() {
  design_canvas = new fabric.Canvas('design-overlay');
  design_canvas.add.apply(design_canvas, rects);
  design_canvas.on({
    'object:modified': do_ifs,
    'object:selected': bring_to_front,
  })
  params_from_hash();
  do_ifs();
}
//]]>
</script>
<script type="text/x-mathjax-config">
  MathJax.Hub.Queue(on_mathjax_load);
</script>


