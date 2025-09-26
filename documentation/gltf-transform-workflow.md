# glTF-Transform Workflow Notes

## Prerequisites

1. **Install Node.js (one-time):**
   Using Homebrew on macOS (if already installed):
   ```bash
   brew install node
   ```
   or use Volta:
   ```bash
   curl https://get.volta.sh | bash
   volta install node@20
   ```

2. **No global install required.** We call the CLI via `npx`, which downloads the exact version on demand:
   ```bash
   npx @gltf-transform/cli --help
   ```

## Single-Command Optimization

To run the most common optimizations in one step, use the `optimize` command:

```bash
gltf-transform optimize input.glb output.glb --compress draco --texture-compress webp
```

Reference: https://gltf-transform.dev/cli

Use this to simplify geometry, Draco-compress, resize textures, and convert to WebP in one go:

```bash
npx @gltf-transform/cli optimize \
  model_1.glb \
  model_1.opt.glb \
  --compress draco \
  --texture-compress webp \
  --texture-size 2048 \
  --simplify-ratio 0.6 \
  --simplify-error 0.001
```

- `--compress draco` → adds `KHR_draco_mesh_compression`.
- `--texture-compress webp` + `--texture-size 2048` → downscale to 2048 px, emit WebP textures (`EXT_texture_webp`).
- `--simplify-ratio` / `--simplify-error` → lower triangle count while keeping shape close to original.

## Optional Step-by-Step Workflow

If you want explicit control over each transform:

```bash
# Geometry reduction first
npx @gltf-transform/cli simplify \
  model_1.glb \
  model_1.simplified.glb \
  --ratio 0.6 \
  --error 0.001

# Draco, texture compression, pruning, etc.
npx @gltf-transform/cli optimize \
  model_1.simplified.glb \
  model_1.opt.glb \
  --compress draco \
  --texture-compress webp \
  --texture-size 2048
```

## Batch Command (zsh)

Optimize all relevant models at once:

This example manually lists the models you want to process:

```bash
for f in AR/Assets/{model_1,model_2,model_3,model_4}.glb; do
  out="${f%.glb}.opt.glb"
  echo "Optimizing $f -> $out"
  npx @gltf-transform/cli optimize \
    "$f" "$out" \
    --compress draco \
    --texture-compress webp \
    --texture-size 2048 \
    --simplify-ratio 0.6 \
    --simplify-error 0.001
done
```

To target every `.glb` file in the folder automatically:

```bash
for f in AR/Assets/*.glb; do
  out="${f%.glb}.opt.glb"
  echo "Optimizing $f -> $out"
  npx @gltf-transform/cli optimize \
    "$f" "$out" \
    --compress draco \
    --texture-compress webp \
    --texture-size 2048 \
    --simplify-ratio 0.6 \
    --simplify-error 0.001
done
```

## Verification

After each run:

```bash
# Check sizes
ls -lh AR/Assets/model_1*.glb

# Inspect required extensions on a single file
npx @gltf-transform/cli inspect AR/Assets/model_1.opt.glb \
  | rg 'extensionsRequired'

# Inspect every glb in the folder
for f in AR/Assets/*.glb; do
  echo "Inspecting $f"
  npx @gltf-transform/cli inspect "$f" | rg 'extensionsRequired'
done
```

You should see `KHR_draco_mesh_compression` and `EXT_texture_webp`. Modern Safari/Chrome support both; if any target device doesn’t support WebP, rerun with `--texture-compress jpg`.

---

## Runtime Notes

- ARx.html already references `.opt.glb` via `data-src` with fallback to the original `.glb`, so it is enough to drop the optimized `.opt.glb` files in `AR/Assets`.
- Because the pipeline uses Draco and WebP, no extra JavaScript wiring is necessary—the GLTFLoader bundled with A-Frame 1.5.0 recognizes Draco automatically when using the official decoders.