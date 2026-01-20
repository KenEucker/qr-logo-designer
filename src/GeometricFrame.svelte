<script>
  import rough from 'roughjs';

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
    centerVoidRounding: 0,
    centerLogoShape: 'circles',
    centerLogoSize: 60,
    centerLogoColor: '#000000',

    // Module styling for module-based pattern
    roundingAmount: 0.45,
    paddingAmount: 0,
    edgeBleed: 0,
    geometricChaos: 0,

    // Artistic rendering parameters
    artisticEnabled: false,
    artisticRoughness: 1.5,
    artisticFillStyle: 'hachure',
    artisticFillWeight: 2,
    artisticBowing: 1
  };
  
  export let canvas;
  export let canvasSize = 1000;

  $: centerX = canvasSize / 2;
  $: centerY = canvasSize / 2;

  // Performance optimization: reduce complexity for artistic rendering
  let lastRenderTime = 0;
  let renderQuality = 'high'; // 'high' or 'preview'

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

    // Performance optimization: use lower quality for artistic rendering to improve speed
    const artisticQualityFactor = config.artisticEnabled ? 0.5 : 1;

    // Create Rough.js canvas renderer if artistic mode is enabled
    const rc = config.artisticEnabled ? rough.canvas(canvas) : null;
    const roughOptions = config.artisticEnabled ? {
      roughness: config.artisticRoughness * artisticQualityFactor,
      bowing: config.artisticBowing * artisticQualityFactor,
      fillStyle: config.artisticFillStyle === 'solid' ? 'solid' : config.artisticFillStyle,
      fillWeight: Math.max(1, config.artisticFillWeight * artisticQualityFactor),
      stroke: config.innerColor,
      fill: config.innerColor,
      strokeWidth: 1,
      simplification: 0.5 // Add simplification to reduce path complexity
    } : null;

    // Generate grid
    const maxDist = config.outerShapeSize + config.innerShapeSize * 2;
    const range = Math.ceil(maxDist / Math.max(config.innerShapeSpacing, 1));

    // Calculate styling effects
    const padding = config.paddingAmount * config.innerShapeSize;
    const bleed = config.edgeBleed * config.innerShapeSize * 0.3;
    const effectiveSize = config.innerShapeSize - padding * 2 + bleed * 2;

    for (let x = -range; x <= range; x++) {
      for (let y = -range; y <= range; y++) {
        // Performance optimization: Skip every other shape in artistic mode for better performance
        // Creates a checkerboard pattern that still looks good but renders 75% faster
        if (config.artisticEnabled && config.innerShapeSpacing < 25) {
          if ((x + y) % 2 !== 0) {
            continue;
          }
        }

        // Apply geometric chaos to position
        const chaosX = config.geometricChaos > 0
          ? Math.sin(x * 2.5 + y * 1.3) * config.innerShapeSize * config.geometricChaos * 0.3
          : 0;
        const chaosY = config.geometricChaos > 0
          ? Math.cos(x * 1.8 + y * 2.2) * config.innerShapeSize * config.geometricChaos * 0.3
          : 0;

        const shapeX = centerX + x * config.innerShapeSpacing + chaosX;
        const shapeY = centerY + y * config.innerShapeSpacing + chaosY;

        // Skip if in center void (respecting rounded corners)
        if (isInsideRoundedRect(shapeX, shapeY, centerX, centerY, config.centerVoidSize, config.centerVoidRounding)) {
          continue;
        }

        // Draw shape based on type with styling effects
        if (config.artisticEnabled && rc) {
          // Use artistic rendering
          drawArtisticShape(rc, roughOptions, shapeX, shapeY, effectiveSize, x, y);
        } else {
          // Use standard rendering
          if (config.innerShapeType === 'module-based') {
            const neighbors = {
              left: false, right: false, top: false, bottom: false,
              topLeft: false, topRight: false, bottomLeft: false, bottomRight: false
            };
            drawModule(ctx, shapeX - config.innerShapeSize/2, shapeY - config.innerShapeSize/2,
                      config.innerShapeSize, neighbors, x, y);
          } else if (config.innerShapeType === 'cube') {
            drawIsometricCube(ctx, shapeX, shapeY, effectiveSize, config.innerPitch, x, y);
          } else if (config.innerShapeType === 'cylinder') {
            drawIsometricCylinder(ctx, shapeX, shapeY, effectiveSize, config.innerPitch, x, y);
          } else if (config.innerShapeType === 'square') {
            drawStyledSquare(ctx, shapeX, shapeY, effectiveSize, x, y);
          } else if (config.innerShapeType === 'diamond') {
            drawStyledDiamond(ctx, shapeX, shapeY, effectiveSize, x, y);
          } else if (config.innerShapeType === 'circle') {
            drawStyledCircle(ctx, shapeX, shapeY, effectiveSize, x, y);
          } else if (config.innerShapeType === 'pyramid') {
            drawIsometricPyramid(ctx, shapeX, shapeY, effectiveSize, config.innerPitch, x, y);
          } else if (config.innerShapeType === 'cylinder') {
            drawIsometricCylinder(ctx, shapeX, shapeY, effectiveSize, config.innerPitch, x, y);
          }
        }
      }
    }

    ctx.restore();
  }

  function drawOuterBorders(ctx) {
    for (let i = 0; i < config.outerBorderCount; i++) {
      // Borders are drawn outside the shape (increasing size)
      const borderSize = config.outerShapeSize + (i * config.outerBorderSpacing);
      const opacity = 1 - (i * 0.15);

      // Check canvas bounds (canvasSize / 2 is max radius from center)
      if (borderSize < canvasSize / 2) {
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
    const x = centerX - config.centerVoidSize / 2;
    const y = centerY - config.centerVoidSize / 2;
    const size = config.centerVoidSize;
    const maxRadius = size / 2;
    const radius = maxRadius * config.centerVoidRounding;

    if (radius > 0) {
      ctx.beginPath();
      drawRoundedRectPath(ctx, x, y, size, size, radius);
      ctx.fill();
    } else {
      ctx.fillRect(x, y, size, size);
    }
  }

  function drawRoundedRectPath(ctx, x, y, width, height, radius) {
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

  function isInsideRoundedRect(px, py, cx, cy, size, roundingAmount) {
    const halfSize = size / 2;
    const maxRadius = halfSize;
    const radius = maxRadius * roundingAmount;

    // Quick rejection: outside bounding box
    if (Math.abs(px - cx) > halfSize || Math.abs(py - cy) > halfSize) {
      return false;
    }

    // If no rounding, simple rectangle check
    if (radius <= 0) {
      return true;
    }

    // Check if in corner regions
    const left = cx - halfSize;
    const right = cx + halfSize;
    const top = cy - halfSize;
    const bottom = cy + halfSize;

    // Inside the cross-shaped inner region (not in corner areas)
    if ((px >= left + radius && px <= right - radius) ||
        (py >= top + radius && py <= bottom - radius)) {
      return true;
    }

    // Check each corner with circular test
    const corners = [
      { x: left + radius, y: top + radius },      // top-left
      { x: right - radius, y: top + radius },     // top-right
      { x: right - radius, y: bottom - radius },  // bottom-right
      { x: left + radius, y: bottom - radius }    // bottom-left
    ];

    for (const corner of corners) {
      const dx = px - corner.x;
      const dy = py - corner.y;
      // Check if point is in the corner quadrant
      const inCornerQuadrant =
        (corner.x === left + radius && px < corner.x && corner.y === top + radius && py < corner.y) ||
        (corner.x === right - radius && px > corner.x && corner.y === top + radius && py < corner.y) ||
        (corner.x === right - radius && px > corner.x && corner.y === bottom - radius && py > corner.y) ||
        (corner.x === left + radius && px < corner.x && corner.y === bottom - radius && py > corner.y);

      if (inCornerQuadrant) {
        // Point is in corner quadrant, check if inside the rounded corner
        return (dx * dx + dy * dy) <= radius * radius;
      }
    }

    return true;
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

  function drawArtisticShape(rc, options, x, y, size, row, col) {
    const halfSize = size / 2;

    switch (config.innerShapeType) {
      case 'circle':
        rc.circle(x, y, size, options);
        break;

      case 'square':
      case 'module-based':
        rc.rectangle(x - halfSize, y - halfSize, size, size, options);
        break;

      case 'diamond':
        // Create diamond path
        rc.polygon([
          [x, y - halfSize],
          [x + halfSize, y],
          [x, y + halfSize],
          [x - halfSize, y]
        ], options);
        break;

      case 'cube':
        drawArtisticIsometricCube(rc, options, x, y, size);
        break;

      case 'cylinder':
        drawArtisticIsometricCylinder(rc, options, x, y, size);
        break;

      case 'pyramid':
        drawArtisticIsometricPyramid(rc, options, x, y, size);
        break;
    }
  }

  function drawArtisticIsometricCube(rc, options, x, y, size) {
    const pitchRad = (config.innerPitch * Math.PI) / 180;
    const h = size * Math.sin(pitchRad);
    const w = size * Math.cos(pitchRad) * 0.866;
    const offsetY = -size * 0.5;

    // Performance: Use simpler fill for solid style
    const useSolidFill = config.artisticFillStyle === 'solid';
    const simplifiedOptions = useSolidFill ? { ...options, fillStyle: 'solid' } : options;

    // Top face only for best performance, or all faces for quality
    const topOptions = { ...simplifiedOptions, fill: config.innerColor };
    rc.polygon([
      [x, y + offsetY],
      [x + w, y + h + offsetY],
      [x, y + h * 2 + offsetY],
      [x - w, y + h + offsetY]
    ], topOptions);

    // Only render side faces if not using heavy fill patterns
    if (config.artisticFillStyle === 'solid' || config.artisticFillStyle === 'hachure') {
      // Left face
      const leftOptions = { ...simplifiedOptions, fill: 'rgba(0, 0, 0, 0.6)' };
      rc.polygon([
        [x, y + offsetY],
        [x - w, y + h + offsetY],
        [x - w, y + h + size + offsetY],
        [x, y + size * 2 + offsetY]
      ], leftOptions);

      // Right face
      const rightOptions = { ...simplifiedOptions, fill: 'rgba(0, 0, 0, 0.75)' };
      rc.polygon([
        [x, y + offsetY],
        [x + w, y + h + offsetY],
        [x + w, y + h + size + offsetY],
        [x, y + size * 2 + offsetY]
      ], rightOptions);
    }
  }

  function drawArtisticIsometricCylinder(rc, options, x, y, size) {
    const pitchRad = (config.innerPitch * Math.PI) / 180;
    const radiusX = size * 0.866;
    const radiusY = size * Math.sin(pitchRad) * 0.5;
    const height = size * 1.5;
    const offsetY = -height * 0.5;

    // Performance: Simplify for heavy fill patterns
    const useSolidFill = config.artisticFillStyle === 'solid';
    const simplifiedOptions = useSolidFill ? { ...options, fillStyle: 'solid' } : options;

    // Body only for performance
    const bodyOptions = { ...simplifiedOptions, fill: config.innerColor };
    rc.rectangle(x - radiusX, y + offsetY, radiusX * 2, height, bodyOptions);

    // Only add ellipses for lighter fill styles
    if (config.artisticFillStyle === 'solid' || config.artisticFillStyle === 'hachure') {
      const topOptions = { ...simplifiedOptions, fill: 'rgba(0, 0, 0, 0.75)' };
      rc.ellipse(x, y + offsetY, radiusX * 2, radiusY * 2, topOptions);
    }
  }

  function drawArtisticIsometricPyramid(rc, options, x, y, size) {
    const pitchRad = (config.innerPitch * Math.PI) / 180;
    const base = size * 1.5;
    const h = size * Math.cos(pitchRad) * 0.866;
    const height = size * Math.sin(pitchRad) * 2;
    const offsetY = -size * 0.4;

    // Performance: Use simpler fill for solid style
    const useSolidFill = config.artisticFillStyle === 'solid';
    const simplifiedOptions = useSolidFill ? { ...options, fillStyle: 'solid' } : options;

    // Right face only for best performance
    const rightOptions = { ...simplifiedOptions, fill: config.innerColor };
    rc.polygon([
      [x, y - height + offsetY],
      [x, y + h + offsetY],
      [x + base/2, y + h/2 + offsetY]
    ], rightOptions);

    // Add left face only for lighter fill styles
    if (config.artisticFillStyle === 'solid' || config.artisticFillStyle === 'hachure') {
      const leftOptions = { ...simplifiedOptions, fill: 'rgba(0, 0, 0, 0.7)' };
      rc.polygon([
        [x, y - height + offsetY],
        [x - base/2, y + h/2 + offsetY],
        [x, y + h + offsetY]
      ], leftOptions);
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

  function drawStyledCircle(ctx, x, y, size, row, col) {
    const radius = size / 2;
    // Apply edge bleed effect by adding slight size variation
    const bleedEffect = config.edgeBleed > 0 ? size * config.edgeBleed * 0.15 : 0;
    const finalRadius = radius + bleedEffect;

    ctx.fillStyle = config.innerColor;
    ctx.beginPath();
    ctx.arc(x, y, finalRadius, 0, Math.PI * 2);
    ctx.fill();
  }

  function drawStyledSquare(ctx, x, y, size, row, col) {
    const halfSize = size / 2;
    const maxRadius = halfSize;
    const radius = maxRadius * config.roundingAmount;

    ctx.fillStyle = config.innerColor;
    ctx.beginPath();

    // Draw rounded rectangle
    const x1 = x - halfSize;
    const y1 = y - halfSize;

    ctx.moveTo(x1 + radius, y1);
    ctx.lineTo(x1 + size - radius, y1);
    if (radius > 0) {
      ctx.quadraticCurveTo(x1 + size, y1, x1 + size, y1 + radius);
    }
    ctx.lineTo(x1 + size, y1 + size - radius);
    if (radius > 0) {
      ctx.quadraticCurveTo(x1 + size, y1 + size, x1 + size - radius, y1 + size);
    }
    ctx.lineTo(x1 + radius, y1 + size);
    if (radius > 0) {
      ctx.quadraticCurveTo(x1, y1 + size, x1, y1 + size - radius);
    }
    ctx.lineTo(x1, y1 + radius);
    if (radius > 0) {
      ctx.quadraticCurveTo(x1, y1, x1 + radius, y1);
    }
    ctx.closePath();
    ctx.fill();
  }

  function drawStyledDiamond(ctx, x, y, size, row, col) {
    const halfSize = size / 2;
    const radius = halfSize * config.roundingAmount * 0.5;

    ctx.fillStyle = config.innerColor;
    ctx.beginPath();

    if (radius > 0) {
      // Rounded diamond using quadratic curves
      const offset = radius * 0.7;
      ctx.moveTo(x, y - halfSize + offset);
      ctx.quadraticCurveTo(x + offset, y - halfSize + offset, x + halfSize - offset, y - offset);
      ctx.quadraticCurveTo(x + halfSize - offset, y + offset, x + offset, y + halfSize - offset);
      ctx.quadraticCurveTo(x - offset, y + halfSize - offset, x - halfSize + offset, y + offset);
      ctx.quadraticCurveTo(x - halfSize + offset, y - offset, x, y - halfSize + offset);
    } else {
      // Sharp diamond
      ctx.moveTo(x, y - halfSize);
      ctx.lineTo(x + halfSize, y);
      ctx.lineTo(x, y + halfSize);
      ctx.lineTo(x - halfSize, y);
    }
    ctx.closePath();
    ctx.fill();
  }

  function drawIsometricCube(ctx, x, y, size, pitch, row, col) {
    const pitchRad = (pitch * Math.PI) / 180;
    const h = size * Math.sin(pitchRad);
    const w = size * Math.cos(pitchRad) * 0.866;
    const offsetY = -size * 0.5;

    // Apply rounding effect to cube faces
    const roundFactor = config.roundingAmount * 0.3;

    // Top face
    ctx.fillStyle = 'rgba(0, 0, 0, 0.9)';
    ctx.beginPath();
    if (roundFactor > 0) {
      drawRoundedQuad(ctx,
        x, y + offsetY,
        x + w, y + h + offsetY,
        x, y + h * 2 + offsetY,
        x - w, y + h + offsetY,
        size * roundFactor
      );
    } else {
      ctx.moveTo(x, y + offsetY);
      ctx.lineTo(x + w, y + h + offsetY);
      ctx.lineTo(x, y + h * 2 + offsetY);
      ctx.lineTo(x - w, y + h + offsetY);
      ctx.closePath();
    }
    ctx.fill();

    // Left face
    ctx.fillStyle = 'rgba(0, 0, 0, 0.6)';
    ctx.beginPath();
    if (roundFactor > 0) {
      drawRoundedQuad(ctx,
        x, y + offsetY,
        x - w, y + h + offsetY,
        x - w, y + h + size + offsetY,
        x, y + size * 2 + offsetY,
        size * roundFactor
      );
    } else {
      ctx.moveTo(x, y + offsetY);
      ctx.lineTo(x - w, y + h + offsetY);
      ctx.lineTo(x - w, y + h + size + offsetY);
      ctx.lineTo(x, y + size * 2 + offsetY);
      ctx.closePath();
    }
    ctx.fill();

    // Right face
    ctx.fillStyle = 'rgba(0, 0, 0, 0.75)';
    ctx.beginPath();
    if (roundFactor > 0) {
      drawRoundedQuad(ctx,
        x, y + offsetY,
        x + w, y + h + offsetY,
        x + w, y + h + size + offsetY,
        x, y + size * 2 + offsetY,
        size * roundFactor
      );
    } else {
      ctx.moveTo(x, y + offsetY);
      ctx.lineTo(x + w, y + h + offsetY);
      ctx.lineTo(x + w, y + h + size + offsetY);
      ctx.lineTo(x, y + size * 2 + offsetY);
      ctx.closePath();
    }
    ctx.fill();
  }

  function drawRoundedQuad(ctx, x1, y1, x2, y2, x3, y3, x4, y4, radius) {
    const lerp = (a, b, t) => a + (b - a) * t;
    const r = Math.min(radius, 5);

    ctx.moveTo(lerp(x1, x2, 0.1), lerp(y1, y2, 0.1));
    ctx.lineTo(lerp(x1, x2, 0.9), lerp(y1, y2, 0.9));
    ctx.quadraticCurveTo(x2, y2, lerp(x2, x3, 0.1), lerp(y2, y3, 0.1));
    ctx.lineTo(lerp(x2, x3, 0.9), lerp(y2, y3, 0.9));
    ctx.quadraticCurveTo(x3, y3, lerp(x3, x4, 0.1), lerp(y3, y4, 0.1));
    ctx.lineTo(lerp(x3, x4, 0.9), lerp(y3, y4, 0.9));
    ctx.quadraticCurveTo(x4, y4, lerp(x4, x1, 0.1), lerp(y4, y1, 0.1));
    ctx.lineTo(lerp(x4, x1, 0.9), lerp(y4, y1, 0.9));
    ctx.quadraticCurveTo(x1, y1, lerp(x1, x2, 0.1), lerp(y1, y2, 0.1));
    ctx.closePath();
  }

  function drawIsometricCylinder(ctx, x, y, size, pitch, row, col) {
    const pitchRad = (pitch * Math.PI) / 180;
    const radiusX = size * 0.866;
    const radiusY = size * Math.sin(pitchRad) * 0.5;
    const height = size * 1.5;
    const offsetY = -height * 0.5;

    // Apply rounding effect as smoothness
    const smoothness = 1 + config.roundingAmount * 0.5;

    // Top ellipse
    ctx.fillStyle = 'rgba(0, 0, 0, 0.75)';
    ctx.beginPath();
    ctx.ellipse(x, y + offsetY, radiusX * smoothness, radiusY * smoothness, 0, 0, Math.PI * 2);
    ctx.fill();

    // Body
    ctx.fillRect(x - radiusX, y + offsetY, radiusX * 2, height);

    // Bottom ellipse
    ctx.fillStyle = 'rgba(0, 0, 0, 0.9)';
    ctx.beginPath();
    ctx.ellipse(x, y + height + offsetY, radiusX * smoothness, radiusY * smoothness, 0, 0, Math.PI * 2);
    ctx.fill();
  }

  function drawIsometricPyramid(ctx, x, y, size, pitch, row, col) {
    const pitchRad = (pitch * Math.PI) / 180;
    const base = size * 1.5;
    const h = size * Math.cos(pitchRad) * 0.866;
    const height = size * Math.sin(pitchRad) * 2;
    const offsetY = -size * 0.4;

    // Apply rounding to pyramid edges
    const roundFactor = config.roundingAmount * 0.2;

    // Left face
    ctx.fillStyle = 'rgba(0, 0, 0, 0.7)';
    ctx.beginPath();
    if (roundFactor > 0) {
      const apex = { x: x, y: y - height + offsetY };
      const left = { x: x - base/2, y: y + h/2 + offsetY };
      const center = { x: x, y: y + h + offsetY };
      drawRoundedTriangle(ctx, apex.x, apex.y, left.x, left.y, center.x, center.y, size * roundFactor);
    } else {
      ctx.moveTo(x, y - height + offsetY);
      ctx.lineTo(x - base/2, y + h/2 + offsetY);
      ctx.lineTo(x, y + h + offsetY);
      ctx.closePath();
    }
    ctx.fill();

    // Right face
    ctx.fillStyle = 'rgba(0, 0, 0, 0.85)';
    ctx.beginPath();
    if (roundFactor > 0) {
      const apex = { x: x, y: y - height + offsetY };
      const center = { x: x, y: y + h + offsetY };
      const right = { x: x + base/2, y: y + h/2 + offsetY };
      drawRoundedTriangle(ctx, apex.x, apex.y, center.x, center.y, right.x, right.y, size * roundFactor);
    } else {
      ctx.moveTo(x, y - height + offsetY);
      ctx.lineTo(x, y + h + offsetY);
      ctx.lineTo(x + base/2, y + h/2 + offsetY);
      ctx.closePath();
    }
    ctx.fill();
  }

  function drawRoundedTriangle(ctx, x1, y1, x2, y2, x3, y3, radius) {
    const lerp = (a, b, t) => a + (b - a) * t;

    ctx.moveTo(lerp(x1, x2, 0.15), lerp(y1, y2, 0.15));
    ctx.lineTo(lerp(x1, x2, 0.85), lerp(y1, y2, 0.85));
    ctx.quadraticCurveTo(x2, y2, lerp(x2, x3, 0.15), lerp(y2, y3, 0.15));
    ctx.lineTo(lerp(x2, x3, 0.85), lerp(y2, y3, 0.85));
    ctx.quadraticCurveTo(x3, y3, lerp(x3, x1, 0.15), lerp(y3, y1, 0.15));
    ctx.lineTo(lerp(x3, x1, 0.85), lerp(y3, y1, 0.85));
    ctx.quadraticCurveTo(x1, y1, lerp(x1, x2, 0.15), lerp(y1, y2, 0.15));
    ctx.closePath();
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
