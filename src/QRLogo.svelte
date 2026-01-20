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

    // Draw center logo on top of QR code so it's visible
    drawCenterLogo(ctx, centerX, centerY);

    console.log('QRLogo: Composite complete');
  }

  function drawCenterLogo(ctx, centerX, centerY) {
    if (!frameConfig.centerLogoShape || frameConfig.centerLogoShape === 'none') return;

    ctx.fillStyle = frameConfig.centerLogoColor || '#000000';
    ctx.strokeStyle = frameConfig.centerLogoColor || '#000000';
    const logoSize = frameConfig.centerLogoSize || 60;

    if (frameConfig.centerLogoShape === 'circles') {
      ctx.lineWidth = 8;
      ctx.beginPath();
      ctx.arc(centerX, centerY, logoSize, 0, Math.PI * 2);
      ctx.stroke();

      ctx.lineWidth = 6;
      ctx.beginPath();
      ctx.arc(centerX, centerY, logoSize * 0.65, 0, Math.PI * 2);
      ctx.stroke();

      ctx.beginPath();
      ctx.arc(centerX, centerY, logoSize * 0.3, 0, Math.PI * 2);
      ctx.fill();
    } else if (frameConfig.centerLogoShape === 'square') {
      ctx.save();
      ctx.translate(centerX, centerY);
      ctx.rotate(Math.PI / 4);

      ctx.lineWidth = 6;
      ctx.strokeRect(-logoSize, -logoSize, logoSize * 2, logoSize * 2);
      ctx.fillRect(-logoSize * 0.6, -logoSize * 0.6, logoSize * 1.2, logoSize * 1.2);

      ctx.restore();
    } else if (frameConfig.centerLogoShape === 'star') {
      ctx.beginPath();
      for (let i = 0; i < 10; i++) {
        const radius = i % 2 === 0 ? logoSize : logoSize * 0.4;
        const angle = (Math.PI / 5) * i - Math.PI / 2;
        const x = centerX + radius * Math.cos(angle);
        const y = centerY + radius * Math.sin(angle);
        if (i === 0) ctx.moveTo(x, y);
        else ctx.lineTo(x, y);
      }
      ctx.closePath();
      ctx.fill();
    } else if (frameConfig.centerLogoShape === 'diamond') {
      ctx.lineWidth = 8;
      ctx.beginPath();
      ctx.moveTo(centerX, centerY - logoSize * 2);
      ctx.lineTo(centerX + logoSize * 2, centerY);
      ctx.lineTo(centerX, centerY + logoSize * 2);
      ctx.lineTo(centerX - logoSize * 2, centerY);
      ctx.closePath();
      ctx.stroke();

      ctx.beginPath();
      ctx.moveTo(centerX, centerY - logoSize);
      ctx.lineTo(centerX + logoSize, centerY);
      ctx.lineTo(centerX, centerY + logoSize);
      ctx.lineTo(centerX - logoSize, centerY);
      ctx.closePath();
      ctx.fill();
    }
  }
</script>

<canvas bind:this={canvas} style="display: none;"></canvas>
