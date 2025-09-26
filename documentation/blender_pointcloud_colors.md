# Visualizing Colored Point Clouds in Blender (3.6 / 4.x)

These instructions show how to display per-point colors from a `.ply` **Point Cloud object** in Blender.

---

## 1. Geometry Nodes Setup

1. Select your imported **Point Cloud object**.
2. Add a **Geometry Nodes modifier** → click **New**.
3. In the *Geometry Node Editor*, build this node tree:

**Nodes to add**
- Instance on Points
- Ico Sphere (Mesh Primitive)
- Realize Instances
- Set Material
- Group Input (already there)
- Group Output (already there)

**Connections**
```
Group Input.Geometry
  → Instance on Points.Points

Ico Sphere.Mesh
  → Instance on Points.Instance

Instance on Points.Instances
  → Realize Instances.Geometry

Realize Instances.Geometry
  → Set Material.Geometry

Set Material.Geometry
  → Group Output.Geometry
```

**Settings**
- **Ico Sphere → Subdivisions**: 1–2 (start with 1 for speed)
- **Ico Sphere → Radius**: `0.005–0.02` (adjust as needed)
- **Instance on Points → Pick Instance**: OFF

(Optional: If your `.ply` has a `"radius"` attribute, you can pipe it into *Instance on Points → Scale* using a **Named Attribute** node and **Capture Attribute** node.)

![A view of the Geometry Nodes for instancing primitives for each point in a point cloud.](image-1.png)

---

## 2. Material Setup

1. Create a **New Material** in the Shader Editor.
2. Keep the **Principled BSDF** node.
3. Add an **Attribute** node (`Shift+A → Input → Attribute`).

**Connections**
```
Attribute("Col").Color
  → Principled BSDF.Base Color

Principled BSDF.BSDF
  → Material Output.Surface
```

![A view of the Shading nodes for setting up material properties in Blender.](image.png)

**Notes**
- The attribute name is usually `"Col"`, but check **Object Data Properties → Attributes** for the exact name (case-sensitive).
- If colors look dim, add an **Emission** node and mix it with the Principled BSDF.

---

## 3. Viewport & Render Checklist

- Switch to **Material Preview** or **Rendered** mode.
- Ensure the **Geometry Nodes modifier** is enabled.
- Ensure the **Set Material** node uses your new material.
- If still gray:
  - Double-check attribute name (`Col`, `color`, etc.).
  - Test by plugging `Attribute → Color` directly into an **Emission → Color**.

---

## Minimal Node Summary

**Geometry Nodes**
```
Group Input → Instance on Points → Realize Instances → Set Material → Group Output
Ico Sphere → Instance on Points
```

**Material**
```
Attribute("Col") → Principled BSDF.Base Color → Material Output.Surface
```
