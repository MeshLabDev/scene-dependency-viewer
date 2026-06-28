# Scene Dependency Viewer

**Visualize project dependencies, find broken references, and detect unused assets in Godot 4.x.**

A lightweight editor addon that scans your project files and shows you exactly what depends on what — before something breaks.

---

## Why This Exists

You delete a texture. Three scenes break. You don't find out until runtime.

Scene Dependency Viewer scans your entire project and shows you:
- **What a file depends on** — see all external references in any scene, resource, or script
- **What depends on a file** — before you move or delete anything, know what will break
- **Broken references** — find missing files instantly
- **Unused assets** — clean up files nothing actually uses

---

## 60-Second Quickstart

1. Copy `addons/scene_dependency_viewer/` into your project.
2. Open **Project → Project Settings → Plugins** and enable **Scene Dependency Viewer**.
3. A new **Dependencies** tab appears at the bottom of the editor.
4. Click **Scan Project**. Your entire dependency tree appears.

That's it.

---

## Features

### Dependency Tree
- Browse all project files grouped by type (scenes, scripts, textures, audio, models)
- Search and filter to find specific files fast
- See dependency count per file at a glance

### Reverse Dependencies
- Click any file to see what depends on it
- Know exactly what breaks if you move or delete a file
- Never guess whether a resource is safe to remove

### Broken Reference Detection
- Instantly find files referencing missing resources
- See exactly which file has the broken reference and what it's looking for
- Fix issues before they cause runtime errors

### Unused Asset Detection
- Find files that nothing in your project actually references
- Clean up bloat and reduce project size
- Safely remove dead assets

### Supported File Types
- **Scenes** (`.tscn`) — all ext_resource and sub_resource references
- **Resources** (`.tres`) — embedded references
- **Scripts** (`.gd`, `.cs`) — `load()`, `preload()`, and `class_name` references
- **Import files** (`.import`) — source file tracking
- **Config files** (`.cfg`) — `res://` references
- **Textures, Audio, Models, Fonts** — tracked as leaf dependencies

---

## API Reference

### DependencyScanner

The core scanning engine. Can be used standalone or via the editor panel.

```gdscript
const DependencyScanner = preload("res://addons/scene_dependency_viewer/dependency_scanner.gd")

var scanner = DependencyScanner.new()

# Connect signals
scanner.scan_progress.connect(func(msg, current, total):
    print("%s (%d/%d)" % [msg, current, total])
)

scanner.scan_completed.connect(func(result):
    print("Found %d files" % result.files.size())
    print("Broken refs: %d" % result.broken_refs.size())
    print("Unused: %d" % result.unused_assets.size())
)

# Run scan
var result = await scanner.scan_project()
```

### Result Structure

```gdscript
{
    "files": {
        "res://path/to/file.tscn": {
            "type": "scene",          # scene|resource|script|texture|audio|model|font|config|other
            "deps": ["res://..."],    # Array of dependency paths
            "uid": "uid://..."        # UID if available
        }
    },
    "reverse_deps": {
        "res://path/to/texture.png": ["res://scene1.tscn", "res://scene2.tscn"]
    },
    "broken_refs": [
        {
            "file": "res://scene.tscn",
            "missing_ref": "res://deleted_texture.png",
            "line": 0
        }
    ],
    "unused_assets": [
        "res://old_sprite.png"
    ]
}
```

### Filtering by Type

```gdscript
# Get only scenes
var scenes = result.files.keys().filter(func(f): return result.files[f].type == "scene")

# Get only broken references in scripts
var script_broken = result.broken_refs.filter(func(r): return r.file.ends_with(".gd"))
```

---

## Integration Steps

### Basic
1. Enable the plugin
2. Click "Scan Project" in the Dependencies tab
3. Browse the tree, click files to see their dependencies

### CI/Headless
Use `DependencyScanner` directly in a headless build or test script:

```gdscript
func test_dependencies():
    var scanner = DependencyScanner.new()
    var result = await scanner.scan_project()
    assert(result.broken_refs.size() == 0, "Found broken references!")
    assert(result.unused_assets.size() < 10, "Too many unused assets")
```

---

## FAQ

**Q: Does this slow down the editor?**
A: No. Scanning is manual (click Scan) and takes 1-3 seconds for most projects. Results are cached until you rescan.

**Q: Does it work with .import files?**
A: Yes. It parses `.import` files to track the relationship between imported assets and their source files.

**Q: Can I use this in CI/automation?**
A: Yes. `DependencyScanner` is a standalone class that works headless.

**Q: Does it detect `class_name` usage?**
A: Yes. Scripts with `class_name` are marked as potentially used (since they can be referenced by type anywhere).

**Q: What about `uid://` references?**
A: Tracked. The scanner resolves UIDs where possible and includes them in the dependency graph.

---

## File Structure

```
addons/scene_dependency_viewer/
├── plugin.cfg
├── plugin.gd
├── dependency_scanner.gd
├── dependency_viewer_panel.gd
├── dependency_viewer_panel.tscn
└── LICENSE
```

---

## License

MIT — use in personal and commercial projects.

## AI Use

This asset was created by MeshLabDev with the help of an AI coding assistant (Anthropic
Claude). The product design and feature set were human-directed, and all code and
documentation were reviewed, tested, and debugged by the developer before release. No
AI-generated art, audio, 3D models, or other media are included; all screenshots and demo
media are genuine in-engine captures.
