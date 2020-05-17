<script>
export let pointerInfo = {
    dragging: false,
    canvasX: 0,
    canvasY: 0,
}
export let imageLayer; 
$: xMax = (imageLayer.position.x + imageLayer.position.w).toFixed(2);
$: yMax = (imageLayer.position.y + imageLayer.position.h).toFixed(2);
$: xMin = (imageLayer.position.x).toFixed(2);
$: yMin = (imageLayer.position.y).toFixed(2);
$: canvasX = (pointerInfo.canvasX).toFixed(2);
$: canvasY = (pointerInfo.canvasY).toFixed(2); 
</script>

<div id='debug'>
    <p>Dragging: {pointerInfo.dragging}</p>
    <p>CanvasX: {canvasX}</p>
    <p>CanvasY: {canvasY}</p>
    <p>Image xMin: {xMin}</p>
    <p>Image yMin: {yMin}</p>
    <p>Image xMax: {xMax}</p>
    <p>Image yMax: {yMax}</p>
    <p>Pointer in image: {pointerInfo.pointerInImage}</p>
</div>

<style>
#debug {
    position: fixed;
    text-align: left;
    left: 5px;
    top: 50%;
    width: 200px;
    background: #aaaaaa20;
    line-height: 9px;
    font-size: 12px;
    font-family: monospace;
}
p::before {
    content: '# ';
    margin-left: 15px;
}
</style>