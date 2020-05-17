<script>

const TO_CM = 2.54
export let imageLayer; 

export let state;


$: imageWidth = (imageLayer.rect.w).toFixed(2);
$: imageWidthCM = ((imageLayer.image ? imageLayer.image.width : 0) / state.dpi * TO_CM).toFixed(2)
</script>

<div id='debug'>
    <p>DPI: {state.dpi}</p>
    <p>Image width (px): {imageWidth}</p>
    <p>Image width (cm): {imageWidthCM}</p>
</div>

<style>
#debug {
    position: fixed;
    text-align: left;
    left: 5px;
    top: 50%;
    width: 240px;
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