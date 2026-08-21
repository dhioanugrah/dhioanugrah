import sys
import base64
import os
from PIL import Image, ImageOps, ImageEnhance

def image_to_retro_base64(image_path):
    try:
        img = Image.open(image_path)
        
        # Crop to square
        width, height = img.size
        min_dim = min(width, height)
        left = (width - min_dim)/2
        top = (height - min_dim)/2
        right = (width + min_dim)/2
        bottom = (height + min_dim)/2
        img = img.crop((left, top, right, bottom))
        
        # Resize down
        img = img.resize((150, 150), Image.Resampling.LANCZOS)
        
        # Convert to grayscale
        img = img.convert("L")
        
        # Invert since the background in the original photo is white
        img = ImageOps.invert(img)
        
        # Enhance contrast
        enhancer = ImageEnhance.Contrast(img)
        img = enhancer.enhance(1.5)
        
        # Apply a blue tint (duotone)
        # We can map grayscale to dark blue -> light cyan
        img = ImageOps.colorize(img, black="#0f172a", white="#00f0ff")
        
        # Save to bytes
        from io import BytesIO
        buffered = BytesIO()
        img.save(buffered, format="PNG")
        img_str = base64.b64encode(buffered.getvalue()).decode("utf-8")
        return f"data:image/png;base64,{img_str}"
    except Exception as e:
        print(f"Warning: Could not process image {image_path}: {e}")
        return None

