"""Generates neofetch-style README card (profile.svg) with ASCII art of your photo.
Usage: python generate.py photo.png   (edit INFO below to change text)"""
import sys, html
from PIL import Image, ImageOps, ImageFilter, ImageDraw

PHOTO = sys.argv[1] if len(sys.argv) > 1 else "photo.png"

# ---------- ASCII art ----------
im = Image.open(PHOTO).convert("L")
w, h = im.size
face = im.crop((int(w*.21), int(h*.09), int(w*.76), int(h*.78)))   # adjust crop if needed
face = ImageOps.equalize(face).filter(ImageFilter.UnsharpMask(radius=3, percent=200))
W = 56
H = int(W * face.size[1] / face.size[0] * 0.5)
small = face.resize((W, H), Image.LANCZOS)
mask = Image.new("L", (W, H), 0)
ImageDraw.Draw(mask).ellipse((1, 0, W-2, H-1), fill=255)
ramp = " .:-=+*#%@"
art = []
for y in range(H):
    row = ""
    for x in range(W):
        if mask.getpixel((x, y)) == 0:
            row += " "; continue
        v = 255 - small.getpixel((x, y))
        row += ramp[min(len(ramp)-1, int(v/256*len(ramp)))]
    art.append(row.rstrip())

# ---------- Info (edit me) ----------
# ("header", text) | ("kv", key, value) | ("blank",)
INFO = [
 ("header", "asad@AsadUllah077"),
 ("kv", "OS", "Windows 11, Linux"),
 ("kv", "Uptime", "X years, Y months"),
 ("kv", "Host", "Your Company / Freelance"),
 ("kv", "Kernel", "Full Stack Developer"),
 ("kv", "IDE", "VS Code"),
 ("kv", "Location", "Pakistan 🇵🇰"),
 ("blank",),
 ("kv", "Languages.Programming", "PHP, JavaScript, Go"),
 ("kv", "Frameworks", "Laravel, Vue, Node.js"),
 ("kv", "Languages.Computer", "HTML, CSS, SASS, SQL"),
 ("kv", "Databases", "MySQL, MongoDB, PostgreSQL, Redis"),
 ("kv", "Languages.Real", "English, Urdu"),
 ("blank",),
 ("kv", "Working.On", "Bank Project"),
 ("kv", "Looking.For", "ERP collaborations"),
 ("kv", "Hobbies.Software", "Creating YouTube videos"),
 ("blank",),
 ("header", "Contact"),
 ("kv", "Email.Personal", "asad.developer069@gmail.com"),
 ("kv", "LinkedIn", "asad-ullah-97521b260"),
 ("kv", "GitHub", "AsadUllah077"),
 ("blank",),
 ("header", "GitHub Stats"),
 ("kv", "Repos", "00 {Contributed: 00} | Stars: 00"),
 ("kv", "Commits", "0,000 | Followers: 00"),
 ("kv", "Lines of Code on GitHub", "000,000 ( 000,000++, 00,000-- )"),
]

# ---------- Render SVG ----------
FS, LH, CW = 13, 16, 7.8          # font size, line height, char width
ART_X, INFO_X, TOP = 24, 480, 36
INFO_W = 78                         # characters per info line
rows = max(len(art), len(INFO))
width = INFO_X + INFO_W * CW + 24
height = TOP + rows * LH + 24
esc = html.escape
out = [f'<svg xmlns="http://www.w3.org/2000/svg" width="{width:.0f}" height="{height}" viewBox="0 0 {width:.0f} {height}">',
 '<style>text{font-family:"DejaVu Sans Mono","Courier New",monospace;font-size:%dpx;white-space:pre}'
 '.a{fill:#c9d1d9}.h{fill:#c9d1d9}.k{fill:#f0883e}.d{fill:#6e7681}.v{fill:#58a6ff}</style>' % FS,
 f'<rect width="100%" height="100%" rx="14" fill="#0d1117"/>']
for i, line in enumerate(art):
    out.append(f'<text class="a" x="{ART_X}" y="{TOP+i*LH}" xml:space="preserve">{esc(line)}</text>')
for i, item in enumerate(INFO):
    y = TOP + i*LH
    if item[0] == "header":
        t = item[1]
        out.append(f'<text class="h" x="{INFO_X}" y="{y}" xml:space="preserve">{esc(t)} <tspan class="d">{"—"*(INFO_W-len(t)-1)}</tspan></text>')
    elif item[0] == "kv":
        k, v = item[1], item[2]
        dots = "." * max(2, INFO_W - len(k) - len(v) - 4)
        out.append(f'<text x="{INFO_X}" y="{y}" xml:space="preserve"><tspan class="d">. </tspan><tspan class="k">{esc(k)}:</tspan> <tspan class="d">{dots}</tspan> <tspan class="v">{esc(v)}</tspan></text>')
out.append("</svg>")
open("profile.svg", "w", encoding="utf-8").write("\n".join(out))
print("wrote profile.svg")
