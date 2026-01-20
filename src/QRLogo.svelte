<script>
  export let qrCanvas;
  export let frameCanvas;
  export let canvas;
  export let frameConfig = {};
  
  const canvasSize = 1000;
  
  export function composite() {
    if (!qrCanvas || !frameCanvas || !canvas) {
      console.error('QRLogo: Missing canvas', { 
        hasQR: !!qrCanvas, 
        hasFrame: !!frameCanvas, 
        hasCanvas: !!canvas 
      });
      return;
    }
    
    console.log('QRLogo: Compositing...', {
      qrSize: `${qrCanvas.width}x${qrCanvas.height}`,
      frameSize: `${frameCanvas.width}x${frameCanvas.height}`,
      voidSize: frameConfig.centerVoidSize
    });
    
    const ctx = canvas.getContext('2d');
    canvas.width = canvasSize;
    canvas.height = canvasSize;
    
    // Draw geometric frame first
    ctx.drawImage(frameCanvas, 0, 0);
    
    // Calculate QR placement in center void
    const centerX = canvasSize / 2;
    const centerY = canvasSize / 2;
    const qrDisplaySize = frameConfig.centerVoidSize * 0.95;
    const qrOffset = centerX - qrDisplaySize / 2;
    
    // Draw QR code on top
    ctx.drawImage(qrCanvas, qrOffset, qrOffset, qrDisplaySize, qrDisplaySize);
    
    console.log('QRLogo: Composite complete');
  }
</script>

<canvas bind:this={canvas} style="display: none;"></canvas>
