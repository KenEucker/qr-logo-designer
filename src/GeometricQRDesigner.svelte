<script>
  // Canvas size
  let canvasSize = 600;
  
  // Shape selections
  let outerShape = 'hexagon'; // hexagon, circle, square, octagon
  let innerShape = 'cube'; // cube, cylinder, pyramid, triangle, diamond, square, circle
  let centerShape = 'circles'; // circles, square, star, diamond, flower
  
  // Size controls
  let outerShapeSize = 280; // Radius/half-width from center
  let innerShapeSize = 30; // Size of each repeating shape
  let innerShapeSpacing = 55; // Spacing between inner shapes (can be less than size for overlap)
  let centerVoidSize = 180; // Size of the center SQUARE void
  let centerShapeSize = 70; // Size of the center design
  
  // Inner pattern rotation and pitch
  let innerRotation = 0; // Rotation angle in degrees (0-360)
  let innerPitch = 30; // Isometric pitch/perspective (0-60 degrees)
  
  // Outer shape border settings
  let outerBorderCount = 1; // Number of border lines (1-5)
  let outerBorderSpacing = 15; // Spacing between borders
  
  // Colors
  let outerColor = "#2563eb";
  let innerPatternColor = "#8b5cf6";
  let centerColor = "#ec4899";
  let backgroundColor = "#1e293b";
  
  // Center point - everything centered here
  $: centerX = canvasSize / 2;
  $: centerY = canvasSize / 2;
  
  // Shape generation functions
  function getHexagonPoints(cx, cy, radius) {
    const points = [];
    for (let i = 0; i < 6; i++) {
      const angle = (Math.PI / 3) * i - Math.PI / 2;
      const x = cx + radius * Math.cos(angle);
      const y = cy + radius * Math.sin(angle);
      points.push(`${x},${y}`);
    }
    return points.join(" ");
  }
  
  function getOctagonPoints(cx, cy, radius) {
    const points = [];
    for (let i = 0; i < 8; i++) {
      const angle = (Math.PI / 4) * i - Math.PI / 2;
      const x = cx + radius * Math.cos(angle);
      const y = cy + radius * Math.sin(angle);
      points.push(`${x},${y}`);
    }
    return points.join(" ");
  }
  
  // Generate clip path for outer shape
  function getOuterClipPath() {
    switch(outerShape) {
      case 'circle':
        return `<circle cx="${centerX}" cy="${centerY}" r="${outerShapeSize}" />`;
      case 'square':
        return `<rect x="${centerX - outerShapeSize}" y="${centerY - outerShapeSize}" width="${outerShapeSize * 2}" height="${outerShapeSize * 2}" />`;
      case 'hexagon':
        return `<polygon points="${getHexagonPoints(centerX, centerY, outerShapeSize)}" />`;
      case 'octagon':
        return `<polygon points="${getOctagonPoints(centerX, centerY, outerShapeSize)}" />`;
      default:
        return '';
    }
  }
  
  // Generate inner pattern shapes - grid pattern that will be clipped
  function generateInnerPattern() {
    const shapes = [];
    
    // Create a full grid that extends beyond boundaries
    // It will be clipped by SVG
    const maxDist = outerShapeSize + innerShapeSize * 2;
    const range = Math.ceil(maxDist / Math.max(innerShapeSpacing, 1));
    
    for (let x = -range; x <= range; x++) {
      for (let y = -range; y <= range; y++) {
        const shapeX = centerX + x * innerShapeSpacing;
        const shapeY = centerY + y * innerShapeSpacing;
        shapes.push({ x: shapeX, y: shapeY });
      }
    }
    
    return shapes;
  }
  
  // Get path for isometric cube - CENTERED on x, y with adjustable pitch
  function getCubePath(x, y, size, pitch = 30) {
    const pitchRad = (pitch * Math.PI) / 180;
    const h = size * Math.sin(pitchRad);
    const w = size * Math.cos(pitchRad) * 0.866;
    
    // Offset to center the cube visually
    const offsetY = -size * 0.5;
    
    // Top face
    const top = `M ${x} ${y + offsetY} 
                 L ${x + w} ${y + h + offsetY} 
                 L ${x} ${y + h * 2 + offsetY} 
                 L ${x - w} ${y + h + offsetY} Z`;
    
    // Left face
    const left = `M ${x} ${y + offsetY} 
                  L ${x - w} ${y + h + offsetY} 
                  L ${x - w} ${y + h + size + offsetY} 
                  L ${x} ${y + size * 2 + offsetY} Z`;
    
    // Right face
    const right = `M ${x} ${y + offsetY} 
                   L ${x + w} ${y + h + offsetY} 
                   L ${x + w} ${y + h + size + offsetY} 
                   L ${x} ${y + size * 2 + offsetY} Z`;
    
    return { top, left, right };
  }
  
  // Get path for isometric cylinder - CENTERED on x, y with adjustable pitch
  function getCylinderPath(x, y, size, pitch = 30) {
    const pitchRad = (pitch * Math.PI) / 180;
    const radiusX = size * 0.866;
    const radiusY = size * Math.sin(pitchRad) * 0.5;
    const height = size * 1.5;
    
    // Offset to center
    const offsetY = -height * 0.5;
    
    // Top ellipse (visible part)
    const topEllipse = `M ${x - radiusX} ${y + offsetY} 
                        A ${radiusX} ${radiusY} 0 0 1 ${x + radiusX} ${y + offsetY}
                        A ${radiusX} ${radiusY} 0 0 1 ${x - radiusX} ${y + offsetY} Z`;
    
    // Side surface
    const side = `M ${x - radiusX} ${y + offsetY}
                  L ${x - radiusX} ${y + height + offsetY}
                  A ${radiusX} ${radiusY} 0 0 0 ${x + radiusX} ${y + height + offsetY}
                  L ${x + radiusX} ${y + offsetY}`;
    
    // Bottom ellipse
    const bottomEllipse = `M ${x - radiusX} ${y + height + offsetY}
                           A ${radiusX} ${radiusY} 0 0 1 ${x + radiusX} ${y + height + offsetY}
                           A ${radiusX} ${radiusY} 0 0 1 ${x - radiusX} ${y + height + offsetY} Z`;
    
    return { top: topEllipse, side, bottom: bottomEllipse };
  }
  
  // Get path for isometric pyramid (triangle) - CENTERED on x, y with adjustable pitch
  function getPyramidPath(x, y, size, pitch = 30) {
    const pitchRad = (pitch * Math.PI) / 180;
    const base = size * 1.5;
    const h = size * Math.cos(pitchRad) * 0.866;
    const height = size * Math.sin(pitchRad) * 2;
    
    // Offset to center
    const offsetY = -size * 0.4;
    
    // Front left face
    const leftFace = `M ${x} ${y - height + offsetY}
                      L ${x - base/2} ${y + h/2 + offsetY}
                      L ${x} ${y + h + offsetY} Z`;
    
    // Front right face  
    const rightFace = `M ${x} ${y - height + offsetY}
                       L ${x} ${y + h + offsetY}
                       L ${x + base/2} ${y + h/2 + offsetY} Z`;
    
    // Base (optional, usually hidden)
    const baseFace = `M ${x - base/2} ${y + h/2 + offsetY}
                      L ${x} ${y + h + offsetY}
                      L ${x + base/2} ${y + h/2 + offsetY} Z`;
    
    return { left: leftFace, right: rightFace, base: baseFace };
  }
  
  // Get path for triangle
  function getTrianglePath(x, y, size) {
    const h = size * 0.866;
    return `M ${x} ${y - size/2} L ${x + size/2} ${y + h/2} L ${x - size/2} ${y + h/2} Z`;
  }
  
  // Get path for diamond
  function getDiamondPath(x, y, size) {
    return `M ${x} ${y - size/2} L ${x + size/2} ${y} L ${x} ${y + size/2} L ${x - size/2} ${y} Z`;
  }
  
  // Get path for star (5-pointed)
  function getStarPath(cx, cy, size) {
    const points = [];
    const outerRadius = size;
    const innerRadius = size * 0.4;
    
    for (let i = 0; i < 10; i++) {
      const radius = i % 2 === 0 ? outerRadius : innerRadius;
      const angle = (Math.PI / 5) * i - Math.PI / 2;
      const x = cx + radius * Math.cos(angle);
      const y = cy + radius * Math.sin(angle);
      points.push(i === 0 ? `M ${x} ${y}` : `L ${x} ${y}`);
    }
    points.push('Z');
    return points.join(' ');
  }
  
  // Get path for flower (5 petals)
  function getFlowerPath(cx, cy, size) {
    const petals = [];
    for (let i = 0; i < 5; i++) {
      const angle = (Math.PI * 2 / 5) * i - Math.PI / 2;
      const petalX = cx + size * 0.6 * Math.cos(angle);
      const petalY = cy + size * 0.6 * Math.sin(angle);
      petals.push({ cx: petalX, cy: petalY, r: size * 0.4 });
    }
    return petals;
  }
  
  $: innerPatternShapes = generateInnerPattern();
  
  // Generate unique clip path ID when outer shape or size changes
  $: clipPathId = `clip-${outerShape}-${outerShapeSize}`;
  
  // Reactive update when any of these change
  $: {
    outerShape;
    outerShapeSize;
    innerShapeSize;
    innerShapeSpacing;
    centerVoidSize;
    centerX;
    centerY;
    innerRotation;
    innerPitch;
    innerPatternShapes = generateInnerPattern();
  }

