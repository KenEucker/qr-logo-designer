<script>
  import { onMount } from 'svelte';
  
  export let url = 'https://example.com';
  export let config = {
    roundingAmount: 0.45,
    paddingAmount: 0,
    edgeBleed: 0,
    geometricChaos: 0,
    finderOuterShape: 'rounded-square',
    finderInnerShape: 'rounded-square',
    finderCenterOverlap: 0,
    centerVoidRadius: 0.15 // Percentage of canvas size
  };
  
  export let canvas;
  let qrLibraryLoaded = false;

  onMount(() => {
    // Check if library already loaded
    if (window.qrcode) {
      qrLibraryLoaded = true;
      console.log('QR library already loaded');
      return;
    }
    
    const script = document.createElement('script');
    script.src = 'https://cdn.jsdelivr.net/npm/qrcode-generator@1.4.4/qrcode.min.js';
    script.onload = () => {
      qrLibraryLoaded = true;
      console.log('QR library loaded successfully', { hasqrcode: !!window.qrcode });
    };
    script.onerror = () => {
      console.error('Failed to load QR library');
    };
    document.head.appendChild(script);
  });

  export function generateQR() {
    console.log('generateQR called', {
      hasqrcode: !!window.qrcode,
      hasCanvas: !!canvas,
      libraryLoaded: qrLibraryLoaded,
      url
    });

    if (!qrLibraryLoaded) {
      console.warn('QR library not loaded yet');
      return;
    }

    if (!window.qrcode) {
      console.error('QRCode library not available on window object');
      return;
    }

    if (!canvas) {
      console.error('Canvas not available');
      return;
    }

    try {
      // qrcode-generator API: qrcode(typeNumber, errorCorrectionLevel)
      // typeNumber 0 = auto-detect
      const qr = window.qrcode(0, 'H');
      qr.addData(url);
      qr.make();

      const moduleCount = qr.getModuleCount();
      console.log('QR generated, module count:', moduleCount);
      drawQRCode(qr, moduleCount);
    } catch (error) {
      console.error('QR generation error:', error);
    }
  }

  export function drawQRCode(qr, moduleCount) {
    const ctx = canvas.getContext('2d');
    const canvasSize = 800;
    const moduleSize = canvasSize / moduleCount;
    
    canvas.width = canvasSize;
    canvas.height = canvasSize;
    
    ctx.fillStyle = '#ffffff';
    ctx.fillRect(0, 0, canvasSize, canvasSize);
    
    const centerX = canvasSize / 2;
    const centerY = canvasSize / 2;
    const voidRadius = canvasSize * config.centerVoidRadius;
    
    ctx.fillStyle = '#000000';
    
    // Draw data modules
    for (let row = 0; row < moduleCount; row++) {
      for (let col = 0; col < moduleCount; col++) {
        if (!qr.isDark(row, col)) continue;
        
        const x = col * moduleSize;
        const y = row * moduleSize;
        const moduleCenterX = x + moduleSize / 2;
        const moduleCenterY = y + moduleSize / 2;
        
        const distFromCenter = Math.sqrt(
          Math.pow(moduleCenterX - centerX, 2) + 
          Math.pow(moduleCenterY - centerY, 2)
        );
        if (distFromCenter < voidRadius) continue;
        
        if (isInFinder(row, col, moduleCount)) continue;
        
        const neighbors = getNeighbors(qr, row, col, moduleCount);
        drawModule(ctx, x, y, moduleSize, neighbors, row, col);
      }
    }
    
    // Draw finder patterns
    const finderSize = moduleSize * 7;
    drawFinderPattern(ctx, finderSize / 2, finderSize / 2, moduleSize);
    drawFinderPattern(ctx, canvasSize - finderSize / 2, finderSize / 2, moduleSize);
    drawFinderPattern(ctx, finderSize / 2, canvasSize - finderSize / 2, moduleSize);
  }

  function getNeighbors(qr, row, col, moduleCount) {
    return {
      left: col > 0 && qr.isDark(row, col - 1) && !isInFinder(row, col - 1, moduleCount),
      right: col < moduleCount - 1 && qr.isDark(row, col + 1) && !isInFinder(row, col + 1, moduleCount),
      top: row > 0 && qr.isDark(row - 1, col) && !isInFinder(row - 1, col, moduleCount),
      bottom: row < moduleCount - 1 && qr.isDark(row + 1, col) && !isInFinder(row + 1, col, moduleCount),
      topLeft: row > 0 && col > 0 && qr.isDark(row - 1, col - 1) && !isInFinder(row - 1, col - 1, moduleCount),
      topRight: row > 0 && col < moduleCount - 1 && qr.isDark(row - 1, col + 1) && !isInFinder(row - 1, col + 1, moduleCount),
      bottomLeft: row < moduleCount - 1 && col > 0 && qr.isDark(row + 1, col - 1) && !isInFinder(row + 1, col - 1, moduleCount),
      bottomRight: row < moduleCount - 1 && col < moduleCount - 1 && qr.isDark(row + 1, col + 1) && !isInFinder(row + 1, col + 1, moduleCount)
    };
  }

  function isInFinder(row, col, moduleCount) {
    const isTopLeft = row < 7 && col < 7;
    const isTopRight = row < 7 && col >= moduleCount - 7;
    const isBottomLeft = row >= moduleCount - 7 && col < 7;
    return isTopLeft || isTopRight || isBottomLeft;
  }

  function drawModule(ctx, x, y, size, neighbors, row, col) {
    const actualPadding = config.paddingAmount === 0 ? 0 : size * config.paddingAmount;
    
    const bleed = size * config.edgeBleed * 0.3;
    const bleedLeft = neighbors.left ? bleed : 0;
    const bleedRight = neighbors.right ? bleed : 0;
    const bleedTop = neighbors.top ? bleed : 0;
    const bleedBottom = neighbors.bottom ? bleed : 0;
    
    const x1 = x + actualPadding - bleedLeft;
    const y1 = y + actualPadding - bleedTop;
    const w = size - actualPadding * 2 + bleedLeft + bleedRight;
    const h = size - actualPadding * 2 + bleedTop + bleedBottom;
    
    const maxRadius = Math.min(w, h) / 2;
    const radius = maxRadius * config.roundingAmount * 2;
    
    const roundTL = !neighbors.top && !neighbors.left;
    const roundTR = !neighbors.top && !neighbors.right;
    const roundBR = !neighbors.bottom && !neighbors.right;
    const roundBL = !neighbors.bottom && !neighbors.left;
    
    const softRoundTL = neighbors.top && neighbors.left && !neighbors.topLeft ? radius * 0.3 : 0;
    const softRoundTR = neighbors.top && neighbors.right && !neighbors.topRight ? radius * 0.3 : 0;
    const softRoundBR = neighbors.bottom && neighbors.right && !neighbors.bottomRight ? radius * 0.3 : 0;
    const softRoundBL = neighbors.bottom && neighbors.left && !neighbors.bottomLeft ? radius * 0.3 : 0;
    
    const chaosX = config.geometricChaos > 0 ? (Math.sin(row * 2.5 + col * 1.3) * size * config.geometricChaos * 0.3) : 0;
    const chaosY = config.geometricChaos > 0 ? (Math.cos(row * 1.8 + col * 2.2) * size * config.geometricChaos * 0.3) : 0;
    
    ctx.beginPath();
    ctx.moveTo(x1 + (roundTL ? radius : softRoundTL) + chaosX, y1 + chaosY);
    ctx.lineTo(x1 + w - (roundTR ? radius : softRoundTR) + chaosX, y1 + chaosY);
    
    if (roundTR || softRoundTR > 0) {
      const r = roundTR ? radius : softRoundTR;
      ctx.quadraticCurveTo(x1 + w + chaosX, y1 + chaosY, x1 + w + chaosX, y1 + r + chaosY);
    }
    
    ctx.lineTo(x1 + w + chaosX, y1 + h - (roundBR ? radius : softRoundBR) + chaosY);
    
    if (roundBR || softRoundBR > 0) {
      const r = roundBR ? radius : softRoundBR;
      ctx.quadraticCurveTo(x1 + w + chaosX, y1 + h + chaosY, x1 + w - r + chaosX, y1 + h + chaosY);
    }
    
    ctx.lineTo(x1 + (roundBL ? radius : softRoundBL) + chaosX, y1 + h + chaosY);
    
    if (roundBL || softRoundBL > 0) {
      const r = roundBL ? radius : softRoundBL;
      ctx.quadraticCurveTo(x1 + chaosX, y1 + h + chaosY, x1 + chaosX, y1 + h - r + chaosY);
    }
    
    ctx.lineTo(x1 + chaosX, y1 + (roundTL ? radius : softRoundTL) + chaosY);
    
    if (roundTL || softRoundTL > 0) {
      const r = roundTL ? radius : softRoundTL;
      ctx.quadraticCurveTo(x1 + chaosX, y1 + chaosY, x1 + r + chaosX, y1 + chaosY);
    }
    
    ctx.closePath();
    ctx.fill();
  }

  function drawFinderPattern(ctx, cx, cy, moduleSize) {
    const size = moduleSize * 7;
    // Correct finder pattern proportions: 7:5:3 ratio
    // Outer black: 7 modules, Middle white: 5 modules, Inner black: 3 modules
    const outerSize = size;  // 7/7 = 100%
    const middleSize = size * (5 / 7);  // 5/7 ≈ 71.4%
    const innerSize = size * (3 / 7);  // 3/7 ≈ 42.8%
    
    ctx.fillStyle = '#000000';
    
    if (config.finderOuterShape === 'rounded-square') {
      drawRoundedRect(ctx, cx - outerSize/2, cy - outerSize/2, outerSize, outerSize, outerSize * 0.15);
      ctx.fill();
      ctx.fillStyle = '#ffffff';
      drawRoundedRect(ctx, cx - middleSize/2, cy - middleSize/2, middleSize, middleSize, middleSize * 0.15);
      ctx.fill();
    } else {
      ctx.beginPath();
      ctx.arc(cx, cy, outerSize/2, 0, Math.PI * 2);
      ctx.fill();
      ctx.fillStyle = '#ffffff';
      ctx.beginPath();
      ctx.arc(cx, cy, middleSize/2, 0, Math.PI * 2);
      ctx.fill();
    }
    
    ctx.fillStyle = '#000000';
    if (config.finderInnerShape === 'rounded-square') {
      // Apply overlap adjustment (positive = larger, negative = smaller)
      const adjustedSize = innerSize * (1 + config.finderCenterOverlap * 0.5);
      drawRoundedRect(ctx, cx - adjustedSize/2, cy - adjustedSize/2, adjustedSize, adjustedSize, adjustedSize * 0.2);
      ctx.fill();
    } else {
      ctx.beginPath();
      ctx.arc(cx, cy, innerSize/2 * (1 + config.finderCenterOverlap * 0.5), 0, Math.PI * 2);
      ctx.fill();
    }
  }

  function drawRoundedRect(ctx, x, y, width, height, radius) {
    ctx.beginPath();
    ctx.moveTo(x + radius, y);
    ctx.lineTo(x + width - radius, y);
    ctx.quadraticCurveTo(x + width, y, x + width, y + radius);
    ctx.lineTo(x + width, y + height - radius);
    ctx.quadraticCurveTo(x + width, y + height, x + width - radius, y + height);
    ctx.lineTo(x + radius, y + height);
    ctx.quadraticCurveTo(x, y + height, x, y + height - radius);
    ctx.lineTo(x, y + radius);
    ctx.quadraticCurveTo(x, y, x + radius, y);
    ctx.closePath();
  }
</script>

<canvas bind:this={canvas} style="display: none;"></canvas>
