<script>
  export let config = {
    // Outer shape
    outerShape: 'hexagon',
    outerShapeSize: 480,
    outerBorderCount: 3,
    outerBorderSpacing: 15,
    outerColor: '#000000',
    
    // Inner pattern
    innerShapeType: 'cube',
    innerShapeSize: 15,
    innerShapeSpacing: 18,
    innerRotation: 0,
    innerPitch: 30,
    innerColor: '#000000',
    
    // Center
    centerVoidSize: 300,
    centerLogoShape: 'circles',
    centerLogoSize: 60,
    centerLogoColor: '#000000',
    
    // Module styling for module-based pattern
    roundingAmount: 0.45,
    paddingAmount: 0,
    edgeBleed: 0,
    geometricChaos: 0
  };
  
  export let canvas;
  export let canvasSize = 1000;
  
  $: centerX = canvasSize / 2;
  $: centerY = canvasSize / 2;

  export function render() {
    if (!canvas) {
      console.error('GeometricFrame: Canvas not available');
      return;
    }
    
    console.log('GeometricFrame: Rendering...', {
      outerShape: config.outerShape,
      outerSize: config.outerShapeSize,
      innerType: config.innerShapeType
    });
    
    const ctx = canvas.getContext('2d');
    canvas.width = canvasSize;
    canvas.height = canvasSize;
    
    ctx.fillStyle = '#ffffff';
    ctx.fillRect(0, 0, canvasSize, canvasSize);
    
    drawInnerPattern(ctx);
    drawOuterBorders(ctx);
    drawCenterVoid(ctx);
    drawCenterLogo(ctx);
    
    console.log('GeometricFrame: Render complete');
  }

  function drawInnerPattern(ctx) {
    ctx.save();
    
    // Create clip path for outer shape
    ctx.beginPath();
    drawShapePath(ctx, centerX, centerY, config.outerShapeSize, config.outerShape);
    ctx.clip();
    
    // Apply rotation
    ctx.translate(centerX, centerY);
    ctx.rotate((config.innerRotation * Math.PI) / 180);
    ctx.translate(-centerX, -centerY);
    
    // Generate grid
    const maxDist = config.outerShapeSize + config.innerShapeSize * 2;
    const range = Math.ceil(maxDist / Math.max(config.innerShapeSpacing, 1));
    
    for (let x = -range; x <= range; x++) {
      for (let y = -range; y <= range; y++) {
        const shapeX = centerX + x * config.innerShapeSpacing;
        const shapeY = centerY + y * config.innerShapeSpacing;
        
        // Skip if in center void
        if (Math.abs(shapeX - centerX) < config.centerVoidSize / 2 && 
            Math.abs(shapeY - centerY) < config.centerVoidSize / 2) {
          continue;
        }
        
        // Draw shape based on type
        if (config.innerShapeType === 'module-based') {
          const neighbors = {
            left: false, right: false, top: false, bottom: false,
            topLeft: false, topRight: false, bottomLeft: false, bottomRight: false
          };
          drawModule(ctx, shapeX - config.innerShapeSize/2, shapeY - config.innerShapeSize/2, 
                    config.innerShapeSize, neighbors, x, y);
        } else if (config.innerShapeType === 'cube') {
          drawIsometricCube(ctx, shapeX, shapeY, config.innerShapeSize, config.innerPitch);
        } else if (config.innerShapeType === 'cylinder') {
          drawIsometricCylinder(ctx, shapeX, shapeY, config.innerShapeSize, config.innerPitch);
        } else if (config.innerShapeType === 'pyramid') {
          drawIsometricPyramid(ctx, shapeX, shapeY, config.innerShapeSize, config.innerPitch);
        } else if (config.innerShapeType === 'circle') {
          ctx.fillStyle = config.innerColor;
          ctx.beginPath();
          ctx.arc(shapeX, shapeY, config.innerShapeSize / 2, 0, Math.PI * 2);
          ctx.fill();
        } else if (config.innerShapeType === 'square') {
          ctx.fillStyle = config.innerColor;
          ctx.fillRect(
            shapeX - config.innerShapeSize / 2,
            shapeY - config.innerShapeSize / 2,
            config.innerShapeSize,
            config.innerShapeSize
          );
        } else if (config.innerShapeType === 'diamond') {
          ctx.fillStyle = config.innerColor;
          ctx.beginPath();
          ctx.moveTo(shapeX, shapeY - config.innerShapeSize / 2);
          ctx.lineTo(shapeX + config.innerShapeSize / 2, shapeY);
          ctx.lineTo(shapeX, shapeY + config.innerShapeSize / 2);
          ctx.lineTo(shapeX - config.innerShapeSize / 2, shapeY);
          ctx.closePath();
          ctx.fill();
        }
      }
    }
    
    ctx.restore();
  }

  function drawOuterBorders(ctx) {
    for (let i = 0; i < config.outerBorderCount; i++) {
      const borderSize = config.outerShapeSize - (i * config.outerBorderSpacing);
      const opacity = 1 - (i * 0.15);
      
      if (borderSize > 50) {
        ctx.strokeStyle = config.outerColor;
        ctx.globalAlpha = opacity;
        ctx.lineWidth = 3;
        ctx.beginPath();
        drawShapePath(ctx, centerX, centerY, borderSize, config.outerShape);
        ctx.stroke();
        ctx.globalAlpha = 1;
      }
    }
  }

  function drawCenterVoid(ctx) {
    ctx.fillStyle = '#ffffff';
    ctx.fillRect(
      centerX - config.centerVoidSize / 2,
      centerY - config.centerVoidSize / 2,
      config.centerVoidSize,
      config.centerVoidSize
    );
  }

  function drawCenterLogo(ctx) {
    ctx.fillStyle = config.centerLogoColor;
    ctx.strokeStyle = config.centerLogoColor;
    
    if (config.centerLogoShape === 'circles') {
      ctx.lineWidth = 8;
      ctx.beginPath();
      ctx.arc(centerX, centerY, config.centerLogoSize, 0, Math.PI * 2);
      ctx.stroke();
      
      ctx.lineWidth = 6;
      ctx.beginPath();
      ctx.arc(centerX, centerY, config.centerLogoSize * 0.65, 0, Math.PI * 2);
      ctx.stroke();
      
      ctx.beginPath();
      ctx.arc(centerX, centerY, config.centerLogoSize * 0.3, 0, Math.PI * 2);
      ctx.fill();
    } else if (config.centerLogoShape === 'square') {
      ctx.save();
      ctx.translate(centerX, centerY);
      ctx.rotate(Math.PI / 4);
      
      ctx.lineWidth = 6;
      ctx.strokeRect(-config.centerLogoSize, -config.centerLogoSize, config.centerLogoSize * 2, config.centerLogoSize * 2);
      ctx.fillRect(-config.centerLogoSize * 0.6, -config.centerLogoSize * 0.6, config.centerLogoSize * 1.2, config.centerLogoSize * 1.2);
      
      ctx.restore();
    } else if (config.centerLogoShape === 'star') {
      ctx.beginPath();
      for (let i = 0; i < 10; i++) {
        const radius = i % 2 === 0 ? config.centerLogoSize : config.centerLogoSize * 0.4;
        const angle = (Math.PI / 5) * i - Math.PI / 2;
        const x = centerX + radius * Math.cos(angle);
        const y = centerY + radius * Math.sin(angle);
        if (i === 0) ctx.moveTo(x, y);
        else ctx.lineTo(x, y);
      }
      ctx.closePath();
      ctx.fill();
    } else if (config.centerLogoShape === 'diamond') {
      ctx.lineWidth = 8;
      ctx.beginPath();
      ctx.moveTo(centerX, centerY - config.centerLogoSize * 2);
      ctx.lineTo(centerX + config.centerLogoSize * 2, centerY);
      ctx.lineTo(centerX, centerY + config.centerLogoSize * 2);
      ctx.lineTo(centerX - config.centerLogoSize * 2, centerY);
      ctx.closePath();
      ctx.stroke();
      
      ctx.beginPath();
      ctx.moveTo(centerX, centerY - config.centerLogoSize);
      ctx.lineTo(centerX + config.centerLogoSize, centerY);
      ctx.lineTo(centerX, centerY + config.centerLogoSize);
      ctx.lineTo(centerX - config.centerLogoSize, centerY);
      ctx.closePath();
      ctx.fill();
    }
  }

  function drawShapePath(ctx, cx, cy, size, shape) {
    if (shape === 'circle') {
      ctx.arc(cx, cy, size, 0, Math.PI * 2);
    } else if (shape === 'square') {
      ctx.rect(cx - size, cy - size, size * 2, size * 2);
    } else if (shape === 'hexagon') {
      for (let i = 0; i < 6; i++) {
        const angle = (Math.PI / 3) * i - Math.PI / 2;
        const x = cx + size * Math.cos(angle);
        const y = cy + size * Math.sin(angle);
        if (i === 0) ctx.moveTo(x, y);
        else ctx.lineTo(x, y);
      }
      ctx.closePath();
    } else if (shape === 'octagon') {
      for (let i = 0; i < 8; i++) {
        const angle = (Math.PI / 4) * i - Math.PI / 2;
        const x = cx + size * Math.cos(angle);
        const y = cy + size * Math.sin(angle);
        if (i === 0) ctx.moveTo(x, y);
        else ctx.lineTo(x, y);
      }
      ctx.closePath();
    }
  }

  function drawIsometricCube(ctx, x, y, size, pitch) {
    const pitchRad = (pitch * Math.PI) / 180;
    const h = size * Math.sin(pitchRad);
    const w = size * Math.cos(pitchRad) * 0.866;
    const offsetY = -size * 0.5;
    
    ctx.fillStyle = 'rgba(0, 0, 0, 0.9)';
    ctx.beginPath();
    ctx.moveTo(x, y + offsetY);
    ctx.lineTo(x + w, y + h + offsetY);
    ctx.lineTo(x, y + h * 2 + offsetY);
    ctx.lineTo(x - w, y + h + offsetY);
    ctx.closePath();
    ctx.fill();
    
    ctx.fillStyle = 'rgba(0, 0, 0, 0.6)';
    ctx.beginPath();
    ctx.moveTo(x, y + offsetY);
    ctx.lineTo(x - w, y + h + offsetY);
    ctx.lineTo(x - w, y + h + size + offsetY);
    ctx.lineTo(x, y + size * 2 + offsetY);
    ctx.closePath();
    ctx.fill();
    
    ctx.fillStyle = 'rgba(0, 0, 0, 0.75)';
    ctx.beginPath();
    ctx.moveTo(x, y + offsetY);
    ctx.lineTo(x + w, y + h + offsetY);
    ctx.lineTo(x + w, y + h + size + offsetY);
    ctx.lineTo(x, y + size * 2 + offsetY);
    ctx.closePath();
    ctx.fill();
  }

  function drawIsometricCylinder(ctx, x, y, size, pitch) {
    const pitchRad = (pitch * Math.PI) / 180;
    const radiusX = size * 0.866;
    const radiusY = size * Math.sin(pitchRad) * 0.5;
    const height = size * 1.5;
    const offsetY = -height * 0.5;
    
    ctx.fillStyle = 'rgba(0, 0, 0, 0.75)';
    ctx.beginPath();
    ctx.ellipse(x, y + offsetY, radiusX, radiusY, 0, 0, Math.PI * 2);
    ctx.fill();
    
    ctx.fillRect(x - radiusX, y + offsetY, radiusX * 2, height);
    
    ctx.fillStyle = 'rgba(0, 0, 0, 0.9)';
    ctx.beginPath();
    ctx.ellipse(x, y + height + offsetY, radiusX, radiusY, 0, 0, Math.PI * 2);
    ctx.fill();
  }

  function drawIsometricPyramid(ctx, x, y, size, pitch) {
    const pitchRad = (pitch * Math.PI) / 180;
    const base = size * 1.5;
    const h = size * Math.cos(pitchRad) * 0.866;
    const height = size * Math.sin(pitchRad) * 2;
    const offsetY = -size * 0.4;
    
    ctx.fillStyle = 'rgba(0, 0, 0, 0.7)';
    ctx.beginPath();
    ctx.moveTo(x, y - height + offsetY);
    ctx.lineTo(x - base/2, y + h/2 + offsetY);
    ctx.lineTo(x, y + h + offsetY);
    ctx.closePath();
    ctx.fill();
    
    ctx.fillStyle = 'rgba(0, 0, 0, 0.85)';
    ctx.beginPath();
    ctx.moveTo(x, y - height + offsetY);
    ctx.lineTo(x, y + h + offsetY);
    ctx.lineTo(x + base/2, y + h/2 + offsetY);
    ctx.closePath();
    ctx.fill();
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
    
    ctx.fillStyle = config.innerColor;
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
</script>

<canvas bind:this={canvas} style="display: none;"></canvas>