</script>

<div class="app">
  <div class="controls">
    <h2>Geometric QR Frame Designer</h2>
    
    <div class="control-section">
      <h3>Outer Shape</h3>
      <div class="control-group">
        <label>
          Shape Type:
          <select bind:value={outerShape}>
            <option value="hexagon">Hexagon</option>
            <option value="circle">Circle</option>
            <option value="square">Square</option>
            <option value="octagon">Octagon</option>
          </select>
        </label>
        <label>
          Size: {outerShapeSize}px
          <input type="range" bind:value={outerShapeSize} min="150" max="290" step="10" />
        </label>
        <label>
          Color:
          <input type="color" bind:value={outerColor} />
        </label>
      </div>
    </div>
    
    <div class="control-section">
      <h3>Inner Pattern</h3>
      <div class="control-group">
        <label>
          Shape Type:
          <select bind:value={innerShape}>
            <option value="cube">Isometric Cube</option>
            <option value="cylinder">Isometric Cylinder</option>
            <option value="pyramid">Isometric Pyramid</option>
            <option value="circle">Circle</option>
            <option value="square">Square</option>
            <option value="triangle">Triangle</option>
            <option value="diamond">Diamond</option>
          </select>
        </label>
        <label>
          Shape Size: {innerShapeSize}px
          <input type="range" bind:value={innerShapeSize} min="15" max="50" step="5" />
        </label>
        <label>
          Spacing: {innerShapeSpacing}px {innerShapeSpacing < innerShapeSize ? '(overlapping)' : ''}
          <input type="range" bind:value={innerShapeSpacing} min="10" max="100" step="5" />
        </label>
        <label>
          Color:
          <input type="color" bind:value={innerPatternColor} />
        </label>
      </div>
      
      <h3>Pattern Transform</h3>
      <div class="control-group">
        <label>
          Rotation: {innerRotation}°
          <input type="range" bind:value={innerRotation} min="0" max="360" step="5" />
        </label>
        <label>
          Isometric Pitch: {innerPitch}°
          <input type="range" bind:value={innerPitch} min="0" max="60" step="5" />
        </label>
      </div>
    </div>
    
    <div class="control-section">
      <h3>Outer Borders</h3>
      <div class="control-group">
        <label>
          Border Lines: {outerBorderCount}
          <input type="range" bind:value={outerBorderCount} min="1" max="5" step="1" />
        </label>
        <label>
          Border Spacing: {outerBorderSpacing}px
          <input type="range" bind:value={outerBorderSpacing} min="5" max="30" step="5" />
        </label>
        <label>
          Color:
          <input type="color" bind:value={innerPatternColor} />
        </label>
      </div>
    </div>
    
    <div class="control-section">
      <h3>Center Design</h3>
      <div class="control-group">
        <label>
          Shape Type:
          <select bind:value={centerShape}>
            <option value="circles">Concentric Circles</option>
            <option value="square">Square</option>
            <option value="star">Star</option>
            <option value="diamond">Diamond</option>
            <option value="flower">Flower</option>
          </select>
        </label>
        <label>
          Square Void Size: {centerVoidSize}px
          <input type="range" bind:value={centerVoidSize} min="120" max="280" step="10" />
        </label>
        <label>
          Center Size: {centerShapeSize}px
          <input type="range" bind:value={centerShapeSize} min="40" max="100" step="5" />
        </label>
        <label>
          Color:
          <input type="color" bind:value={centerColor} />
        </label>
      </div>
    </div>
    
    <div class="control-section">
      <h3>General</h3>
      <div class="control-group">
        <label>
          Background:
          <input type="color" bind:value={backgroundColor} />
        </label>
      </div>
    </div>
    
    <div class="info">
      <p><strong>Tip:</strong> The center square void is designed to hold a QR code. The center design overlays in the middle of the QR code (safe zone).</p>
      <p>Use rotation and pitch to adjust isometric shapes. Add multiple borders for decorative effects. Set spacing below shape size for overlapping patterns.</p>
    </div>
  </div>
  
  <div class="canvas-container">
    <svg 
      width={canvasSize} 
      height={canvasSize} 
      viewBox={`0 0 ${canvasSize} ${canvasSize}`}
      xmlns="http://www.w3.org/2000/svg"
    >
      <!-- Define clipping paths -->
      <defs>
        <!-- Outer shape clip path with unique ID -->
        <clipPath id={clipPathId}>
          {@html getOuterClipPath()}
        </clipPath>
      </defs>
      
      <!-- Background -->
      <rect width={canvasSize} height={canvasSize} fill={backgroundColor} />
      
      <!-- Inner pattern with clipping and rotation -->
      <g clip-path={`url(#${clipPathId})`}>
        <g transform={`rotate(${innerRotation} ${centerX} ${centerY})`}>
          {#each innerPatternShapes as shape}
            {#if innerShape === 'cube'}
              {@const paths = getCubePath(shape.x, shape.y, innerShapeSize, innerPitch)}
              <path d={paths.top} fill={innerPatternColor} opacity="0.9" />
              <path d={paths.left} fill={innerPatternColor} opacity="0.6" />
              <path d={paths.right} fill={innerPatternColor} opacity="0.75" />
              <path d={paths.top} fill="none" stroke={backgroundColor} stroke-width="1" />
            {:else if innerShape === 'cylinder'}
              {@const paths = getCylinderPath(shape.x, shape.y, innerShapeSize, innerPitch)}
              <path d={paths.side} fill={innerPatternColor} opacity="0.75" />
              <path d={paths.bottom} fill={innerPatternColor} opacity="0.6" />
              <path d={paths.top} fill={innerPatternColor} opacity="0.9" />
              <path d={paths.top} fill="none" stroke={backgroundColor} stroke-width="1" />
            {:else if innerShape === 'pyramid'}
              {@const paths = getPyramidPath(shape.x, shape.y, innerShapeSize, innerPitch)}
              <path d={paths.left} fill={innerPatternColor} opacity="0.7" />
              <path d={paths.right} fill={innerPatternColor} opacity="0.85" />
              <path d={paths.left} fill="none" stroke={backgroundColor} stroke-width="1" />
              <path d={paths.right} fill="none" stroke={backgroundColor} stroke-width="1" />
            {:else if innerShape === 'circle'}
              <circle 
                cx={shape.x} 
                cy={shape.y} 
                r={innerShapeSize / 2} 
                fill={innerPatternColor}
                opacity="0.8"
              />
            {:else if innerShape === 'square'}
              <rect 
                x={shape.x - innerShapeSize / 2} 
                y={shape.y - innerShapeSize / 2} 
                width={innerShapeSize} 
                height={innerShapeSize} 
                fill={innerPatternColor}
                opacity="0.8"
              />
            {:else if innerShape === 'triangle'}
              <path 
                d={getTrianglePath(shape.x, shape.y, innerShapeSize)} 
                fill={innerPatternColor}
                opacity="0.8"
              />
            {:else if innerShape === 'diamond'}
              <path 
                d={getDiamondPath(shape.x, shape.y, innerShapeSize)} 
                fill={innerPatternColor}
                opacity="0.8"
              />
            {/if}
          {/each}
        </g>
        
        <!-- Clip out the center void -->
        <rect 
          x={centerX - centerVoidSize / 2} 
          y={centerY - centerVoidSize / 2} 
          width={centerVoidSize} 
          height={centerVoidSize}
          fill={backgroundColor}
        />
      </g>
      
      <!-- Multiple outer shape borders -->
      {#each Array(outerBorderCount) as _, i}
        {@const borderSize = outerShapeSize - (i * outerBorderSpacing)}
        {@const opacity = 1 - (i * 0.15)}
        {#if borderSize > 50}
          {#if outerShape === 'circle'}
            <circle 
              cx={centerX} 
              cy={centerY} 
              r={borderSize} 
              fill="none" 
              stroke={outerColor} 
              stroke-width="4"
              opacity={opacity}
            />
          {:else if outerShape === 'square'}
            <rect 
              x={centerX - borderSize} 
              y={centerY - borderSize} 
              width={borderSize * 2} 
              height={borderSize * 2} 
              fill="none" 
              stroke={outerColor} 
              stroke-width="4"
              opacity={opacity}
            />
          {:else if outerShape === 'hexagon'}
            <polygon 
              points={getHexagonPoints(centerX, centerY, borderSize)} 
              fill="none" 
              stroke={outerColor} 
              stroke-width="4"
              opacity={opacity}
            />
          {:else if outerShape === 'octagon'}
            <polygon 
              points={getOctagonPoints(centerX, centerY, borderSize)} 
              fill="none" 
              stroke={outerColor} 
              stroke-width="4"
              opacity={opacity}
            />
          {/if}
        {/if}
      {/each}
      
      <!-- Center void outline (showing QR code area) -->
      <rect 
        x={centerX - centerVoidSize / 2} 
        y={centerY - centerVoidSize / 2} 
        width={centerVoidSize} 
        height={centerVoidSize} 
        fill="none"
        stroke="#475569"
        stroke-width="2"
        stroke-dasharray="5,5"
        opacity="0.5"
      />
      
      <!-- Center design -->
      <g class="center-design">
        {#if centerShape === 'circles'}
          <circle 
            cx={centerX} 
            cy={centerY} 
            r={centerShapeSize} 
            fill="none" 
            stroke={centerColor} 
            stroke-width="8"
          />
          <circle 
            cx={centerX} 
            cy={centerY} 
            r={centerShapeSize * 0.65} 
            fill="none" 
            stroke={centerColor} 
            stroke-width="6"
          />
          <circle 
            cx={centerX} 
            cy={centerY} 
            r={centerShapeSize * 0.3} 
            fill={centerColor}
            opacity="0.8"
          />
        {:else if centerShape === 'square'}
          <rect 
            x={centerX - centerShapeSize} 
            y={centerY - centerShapeSize} 
            width={centerShapeSize * 2} 
            height={centerShapeSize * 2} 
            fill="none" 
            stroke={centerColor} 
            stroke-width="6"
            transform="rotate(45 {centerX} {centerY})"
          />
          <rect 
            x={centerX - centerShapeSize * 0.6} 
            y={centerY - centerShapeSize * 0.6} 
            width={centerShapeSize * 1.2} 
            height={centerShapeSize * 1.2} 
            fill={centerColor}
            opacity="0.8"
            transform="rotate(45 {centerX} {centerY})"
          />
        {:else if centerShape === 'star'}
          <path 
            d={getStarPath(centerX, centerY, centerShapeSize)} 
            fill={centerColor}
            opacity="0.8"
            stroke={centerColor}
            stroke-width="3"
          />
        {:else if centerShape === 'diamond'}
          <path 
            d={getDiamondPath(centerX, centerY, centerShapeSize * 2)} 
            fill="none"
            stroke={centerColor}
            stroke-width="8"
          />
          <path 
            d={getDiamondPath(centerX, centerY, centerShapeSize)} 
            fill={centerColor}
            opacity="0.8"
          />
        {:else if centerShape === 'flower'}
          {@const petals = getFlowerPath(centerX, centerY, centerShapeSize)}
          {#each petals as petal}
            <circle 
              cx={petal.cx} 
              cy={petal.cy} 
              r={petal.r} 
              fill={centerColor}
              opacity="0.7"
            />
          {/each}
          <circle 
            cx={centerX} 
            cy={centerY} 
            r={centerShapeSize * 0.3} 
            fill={centerColor}
          />
        {/if}
      </g>
    </svg>
  </div>
</div>

<style>
  .app {
    display: flex;
    min-height: 100vh;
    background: #0f172a;
    color: #e2e8f0;
    font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  }
  
  .controls {
    width: 350px;
    padding: 2rem;
    background: #1e293b;
    overflow-y: auto;
    border-right: 2px solid #334155;
  }
  
  h2 {
    margin: 0 0 2rem 0;
    font-size: 1.5rem;
    color: #60a5fa;
  }
  
  h3 {
    margin: 0 0 1rem 0;
    font-size: 1.1rem;
    color: #94a3b8;
    border-bottom: 1px solid #334155;
    padding-bottom: 0.5rem;
  }
  
  .control-section {
    margin-bottom: 2rem;
  }
  
  .control-group {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }
  
  label {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
    font-size: 0.875rem;
    color: #cbd5e1;
  }
  
  select, input[type="range"], input[type="color"] {
    padding: 0.5rem;
    background: #0f172a;
    border: 1px solid #475569;
    border-radius: 4px;
    color: #e2e8f0;
    font-size: 0.875rem;
  }
  
  input[type="range"] {
    cursor: pointer;
    accent-color: #60a5fa;
  }
  
  input[type="color"] {
    height: 40px;
    cursor: pointer;
  }
  
  select {
    cursor: pointer;
  }
  
  select:focus, input:focus {
    outline: 2px solid #60a5fa;
    outline-offset: 2px;
  }
  
  .canvas-container {
    flex: 1;
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 2rem;
  }
  
  svg {
    border-radius: 8px;
    box-shadow: 0 20px 25px -5px rgb(0 0 0 / 0.5);
  }
  
  .info {
    background: #334155;
    padding: 1rem;
    border-radius: 6px;
    font-size: 0.875rem;
    color: #94a3b8;
    line-height: 1.5;
  }
  
  .info strong {
    color: #60a5fa;
  }
  
  @media (max-width: 1024px) {
    .app {
      flex-direction: column;
    }
    
    .controls {
      width: 100%;
      border-right: none;
      border-bottom: 2px solid #334155;
    }
  }
</style>
