---
name: plymouth-script-theme
description: API reference and guidelines for authoring smooth Plymouth script-module boot themes.
---

# Plymouth Script Theme Authoring Guide

This skill provides an authoritative reference for writing custom Plymouth themes using Plymouth's `script` module (`ModuleName=script`).

## Directory & File Structure
```text
/usr/share/plymouth/themes/<theme-name>/
├── <theme-name>.plymouth   # Theme metadata
├── <theme-name>.script     # Animation script
├── logo.png                # Transparent PNG logo
└── dot.png                 # Transparent PNG dot/frame
```

### `<theme-name>.plymouth` Example
```ini
[Plymouth Theme]
Name=Arch Smooth
Description=Smooth Plymouth theme using script module
ModuleName=script

[script]
ImageDir=/usr/share/plymouth/themes/arch-smooth
ScriptFile=/usr/share/plymouth/themes/arch-smooth/arch-smooth.script
```

---

## Plymouth Script API Reference

### Image Operations
```
// Load image from ImageDir (MUST be a valid PNG file!)
img = Image("logo.png");

// Scale image (returns a new scaled Image object)
img_scaled = img.Scale(new_width, new_height);

// Get image dimensions
w = img.GetWidth();
h = img.GetHeight();
```

### Sprite Operations
```
// Create a sprite containing an image
s = Sprite(img_scaled);

// Set position & layer (Z-index)
s.SetX(x_pos);
s.SetY(y_pos);
s.SetZ(z_layer);

// Set opacity (0.0 = invisible, 1.0 = opaque)
s.SetOpacity(alpha_val);

// Update sprite image
s.SetImage(new_img);
```

### Screen & Math API
```
// Screen bounds
screen_w = Plymouth.GetWidth();
screen_h = Plymouth.GetHeight();

// Math utilities
Math.Sin(radians);
Math.Cos(radians);
Math.Pi;
Math.Pow(base, exponent);
Math.Sqrt(value);
Math.Int(float_value);
```

### Refresh Loop & Message Handlers
```
// Refresh callback runs at Plymouth's frame rate (~30-60 FPS)
fun refresh_callback() {
    // Update sprite positions, rotations, opacities
}
Plymouth.SetRefreshFunction(refresh_callback);

// Suppress on-screen status text messages
fun display_normal_callback() {}
fun message_callback(text) {}
Plymouth.SetDisplayNormalFunction(display_normal_callback);
Plymouth.SetMessageFunction(message_callback);
```

---

## Critical Rules & Prohibited Constructs

1. **NO Non-Existent API Calls**:
   - ❌ `Image(width, height)` — Does NOT exist in Plymouth script engine.
   - ❌ `Image.CreateFromFile(...)` — Does NOT exist.
   - ❌ `image.SetPixel(...)` — Does NOT exist.
   - ❌ `Window.SetBackgroundTopColor(...)` — Does NOT exist.
   Using invalid API calls will silently crash Plymouth on boot, causing Linux to drop back to text logs!

2. **Image Format Requirement**:
   - Assets MUST be genuine PNG files (e.g. RGBA 8-bit non-interlaced PNG).
   - Copying a JPEG or WebP file and renaming it to `.png` will cause Plymouth's image decoder to fail and render fallback text squares (`■`).

3. **Initramfs Hook Order**:
   - In `/etc/mkinitcpio.conf`, `kms` MUST be placed BEFORE `plymouth`:
     `HOOKS=(... kms plymouth ...)`
   - If `plymouth` runs before `kms`, Plymouth cannot acquire the GPU DRM framebuffer early enough, causing boot text to leak onto the screen.

4. **Live Preview Limitation**:
   - Plymouth splash themes CANNOT be previewed live inside a running Wayland or X11 compositor (e.g. Hyprland), because the display compositor owns the DRM GPU device.
   - Preview must be done by rebooting the system.
