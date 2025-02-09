# Interactive-Dots

This code is an interactive HTML5 canvas animation that displays a grid of colorful dots that respond dynamically to user interactions (mouse or touch). Below is a detailed breakdown of the code:

---

## **1. HTML Structure**
### **`<head>` Section**
- **Title:** Sets the page title to "Interactive Dots."
- **CSS Styling:**  
  - The `#main-container` and `<canvas>` elements are styled to take up the full screen.  
  - `#infoButton`: A circular button with an information icon (`&#9432;`) positioned in the top-left corner.
  - `#infoModal`: A centered modal (popup) that provides details about the animation. It is initially hidden (`display: none`).
  - `#closeModal`: A button inside the modal that allows users to close it.

### **`<body>` Section**
- **Main Container (`#main-container`)**: Contains the canvas and UI elements.
- **Canvas (`#canvas`)**: Used to draw the animated dots.
- **Info Button (`#infoButton`)**: A button that opens a modal with instructions.
- **Info Modal (`#infoModal`)**: Displays details about how the animation works.
- **Close Button (`#closeModal`)**: Closes the modal when clicked.


## **2. JavaScript Functionality**
### **Global Variables**
```js
var $mainCont, cv, ctx, osCv, osCtx, osDotCv, osDotCtx, center, windowSize;
var mouse = { x: -888, y: -888, down: false };
var aPoints = [];
var DOT_GAP = 45;
var DOT_RADIUS = 5;
var BG_COLOR = '#000';
var DOT_COLORS = ['#e8f707', '#ff5733', '#33ff57', '#3385ff', '#ff33a8'];
var APPROACH = 80;
var JELLY = 10;
var VISCOSITY = 0.1;
```
- **Canvas References (`cv`, `ctx`)**: Main canvas and its 2D drawing context.
- **Off-screen Canvases (`osCv`, `osCtx`, `osDotCv`, `osDotCtx`)**: Used for optimized rendering.
- **`mouse` Object**: Stores the mouse position and click state.
- **`aPoints` Array**: Stores dot positions and properties.
- **Constants**:
  - `DOT_GAP`: Spacing between dots.
  - `DOT_RADIUS`: Size of each dot.
  - `BG_COLOR`: Background color (`black`).
  - `DOT_COLORS`: Array of colors used for the dots.
  - `APPROACH`: Maximum interaction distance for mouse clicks.
  - `JELLY`: Controls the elastic movement of dots.
  - `VISCOSITY`: Controls how quickly dots return to their original position.


### **Initialization (`init`)**
```js
function init() {
    createOffScreenCanvas();
    
    $mainCont = document.getElementById('main-container');
    cv = document.getElementById('canvas');
    ctx = cv.getContext('2d');
    
    setBgColor();
    
    window.onresize = resize;
    cv.addEventListener('mousedown', startDrag);
    cv.addEventListener('mousemove', onMouseMove);
    cv.addEventListener('mouseup', stopDrag);
    cv.addEventListener('mouseleave', stopDrag);
    cv.addEventListener('touchstart', startDrag);
    cv.addEventListener('touchmove', onTouchMove);
    cv.addEventListener('touchend', stopDrag);
    
    resize();
    draw();
}
```
- Initializes the canvas and event listeners.
- Calls `resize()` to adjust the canvas size.
- Calls `draw()` to start the animation loop.


### **Creating Off-Screen Canvases**
```js
function createOffScreenCanvas() {
    osCv = document.createElement('canvas');
    osCtx = osCv.getContext('2d');
    
    osDotCv = document.createElement('canvas');
    osDotCtx = osDotCv.getContext('2d');
}
```
- Creates two off-screen canvases to optimize rendering.


