<script>
  import { onMount } from 'svelte';
  import QRCode from './QRCode.svelte';
  import GeometricFrame from './GeometricFrame.svelte';
  import QRLogo from './QRLogo.svelte';

  let url = 'https://example.com';

  // Designer mode: 'combined', 'qr-only', 'frame-only'
  let designerMode = 'combined';

  // Collapsible sections
  let qrControlsOpen = true;
  let frameControlsOpen = true;
  let centerControlsOpen = false;
  
  // QR Configuration
  let qrConfig = {
    roundingAmount: 0.45,
    paddingAmount: 0,
    edgeBleed: 0,
    geometricChaos: 0,
    finderOuterShape: 'rounded-square',
    finderInnerShape: 'rounded-square',
    finderCenterOverlap: 0,
    centerVoidRadius: 0.15
  };
  
  // Frame Configuration
  let frameConfig = {
    outerShape: 'hexagon',
    outerShapeSize: 480,
    outerBorderCount: 3,
    outerBorderSpacing: 15,
    outerColor: '#000000',

    innerShapeType: 'cube',
    innerShapeSize: 15,
    innerShapeSpacing: 18,
    innerRotation: 0,
    innerPitch: 30,
    innerColor: '#000000',

    centerVoidSize: 300,
    centerVoidRounding: 0,
    centerLogoShape: 'circles',
    centerLogoSize: 60,
    centerLogoColor: '#000000',

    // Pass through module styling
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
  
  // Component references
  let qrComponent;
  let frameComponent;
  let logoComponent;
  
  // Canvas references
  let qrCanvas;
  let frameCanvas;
  let logoCanvas;
  
  // Display canvases
  let qrDisplayCanvas;
  let frameDisplayCanvas;
  let logoDisplayCanvas;
  
  let initialized = false;
  let qrLibraryReady = false;
  let debounceTimer;
  let isRendering = false;

  // Debounced regenerate for real-time updates
  // Use longer debounce for artistic mode to improve performance
  function debouncedRegenerate() {
    if (!qrLibraryReady) return;
    clearTimeout(debounceTimer);
    const debounceTime = frameConfig.artisticEnabled ? 300 : 50;
    debounceTimer = setTimeout(() => {
      regenerate();
    }, debounceTime);
  }

  // Track config changes by serializing (needed for deep reactivity in Svelte 5)
  $: qrConfigJson = JSON.stringify(qrConfig);
  $: frameConfigJson = JSON.stringify(frameConfig);

  // Reactive statement to trigger updates when configs change
  $: if (qrLibraryReady && (url || qrConfigJson || frameConfigJson)) {
    debouncedRegenerate();
  }

  onMount(() => {
    // Give components time to mount
    setTimeout(() => {
      initialized = true;
      // Wait for QR library to load with retry
      waitForQRLibrary();
    }, 300);
  });
  
  function waitForQRLibrary(attempts = 0) {
    // qrcode-generator exports as window.qrcode
    if (window.qrcode) {
      console.log('QR library detected');
      qrLibraryReady = true;
      regenerate();
    } else if (attempts < 20) {
      // Retry every 250ms for up to 5 seconds
      setTimeout(() => waitForQRLibrary(attempts + 1), 250);
    } else {
      console.error('QR library failed to load after 5 seconds');
    }
  }
  
  function regenerate() {
    if (!initialized) {
      console.warn('Not initialized yet');
      return;
    }
    
    console.log('Regenerating all components...');
    
    // Sync module styling to frame config
    frameConfig.roundingAmount = qrConfig.roundingAmount;
    frameConfig.paddingAmount = qrConfig.paddingAmount;
    frameConfig.edgeBleed = qrConfig.edgeBleed;
    frameConfig.geometricChaos = qrConfig.geometricChaos;
    
    // Trigger re-renders
    if (qrComponent) {
      console.log('Generating QR...');
      qrComponent.generateQR();
    }
    
    if (frameComponent) {
      console.log('Rendering frame...');
      frameComponent.render();
    }
    
    // Wait for renders to complete, then composite and update displays
    setTimeout(() => {
      if (logoComponent) {
        console.log('Compositing...');
        logoComponent.composite();
      }
      updateDisplayCanvases();
    }, 200);
  }
  
  function updateDisplayCanvases() {
    console.log('Updating display canvases...', {
      hasQRCanvas: !!qrCanvas,
      hasFrameCanvas: !!frameCanvas,
      hasLogoCanvas: !!logoCanvas,
      qrCanvasSize: qrCanvas ? `${qrCanvas.width}x${qrCanvas.height}` : 'N/A',
      frameCanvasSize: frameCanvas ? `${frameCanvas.width}x${frameCanvas.height}` : 'N/A',
      logoCanvasSize: logoCanvas ? `${logoCanvas.width}x${logoCanvas.height}` : 'N/A'
    });
    
    if (qrCanvas && qrDisplayCanvas) {
      const ctx = qrDisplayCanvas.getContext('2d');
      qrDisplayCanvas.width = qrCanvas.width;
      qrDisplayCanvas.height = qrCanvas.height;
      ctx.clearRect(0, 0, qrDisplayCanvas.width, qrDisplayCanvas.height);
      ctx.drawImage(qrCanvas, 0, 0);
      console.log('QR display canvas updated');
    }
    
    if (frameCanvas && frameDisplayCanvas) {
      const ctx = frameDisplayCanvas.getContext('2d');
      frameDisplayCanvas.width = frameCanvas.width;
      frameDisplayCanvas.height = frameCanvas.height;
      ctx.clearRect(0, 0, frameDisplayCanvas.width, frameDisplayCanvas.height);
      ctx.drawImage(frameCanvas, 0, 0);
      console.log('Frame display canvas updated');
    }
    
    if (logoCanvas && logoDisplayCanvas) {
      const ctx = logoDisplayCanvas.getContext('2d');
      logoDisplayCanvas.width = logoCanvas.width;
      logoDisplayCanvas.height = logoCanvas.height;
      ctx.clearRect(0, 0, logoDisplayCanvas.width, logoDisplayCanvas.height);
      ctx.drawImage(logoCanvas, 0, 0);
      console.log('Logo display canvas updated');
    }
  }
  
  function downloadQR() {
    if (!qrDisplayCanvas) return;
    const link = document.createElement('a');
    link.download = 'qr-code.png';
    link.href = qrDisplayCanvas.toDataURL();
    link.click();
  }
  
  function downloadFrame() {
    if (!frameDisplayCanvas) return;
    const link = document.createElement('a');
    link.download = 'geometric-frame.png';
    link.href = frameDisplayCanvas.toDataURL();
    link.click();
  }
  
  function downloadLogo() {
    if (!logoDisplayCanvas) return;
    const link = document.createElement('a');
    link.download = 'qr-logo.png';
    link.href = logoDisplayCanvas.toDataURL();
    link.click();
  }

  function switchMode(mode) {
    designerMode = mode;
    // Regenerate when switching modes
    debouncedRegenerate();
  }

  function openInMainDesigner() {
    // Transfer current settings to the combined designer
    designerMode = 'combined';
    debouncedRegenerate();
  }
</script>

<svelte:head>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700;800&display=swap" rel="stylesheet">
</svelte:head>

<!-- Hidden components that do the rendering -->
<QRCode bind:this={qrComponent} bind:canvas={qrCanvas} {url} config={qrConfig} />
<GeometricFrame bind:this={frameComponent} bind:canvas={frameCanvas} config={frameConfig} />
<QRLogo bind:this={logoComponent} bind:canvas={logoCanvas} {qrCanvas} {frameCanvas} {frameConfig} />

<div class="app">
  <!-- Navigation Menu -->
  <nav class="designer-nav">
    <button
      class="nav-button"
      class:active={designerMode === 'combined'}
      on:click={() => switchMode('combined')}
    >
      Combined Designer
    </button>
    <button
      class="nav-button"
      class:active={designerMode === 'qr-only'}
      on:click={() => switchMode('qr-only')}
    >
      QR Code Only
    </button>
    <button
      class="nav-button"
      class:active={designerMode === 'frame-only'}
      on:click={() => switchMode('frame-only')}
    >
      Frame Only
    </button>
  </nav>

  <div class="main-layout">
    <!-- Left Panel: Controls -->
    <div class="controls-panel">
      <h1 class="title">
        {#if designerMode === 'combined'}
          QR Designer
        {:else if designerMode === 'qr-only'}
          QR Code Designer
        {:else}
          Frame Designer
        {/if}
      </h1>

      <!-- URL Input - shown in combined and qr-only modes -->
      {#if designerMode === 'combined' || designerMode === 'qr-only'}
      <div class="url-section">
        <label class="input-label">
          QR Code URL
          <input
            type="text"
            bind:value={url}
            placeholder="Enter URL here..."
            class="url-input"
          />
        </label>
      </div>
      {/if}

      <!-- QR Module Controls - shown in combined and qr-only modes -->
      {#if designerMode === 'combined' || designerMode === 'qr-only'}
      <div class="controls-section">
        <button class="section-header" on:click={() => qrControlsOpen = !qrControlsOpen}>
          <h2 class="controls-title">QR Module Styling</h2>
          <span class="toggle-icon">{qrControlsOpen ? '−' : '+'}</span>
        </button>

        {#if qrControlsOpen}
          <div class="controls-content">
            <p class="section-note">These controls affect both QR modules and all frame inner patterns</p>

            <div class="control-group">
              <label class="control-label">
                Rounding: <span class="value">{qrConfig.roundingAmount.toFixed(2)}</span>
                <input
                  type="range"
                  bind:value={qrConfig.roundingAmount}
                  min="0"
                  max="1"
                  step="0.05"
                  class="slider"
                />
              </label>
            </div>

            <div class="control-group">
              <label class="control-label">
                Padding: <span class="value">{qrConfig.paddingAmount.toFixed(2)}</span>
                <input
                  type="range"
                  bind:value={qrConfig.paddingAmount}
                  min="0"
                  max="0.4"
                  step="0.05"
                  class="slider"
                />
              </label>
            </div>

            <div class="control-group">
              <label class="control-label">
                Edge Bleed: <span class="value">{qrConfig.edgeBleed.toFixed(2)}</span>
                <input
                  type="range"
                  bind:value={qrConfig.edgeBleed}
                  min="0"
                  max="1"
                  step="0.1"
                  class="slider green"
                />
              </label>
            </div>

            <div class="control-group">
              <label class="control-label">
                Geometric Chaos: <span class="value">{qrConfig.geometricChaos.toFixed(2)}</span>
                <input
                  type="range"
                  bind:value={qrConfig.geometricChaos}
                  min="0"
                  max="1"
                  step="0.1"
                  class="slider green"
                />
              </label>
            </div>

            <h3 class="subsection-title">Finder Patterns</h3>

            <div class="control-group">
              <label class="control-label">
                Outer Shape
                <select bind:value={qrConfig.finderOuterShape} class="select">
                  <option value="rounded-square">Rounded Square</option>
                  <option value="circle">Circle</option>
                </select>
              </label>
            </div>

            <div class="control-group">
              <label class="control-label">
                Inner Shape
                <select bind:value={qrConfig.finderInnerShape} class="select">
                  <option value="rounded-square">Rounded Square</option>
                  <option value="circle">Circle</option>
                </select>
              </label>
            </div>

            <div class="control-group">
              <label class="control-label">
                Center Overlap: <span class="value">{qrConfig.finderCenterOverlap.toFixed(2)}</span>
                <input
                  type="range"
                  bind:value={qrConfig.finderCenterOverlap}
                  min="-0.5"
                  max="0.5"
                  step="0.05"
                  class="slider blue"
                />
              </label>
            </div>
          </div>
        {/if}
      </div>
      {/if}

      <!-- Frame Controls - shown in combined and frame-only modes -->
      {#if designerMode === 'combined' || designerMode === 'frame-only'}
      <div class="controls-section frame">
        <button class="section-header" on:click={() => frameControlsOpen = !frameControlsOpen}>
          <h2 class="controls-title">Geometric Frame</h2>
          <span class="toggle-icon">{frameControlsOpen ? '−' : '+'}</span>
        </button>

        {#if frameControlsOpen}
          <div class="controls-content">
            <h3 class="subsection-title">Outer Shape</h3>

            <div class="control-group">
              <label class="control-label">
                Shape Type
                <select bind:value={frameConfig.outerShape} class="select">
                  <option value="hexagon">Hexagon</option>
                  <option value="circle">Circle</option>
                  <option value="square">Square</option>
                  <option value="octagon">Octagon</option>
                </select>
              </label>
            </div>

            <div class="control-group">
              <label class="control-label">
                Size: <span class="value">{frameConfig.outerShapeSize}px</span>
                <input
                  type="range"
                  bind:value={frameConfig.outerShapeSize}
                  min="300"
                  max="490"
                  step="10"
                  class="slider cyan"
                />
              </label>
            </div>

            <div class="control-group">
              <label class="control-label">
                Border Lines: <span class="value">{frameConfig.outerBorderCount}</span>
                <input
                  type="range"
                  bind:value={frameConfig.outerBorderCount}
                  min="1"
                  max="5"
                  step="1"
                  class="slider cyan"
                />
              </label>
            </div>

            <div class="control-group">
              <label class="control-label">
                Border Spacing: <span class="value">{frameConfig.outerBorderSpacing}px</span>
                <input
                  type="range"
                  bind:value={frameConfig.outerBorderSpacing}
                  min="5"
                  max="30"
                  step="5"
                  class="slider cyan"
                />
              </label>
            </div>

            <h3 class="subsection-title">Inner Pattern</h3>

            <div class="control-group">
              <label class="control-label">
                Pattern Type
                <select bind:value={frameConfig.innerShapeType} class="select">
                  <option value="module-based">Module Based</option>
                  <option value="cube">Isometric Cube</option>
                  <option value="cylinder">Isometric Cylinder</option>
                  <option value="pyramid">Isometric Pyramid</option>
                  <option value="circle">Circle</option>
                  <option value="square">Square</option>
                  <option value="diamond">Diamond</option>
                </select>
              </label>
            </div>

            <p class="section-note">All pattern types are styled by QR Module controls (rounding, padding, edge bleed, chaos)</p>

            <div class="control-group">
              <label class="control-label">
                Shape Size: <span class="value">{frameConfig.innerShapeSize}px</span>
                <input
                  type="range"
                  bind:value={frameConfig.innerShapeSize}
                  min="5"
                  max="100"
                  step="5"
                  class="slider cyan"
                />
              </label>
            </div>

            <div class="control-group">
              <label class="control-label">
                Spacing: <span class="value">{frameConfig.innerShapeSpacing}px</span>
                <input
                  type="range"
                  bind:value={frameConfig.innerShapeSpacing}
                  min="5"
                  max="100"
                  step="5"
                  class="slider cyan"
                />
              </label>
            </div>

            <div class="control-group">
              <label class="control-label">
                Rotation: <span class="value">{frameConfig.innerRotation}°</span>
                <input
                  type="range"
                  bind:value={frameConfig.innerRotation}
                  min="0"
                  max="360"
                  step="5"
                  class="slider cyan"
                />
              </label>
            </div>

            {#if frameConfig.innerShapeType === 'cube' || frameConfig.innerShapeType === 'cylinder' || frameConfig.innerShapeType === 'pyramid'}
              <div class="control-group">
                <label class="control-label">
                  Isometric Pitch: <span class="value">{frameConfig.innerPitch}°</span>
                  <input
                    type="range"
                    bind:value={frameConfig.innerPitch}
                    min="0"
                    max="60"
                    step="5"
                    class="slider cyan"
                  />
                </label>
              </div>
            {/if}

            <h3 class="subsection-title">Artistic Rendering</h3>

            <div class="control-group">
              <label class="control-label checkbox-label">
                <input
                  type="checkbox"
                  bind:checked={frameConfig.artisticEnabled}
                  class="checkbox"
                />
                Enable Artistic Mode
              </label>
            </div>

            {#if frameConfig.artisticEnabled}
              <p class="section-note">Artistic mode applies hand-drawn, sketchy effects to the inner pattern</p>
              <p class="section-note performance-tip">⚡ Performance tip: Use 'Solid' or 'Hachure' fill styles and increase spacing (25+) for faster rendering</p>

              <div class="control-group">
                <label class="control-label">
                  Fill Style
                  <select bind:value={frameConfig.artisticFillStyle} class="select">
                    <option value="solid">Solid (Fastest)</option>
                    <option value="hachure">Hachure (Recommended)</option>
                    <option value="zigzag">Zigzag</option>
                    <option value="cross-hatch">Cross-Hatch Dense</option>
                    <option value="dots">Dots</option>
                    <option value="dashed">Dashed</option>
                    <option value="zigzag-line">Zigzag Line</option>
                  </select>
                </label>
              </div>

              <div class="control-group">
                <label class="control-label">
                  Roughness: <span class="value">{frameConfig.artisticRoughness.toFixed(1)}</span>
                  <input
                    type="range"
                    bind:value={frameConfig.artisticRoughness}
                    min="0"
                    max="5"
                    step="0.5"
                    class="slider magenta"
                  />
                </label>
              </div>

              <div class="control-group">
                <label class="control-label">
                  Bowing: <span class="value">{frameConfig.artisticBowing.toFixed(1)}</span>
                  <input
                    type="range"
                    bind:value={frameConfig.artisticBowing}
                    min="0"
                    max="10"
                    step="0.5"
                    class="slider magenta"
                  />
                </label>
              </div>

              <div class="control-group">
                <label class="control-label">
                  Fill Weight: <span class="value">{frameConfig.artisticFillWeight}</span>
                  <input
                    type="range"
                    bind:value={frameConfig.artisticFillWeight}
                    min="1"
                    max="10"
                    step="1"
                    class="slider magenta"
                  />
                </label>
              </div>
            {/if}
          </div>
        {/if}
      </div>
      {/if}

      <!-- Center Logo Controls - shown in combined and frame-only modes -->
      {#if designerMode === 'combined' || designerMode === 'frame-only'}
      <div class="controls-section frame">
        <button class="section-header" on:click={() => centerControlsOpen = !centerControlsOpen}>
          <h2 class="controls-title">Center Logo</h2>
          <span class="toggle-icon">{centerControlsOpen ? '−' : '+'}</span>
        </button>

        {#if centerControlsOpen}
          <div class="controls-content">
            <div class="control-group">
              <label class="control-label">
                Logo Shape
                <select bind:value={frameConfig.centerLogoShape} class="select">
                  <option value="circles">Concentric Circles</option>
                  <option value="square">Rotated Squares</option>
                  <option value="star">Star</option>
                  <option value="diamond">Diamond</option>
                </select>
              </label>
            </div>

            <div class="control-group">
              <label class="control-label">
                Logo Size: <span class="value">{frameConfig.centerLogoSize}px</span>
                <input
                  type="range"
                  bind:value={frameConfig.centerLogoSize}
                  min="30"
                  max="100"
                  step="5"
                  class="slider cyan"
                />
              </label>
            </div>

            <div class="control-group">
              <label class="control-label">
                QR Void Size: <span class="value">{frameConfig.centerVoidSize}px</span>
                <input
                  type="range"
                  bind:value={frameConfig.centerVoidSize}
                  min="200"
                  max="900"
                  step="10"
                  class="slider cyan"
                />
              </label>
            </div>

            <div class="control-group">
              <label class="control-label">
                Void Corner Rounding: <span class="value">{(frameConfig.centerVoidRounding * 100).toFixed(0)}%</span>
                <input
                  type="range"
                  bind:value={frameConfig.centerVoidRounding}
                  min="0"
                  max="1"
                  step="0.05"
                  class="slider cyan"
                />
              </label>
            </div>
          </div>
        {/if}
      </div>
      {/if}

      <!-- QR Styling Controls for Frame-only mode -->
      {#if designerMode === 'frame-only'}
      <div class="controls-section">
        <button class="section-header" on:click={() => qrControlsOpen = !qrControlsOpen}>
          <h2 class="controls-title">Pattern Styling (QR Effects)</h2>
          <span class="toggle-icon">{qrControlsOpen ? '−' : '+'}</span>
        </button>

        {#if qrControlsOpen}
          <div class="controls-content">
            <p class="section-note">These controls affect how the inner frame patterns are rendered</p>

            <div class="control-group">
              <label class="control-label">
                Rounding: <span class="value">{qrConfig.roundingAmount.toFixed(2)}</span>
                <input
                  type="range"
                  bind:value={qrConfig.roundingAmount}
                  min="0"
                  max="1"
                  step="0.05"
                  class="slider"
                />
              </label>
            </div>

            <div class="control-group">
              <label class="control-label">
                Padding: <span class="value">{qrConfig.paddingAmount.toFixed(2)}</span>
                <input
                  type="range"
                  bind:value={qrConfig.paddingAmount}
                  min="0"
                  max="0.4"
                  step="0.05"
                  class="slider"
                />
              </label>
            </div>

            <div class="control-group">
              <label class="control-label">
                Edge Bleed: <span class="value">{qrConfig.edgeBleed.toFixed(2)}</span>
                <input
                  type="range"
                  bind:value={qrConfig.edgeBleed}
                  min="0"
                  max="1"
                  step="0.1"
                  class="slider green"
                />
              </label>
            </div>

            <div class="control-group">
              <label class="control-label">
                Geometric Chaos: <span class="value">{qrConfig.geometricChaos.toFixed(2)}</span>
                <input
                  type="range"
                  bind:value={qrConfig.geometricChaos}
                  min="0"
                  max="1"
                  step="0.1"
                  class="slider green"
                />
              </label>
            </div>
          </div>
        {/if}
      </div>
      {/if}

      <!-- Open in Main Designer Button -->
      {#if designerMode !== 'combined'}
      <button class="main-designer-button" on:click={openInMainDesigner}>
        Open in Main Designer
      </button>
      {/if}
    </div>

    <!-- Right Panel: Previews -->
    <div class="preview-panel">
      <h2 class="preview-header">Live Preview</h2>

      {#if designerMode === 'combined'}
        <!-- Main Combined Preview -->
        <div class="main-preview">
          <div class="canvas-wrapper large">
            <canvas bind:this={logoDisplayCanvas} class="canvas"></canvas>
          </div>
          <button class="download-button" on:click={downloadLogo}>
            Download QR Logo
          </button>
        </div>

        <!-- Secondary Previews -->
        <div class="secondary-previews">
          <div class="preview-card">
            <h3 class="preview-title">QR Code</h3>
            <div class="canvas-wrapper small">
              <canvas bind:this={qrDisplayCanvas} class="canvas"></canvas>
            </div>
            <button class="download-button small" on:click={downloadQR}>
              Download
            </button>
          </div>

          <div class="preview-card">
            <h3 class="preview-title">Frame</h3>
            <div class="canvas-wrapper small">
              <canvas bind:this={frameDisplayCanvas} class="canvas"></canvas>
            </div>
            <button class="download-button small" on:click={downloadFrame}>
              Download
            </button>
          </div>
        </div>
      {:else if designerMode === 'qr-only'}
        <!-- QR Only Preview -->
        <div class="main-preview">
          <div class="canvas-wrapper large">
            <canvas bind:this={qrDisplayCanvas} class="canvas"></canvas>
          </div>
          <button class="download-button" on:click={downloadQR}>
            Download QR Code
          </button>
        </div>
      {:else}
        <!-- Frame Only Preview -->
        <div class="main-preview">
          <div class="canvas-wrapper large">
            <canvas bind:this={frameDisplayCanvas} class="canvas"></canvas>
          </div>
          <button class="download-button" on:click={downloadFrame}>
            Download Frame
          </button>
        </div>
      {/if}
    </div>
  </div>
</div>

<style>
  * {
    box-sizing: border-box;
  }

  :global(body) {
    margin: 0;
    padding: 0;
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
  }

  .app {
    min-height: 100vh;
    background: linear-gradient(135deg, #0f172a 0%, #1e1b4b 50%, #0f172a 100%);
  }

  /* Navigation Menu */
  .designer-nav {
    display: flex;
    justify-content: center;
    gap: 0.5rem;
    padding: 1rem;
    background: rgba(0, 0, 0, 0.4);
    border-bottom: 1px solid rgba(167, 139, 250, 0.2);
    flex-wrap: wrap;
  }

  .nav-button {
    background: rgba(255, 255, 255, 0.05);
    border: 1px solid rgba(167, 139, 250, 0.3);
    color: #94a3b8;
    padding: 0.625rem 1.25rem;
    font-size: 0.85rem;
    font-weight: 600;
    border-radius: 8px;
    cursor: pointer;
    transition: all 0.2s ease;
  }

  .nav-button:hover {
    background: rgba(167, 139, 250, 0.1);
    color: #e2e8f0;
    border-color: rgba(167, 139, 250, 0.5);
  }

  .nav-button.active {
    background: linear-gradient(135deg, #6366f1 0%, #a78bfa 100%);
    color: #fff;
    border-color: transparent;
    box-shadow: 0 4px 15px rgba(99, 102, 241, 0.4);
  }

  .main-layout {
    display: grid;
    grid-template-columns: 1fr;
    min-height: calc(100vh - 60px);
  }

  @media (min-width: 900px) {
    .main-layout {
      grid-template-columns: 380px 1fr;
    }
  }

  @media (min-width: 1200px) {
    .main-layout {
      grid-template-columns: 420px 1fr;
    }
  }

  /* Left Panel - Controls */
  .controls-panel {
    background: rgba(0, 0, 0, 0.3);
    border-right: 1px solid rgba(167, 139, 250, 0.2);
    padding: 1.5rem;
    overflow-y: auto;
    max-height: 100vh;
  }

  @media (max-width: 899px) {
    .controls-panel {
      max-height: none;
      border-right: none;
      border-bottom: 1px solid rgba(167, 139, 250, 0.2);
    }
  }

  .title {
    color: #fff;
    font-size: 1.75rem;
    font-weight: 800;
    margin: 0 0 1.5rem 0;
    background: linear-gradient(135deg, #60a5fa 0%, #a78bfa 50%, #f472b6 100%);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
  }

  .url-section {
    background: rgba(255, 255, 255, 0.05);
    border-radius: 12px;
    padding: 1rem;
    margin-bottom: 1rem;
    border: 1px solid rgba(167, 139, 250, 0.2);
  }

  .input-label {
    display: block;
    color: #e2e8f0;
    font-size: 0.75rem;
    font-weight: 600;
    margin-bottom: 0.5rem;
    text-transform: uppercase;
    letter-spacing: 0.05em;
  }

  .url-input {
    width: 100%;
    padding: 0.75rem;
    background: rgba(0, 0, 0, 0.4);
    border: 1px solid rgba(167, 139, 250, 0.3);
    border-radius: 8px;
    color: #fff;
    font-size: 0.875rem;
    outline: none;
    transition: all 0.2s ease;
  }

  .url-input:focus {
    border-color: #a78bfa;
    box-shadow: 0 0 0 2px rgba(167, 139, 250, 0.1);
  }

  .controls-section {
    background: rgba(59, 130, 246, 0.05);
    border-radius: 12px;
    border: 1px solid rgba(59, 130, 246, 0.2);
    overflow: hidden;
    margin-bottom: 0.75rem;
  }

  .controls-section.frame {
    background: rgba(6, 182, 212, 0.05);
    border-color: rgba(6, 182, 212, 0.2);
  }

  .section-header {
    width: 100%;
    background: none;
    border: none;
    padding: 0.875rem 1rem;
    display: flex;
    justify-content: space-between;
    align-items: center;
    cursor: pointer;
    transition: background 0.2s;
  }

  .section-header:hover {
    background: rgba(255, 255, 255, 0.02);
  }

  .controls-title {
    color: #3b82f6;
    font-size: 0.9rem;
    font-weight: 700;
    margin: 0;
    text-align: left;
  }

  .frame .controls-title {
    color: #06b6d4;
  }

  .toggle-icon {
    color: #94a3b8;
    font-size: 1.25rem;
    font-weight: 300;
  }

  .controls-content {
    padding: 0 1rem 1rem;
  }

  .section-note {
    background: rgba(139, 92, 246, 0.1);
    border-left: 2px solid #8b5cf6;
    padding: 0.5rem 0.75rem;
    margin-bottom: 0.75rem;
    border-radius: 4px;
    color: #c4b5fd;
    font-size: 0.75rem;
    margin-top: 0;
  }

  .section-note.performance-tip {
    background: rgba(234, 179, 8, 0.1);
    border-left: 2px solid #eab308;
    color: #fde047;
  }

  .subsection-title {
    color: #06b6d4;
    font-size: 0.7rem;
    font-weight: 700;
    text-transform: uppercase;
    letter-spacing: 0.05em;
    margin: 1rem 0 0.75rem 0;
    padding-bottom: 0.375rem;
    border-bottom: 1px solid rgba(6, 182, 212, 0.2);
  }

  .control-group {
    margin-bottom: 0.875rem;
  }

  .control-label {
    display: block;
    color: #e2e8f0;
    font-size: 0.8rem;
    font-weight: 500;
    margin-bottom: 0.375rem;
  }

  .value {
    color: #3b82f6;
    font-family: 'Courier New', monospace;
    font-size: 0.75rem;
  }

  .slider {
    width: 100%;
    height: 5px;
    border-radius: 2.5px;
    background: rgba(255, 255, 255, 0.1);
    outline: none;
    -webkit-appearance: none;
  }

  .slider::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 16px;
    height: 16px;
    border-radius: 50%;
    background: linear-gradient(135deg, #3b82f6 0%, #8b5cf6 100%);
    cursor: pointer;
    box-shadow: 0 2px 4px rgba(59, 130, 246, 0.4);
  }

  .slider.green::-webkit-slider-thumb {
    background: linear-gradient(135deg, #10b981 0%, #14b8a6 100%);
  }

  .slider.blue::-webkit-slider-thumb {
    background: linear-gradient(135deg, #0ea5e9 0%, #3b82f6 100%);
  }

  .slider.cyan::-webkit-slider-thumb {
    background: linear-gradient(135deg, #06b6d4 0%, #0ea5e9 100%);
  }

  .slider.magenta::-webkit-slider-thumb {
    background: linear-gradient(135deg, #d946ef 0%, #c026d3 100%);
  }

  .checkbox {
    width: 1.25rem;
    height: 1.25rem;
    margin-right: 0.5rem;
    cursor: pointer;
    accent-color: #d946ef;
  }

  .checkbox-label {
    display: flex;
    align-items: center;
    cursor: pointer;
  }

  .select {
    width: 100%;
    padding: 0.625rem;
    background: rgba(0, 0, 0, 0.4);
    border: 1px solid rgba(167, 139, 250, 0.3);
    border-radius: 6px;
    color: #fff;
    font-size: 0.8rem;
    outline: none;
    cursor: pointer;
  }

  .select:focus {
    border-color: #a78bfa;
  }

  /* Right Panel - Previews */
  .preview-panel {
    padding: 1.5rem;
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: flex-start;
    overflow-y: auto;
  }

  @media (min-width: 900px) {
    .preview-panel {
      position: sticky;
      top: 0;
      max-height: 100vh;
    }
  }

  .preview-header {
    color: #e2e8f0;
    font-size: 1.25rem;
    font-weight: 700;
    margin: 0 0 1.5rem 0;
    text-align: center;
  }

  .main-preview {
    width: 100%;
    max-width: 500px;
    margin-bottom: 1.5rem;
  }

  .canvas-wrapper {
    background: linear-gradient(135deg, #f8fafc 0%, #f1f5f9 100%);
    border-radius: 12px;
    padding: 1rem;
    display: flex;
    justify-content: center;
    align-items: center;
    margin-bottom: 1rem;
  }

  .canvas-wrapper.large {
    padding: 1.5rem;
  }

  .canvas-wrapper.small {
    padding: 0.75rem;
  }

  .canvas {
    max-width: 100%;
    height: auto;
    display: block;
    border-radius: 8px;
  }

  .secondary-previews {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 1rem;
    width: 100%;
    max-width: 500px;
  }

  .preview-card {
    background: rgba(0, 0, 0, 0.2);
    border-radius: 10px;
    padding: 1rem;
    border: 1px solid rgba(255, 255, 255, 0.1);
  }

  .preview-title {
    color: #e2e8f0;
    font-size: 0.8rem;
    font-weight: 600;
    margin: 0 0 0.75rem 0;
    text-align: center;
  }

  .download-button {
    width: 100%;
    background: linear-gradient(135deg, #06b6d4 0%, #3b82f6 100%);
    color: #fff;
    border: none;
    padding: 0.875rem;
    font-size: 0.9rem;
    font-weight: 600;
    border-radius: 10px;
    cursor: pointer;
    transition: all 0.2s ease;
    box-shadow: 0 4px 15px rgba(6, 182, 212, 0.3);
  }

  .download-button:hover {
    transform: translateY(-1px);
    box-shadow: 0 6px 20px rgba(6, 182, 212, 0.4);
  }

  .download-button.small {
    padding: 0.625rem;
    font-size: 0.75rem;
  }

  .main-designer-button {
    width: 100%;
    background: linear-gradient(135deg, #8b5cf6 0%, #a78bfa 100%);
    color: #fff;
    border: none;
    padding: 1rem;
    font-size: 0.95rem;
    font-weight: 700;
    border-radius: 10px;
    cursor: pointer;
    transition: all 0.2s ease;
    box-shadow: 0 4px 15px rgba(139, 92, 246, 0.3);
    margin-top: 1rem;
  }

  .main-designer-button:hover {
    transform: translateY(-2px);
    box-shadow: 0 6px 20px rgba(139, 92, 246, 0.5);
  }

  @media (max-width: 899px) {
    .title {
      font-size: 1.5rem;
    }

    .main-preview {
      max-width: 400px;
    }

    .secondary-previews {
      max-width: 400px;
    }
  }
</style>