def generate_svg(image_path):
    img_b64 = image_to_retro_base64(image_path) if image_path else ""
    
    # Image tag
    image_svg = ""
    if img_b64:
        image_svg = f'<image x="55" y="130" width="150" height="150" href="{img_b64}" preserveAspectRatio="xMidYMid slice" style="filter: brightness(1.2) contrast(1.5); opacity: 0.9;" />'
    
    svg = f"""<svg xmlns="http://www.w3.org/2000/svg" width="900" height="700">
    <defs>
        <pattern id="scanlines" width="4" height="4" patternUnits="userSpaceOnUse">
            <rect width="4" height="2" fill="#020617" opacity="0.3" />
            <rect y="2" width="4" height="2" fill="#000" opacity="0.1" />
        </pattern>
        <filter id="glow">
            <feGaussianBlur stdDeviation="2" result="coloredBlur"/>
            <feMerge>
                <feMergeNode in="coloredBlur"/>
                <feMergeNode in="SourceGraphic"/>
            </feMerge>
        </filter>
    </defs>
    
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Share+Tech+Mono&amp;display=swap');
        
        /* Font and Colors */
        .bg {{ fill: #060b19; }}
        .border {{ stroke: #1e3a8a; stroke-width: 2; fill: none; rx: 8; ry: 8; }}
        .title {{ font-family: 'Share Tech Mono', monospace; font-size: 28px; fill: #38bdf8; font-weight: bold; filter: url(#glow); }}
        .subtitle {{ font-family: 'Share Tech Mono', monospace; font-size: 18px; fill: #0ea5e9; font-weight: bold; letter-spacing: 1px; }}
        .text {{ font-family: 'Share Tech Mono', monospace; font-size: 14px; fill: #94a3b8; }}
        .highlight {{ fill: #00f0ff; font-weight: bold; }}
        .accent {{ fill: #3b82f6; }}
        
        /* Animations */
        @keyframes fadeIn {{
            from {{ opacity: 0; transform: translateY(10px); }}
            to {{ opacity: 1; transform: translateY(0); }}
        }}
        @keyframes blink {{
            0%, 100% {{ opacity: 1; }}
            50% {{ opacity: 0; }}
        }}
        @keyframes pulseDot {{
            0%, 100% {{ opacity: 0.3; transform: scale(1); }}
            50% {{ opacity: 1; transform: scale(1.2); }}
        }}
        @keyframes slideIn {{
            from {{ opacity: 0; transform: translateX(-20px); }}
            to {{ opacity: 1; transform: translateX(0); }}
        }}

        /* Applying Animations */
        .panel1 {{ animation: fadeIn 0.6s ease-out forwards; opacity: 0; }}
        .panel2 {{ animation: fadeIn 0.6s ease-out 0.2s forwards; opacity: 0; }}
        .panel3 {{ animation: fadeIn 0.6s ease-out 0.4s forwards; opacity: 0; }}
        .panel4 {{ animation: fadeIn 0.6s ease-out 0.6s forwards; opacity: 0; }}
        
        .cursor {{ animation: blink 1s step-end infinite; fill: #00f0ff; font-family: 'Share Tech Mono', monospace; font-size: 28px; font-weight: bold; }}
        
        .dot1 {{ animation: pulseDot 1.5s infinite; transform-origin: 840px 630px; }}
        .dot2 {{ animation: pulseDot 1.5s infinite 0.5s; transform-origin: 855px 630px; }}
        .dot3 {{ animation: pulseDot 1.5s infinite 1s; transform-origin: 870px 630px; fill: #00f0ff; filter: url(#glow); }}
        
        /* Hover Effects */
        .hover-box:hover {{ stroke: #00f0ff; transition: stroke 0.3s ease; cursor: pointer; }}
    </style>
    
    <!-- Background -->
    <rect width="900" height="700" class="bg" />
    <rect width="900" height="700" fill="url(#scanlines)" />
    
    <!-- Main Title -->
    <text x="40" y="50" class="title" style="animation: slideIn 0.5s ease-out forwards;">DHIO ANUGRAH. BACKEND DEVELOPER.</text>
    <text x="610" y="50" class="cursor">_</text>
    <text x="800" y="50" class="subtitle" style="font-size: 14px; fill: #1e3a8a;">v1.0.0</text>
    
    <!-- Grid Panels -->
    <!-- Panel 1: Profile -->
    <g class="panel1">
        <rect x="40" y="80" width="400" height="280" class="border hover-box" />
        <text x="55" y="110" class="subtitle">01 PROFILE CARDS</text>
        
        <!-- Profile Image -->
        <rect x="55" y="130" width="150" height="150" fill="#0f172a" stroke="#1e3a8a" stroke-width="1" />
        {image_svg}
        
        <!-- About Me Text -->
        <text x="225" y="145" class="text">Backend Developer with</text>
        <text x="225" y="165" class="text">experience in REST APIs,</text>
        <text x="225" y="185" class="text">warehouse management &amp;</text>
        <text x="225" y="205" class="text">enterprise apps.</text>
        <text x="225" y="235" class="text" style="font-size: 12px; fill: #3b82f6;">CURRENTLY:</text>
        <text x="225" y="250" class="text" style="font-size: 12px;">Backend @ KlinKlin Surabaya</text>
        <text x="225" y="265" class="text" style="font-size: 12px;">Informatics Student @ ITK</text>

        <!-- Web portfolio link -->
        <text x="55" y="325" class="highlight">> dhioanugrah.github.io</text>
    </g>
    
    <!-- Panel 2: Tech Stack -->
    <g class="panel2">
        <rect x="460" y="80" width="400" height="280" class="border hover-box" />
        <text x="475" y="110" class="subtitle">02 TECH STACK</text>
        
        <text x="475" y="150" class="highlight">BACKEND</text>
        <text x="475" y="170" class="text">Laravel, PHP, REST API</text>
        
        <text x="475" y="200" class="highlight">FRONTEND</text>
        <text x="475" y="220" class="text">Vue.js, Vite, Tailwind CSS</text>
        
        <text x="475" y="250" class="highlight">DATABASE &amp; TOOLS</text>
        <text x="475" y="270" class="text">MySQL, Git, Docker, Postman</text>
        
        <text x="475" y="300" class="highlight">AI &amp; DATA</text>
        <text x="475" y="320" class="text">Python, TensorFlow</text>
    </g>
    
    <!-- Panel 3: Experience -->
    <g class="panel3">
        <rect x="40" y="380" width="550" height="280" class="border hover-box" />
        <text x="55" y="410" class="subtitle">03 PROFESSIONAL EXPERIENCE</text>
        
        <text x="55" y="450" class="highlight">Fullstack Developer Intern @ PT KlinKlin</text>
        <text x="70" y="470" class="text">- Building Laravel REST APIs for Flutter &amp; Web</text>
        <text x="70" y="490" class="text">- Building ERP Website for KlinKlin</text>
        
        <text x="55" y="530" class="highlight">Project Manager &amp; Backend Developer</text>
        <text x="70" y="550" class="text">- WMS for PT Hidrolik Teknologi Indotama</text>
        
        <text x="55" y="590" class="highlight">Final Year Research</text>
        <text x="70" y="610" class="text">- Endemic Kalimantan Flora Classification</text>
        <text x="70" y="630" class="text">- Using EfficientNetV2 @ BPN Botanical Garden</text>
    </g>
    
    <!-- Panel 4: Connect -->
    <g class="panel4">
        <rect x="610" y="380" width="250" height="280" class="border hover-box" />
        <text x="625" y="410" class="subtitle">04 CONNECT</text>
        
        <rect x="625" y="450" width="220" height="40" rx="4" fill="#0f172a" stroke="#1e3a8a" />
        <text x="640" y="475" class="highlight">📧 EMAIL</text>
        <text x="640" y="495" class="text" style="font-size: 11px;">dhioanugrahh@gmail.com</text>
        
        <rect x="625" y="530" width="220" height="40" rx="4" fill="#0f172a" stroke="#1e3a8a" />
        <text x="640" y="555" class="highlight">🐙 GITHUB</text>
        <text x="640" y="575" class="text" style="font-size: 11px;">github.com/dhioanugrah</text>
    </g>

    <!-- Retro Dots -->
    <circle cx="840" cy="630" r="4" fill="#1e3a8a" class="dot1" />
    <circle cx="855" cy="630" r="4" fill="#1e3a8a" class="dot2" />
    <circle cx="870" cy="630" r="4" fill="#38bdf8" class="dot3" />
    
</svg>"""
    return svg

def create_readme(svg_filename):
    md = f"""# Dhio Anugrah - Backend Developer

Welcome to my GitHub profile!

<p align="center">
  <img src="{svg_filename}" alt="Dhio Anugrah Profile" />
</p>

<!-- 
This profile was generated using a custom SVG script to create a retro terminal theme. 
Update your profile by re-running the python script with your photo!
-->
"""
    return md

if __name__ == "__main__":
    image_path = None
    if len(sys.argv) > 1:
        image_path = sys.argv[1]
    
    print("Generating retro dark blue SVG profile...")
    svg_content = generate_svg(image_path)
    with open("profile.svg", "w") as f:
        f.write(svg_content)
    
    print("Generating README.md...")
    md_content = create_readme("profile.svg")
    with open("README.md", "w") as f:
        f.write(md_content)
        
    print("Done! Check profile.svg and README.md.")
    print("To use this on GitHub, push both files to your special profile repository.")