### **Drawing Dots**
```js
function drawDot(color) {
    var dotCanvas = document.createElement('canvas');
    var dotCtx = dotCanvas.getContext('2d');
    
    dotCanvas.width = DOT_RADIUS * 2;
    dotCanvas.height = DOT_RADIUS * 2;
    
    dotCtx.clearRect(0, 0, DOT_RADIUS * 2, DOT_RADIUS * 2);
    dotCtx.beginPath();
    dotCtx.fillStyle = color;
    dotCtx.arc(DOT_RADIUS, DOT_RADIUS, DOT_RADIUS, 0, Math.PI * 2);
    dotCtx.fill();
    
    return dotCanvas;
}
```
- Creates a separate canvas for each dot to optimize rendering.
- Uses `arc()` to draw circular dots.


### **Resizing the Canvas**
```js
function resize() {
    windowSize = { w: window.innerWidth, h: window.innerHeight };
    center = { x: Math.round(windowSize.w / 2), y: Math.round(windowSize.h / 2) };
    
    cv.width = windowSize.w;
    cv.height = windowSize.h;
    osCv.width = windowSize.w;
    osCv.height = windowSize.h;
    
    createPoints();
}
```
- Adjusts canvas size based on the window dimensions.
- Calls `createPoints()` to generate dots.


### **Creating Dots**
```js
function createPoints() {
    aPoints = [];
    for (var i = 0; i <= windowSize.w + DOT_GAP; i += DOT_GAP) {
        for (var j = 0; j <= windowSize.h + DOT_GAP; j += DOT_GAP) {
            aPoints.push(new Point(i, j, DOT_COLORS[Math.floor(Math.random() * DOT_COLORS.length)]));
        }
    }
}
```
- Generates a grid of dots and assigns them random colors.


### **Mouse and Touch Interaction**
```js
function onMouseMove(e) {
    if (mouse.down) {
        mouse.x = e.pageX;
        mouse.y = e.pageY;
    }
}

function onTouchMove(e) {
    if (mouse.down) {
        var touch = e.touches[0];
        mouse.x = touch.clientX;
        mouse.y = touch.clientY;
    }
}

function startDrag(e) {
    mouse.down = true;
    if (e.touches) {
        var touch = e.touches[0];
        mouse.x = touch.clientX;
        mouse.y = touch.clientY;
    } else {
        mouse.x = e.pageX;
        mouse.y = e.pageY;
    }
}

function stopDrag() {
    mouse.down = false;
    mouse.x = -888;
    mouse.y = -888;
}
```
- Tracks mouse and touch movement to interact with dots.


### **Animation Loop**
```js
function draw() {
    requestAnimationFrame(draw);
    render();
}

function render() {
    ctx.clearRect(0, 0, cv.width, cv.height);
    osCtx.clearRect(0, 0, osCv.width, osCv.height);
    
    for (var i = 0; i < aPoints.length; i++) {
        var p = aPoints[i];
        p.update();
        osCtx.drawImage(drawDot(p.color), p.x - DOT_RADIUS, p.y - DOT_RADIUS);
    }
    
    ctx.drawImage(osCv, 0, 0);
}
```
- Clears the canvas and updates the position of each dot before redrawing them.


### **Dot Movement**
```js
function Point(x, y, color) {
    this.oX = x + DOT_RADIUS;
    this.oY = y + DOT_RADIUS;
    this.x = x + DOT_RADIUS;
    this.y = y + DOT_RADIUS;
    this.color = color;
}

Point.prototype.update = function() {
    var distX = this.x - mouse.x;
    var distY = this.y - mouse.y;
    var distance = Math.sqrt(distX * distX + distY * distY);
    
    if (mouse.down && distance <= APPROACH) {
        var angle = Math.atan2(distY, distX);
        this.x += Math.cos(angle) * APPROACH;
        this.y += Math.sin(angle) * APPROACH;
    }
};
```
- Dots move toward the mouse when clicked.


### **Modal Logic**
```js
document.getElementById("infoButton").addEventListener("click", function() {
    document.getElementById("infoModal").style.display = "block";
});
document.getElementById("closeModal").addEventListener("click", function() {
    document.getElementById("infoModal").style.display = "none";
});
```
- Displays/hides the info modal.
