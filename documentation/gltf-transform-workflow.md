# glTF-Transform Workflow Notes

## Prerequisites

1. **Install Node.js (one-time):**
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

Use this to simplify geometry, Draco-compress, resize textures, and convert to WebP in one go:

```bash
npx @gltf-transform/cli optimize \
  CubeSat_-_1_RU_Generic.glb \
  CubeSat_-_1_RU_Generic.opt.glb \
  --compress draco \
  --texture-compress webp \
  --texture-size 2048 \
  --simplify-ratio 0.6 \
  --simplify-error 0.001
```

- `--compress draco` → adds `KHR_draco_mesh_compression`.
- `--texture-compress webp` + `--texture-size 2048` → downscale to 2048 px, emit WebP textures (`EXT_texture_webp`).
- `--simplify-ratio` / `--simplify-error` → lower triangle count while keeping shape close to original.

> Note: `--simplify-lock-normals` isn’t available; the CLI ignores it.

## Optional Step-by-Step Workflow

If you want explicit control over each transform:

```bash
# Geometry reduction first
npx @gltf-transform/cli simplify \
  CubeSat_-_1_RU_Generic.glb \
  CubeSat_-_1_RU_Generic.simplified.glb \
  --ratio 0.6 \
  --error 0.001

# Draco, texture compression, pruning, etc.
npx @gltf-transform/cli optimize \
  CubeSat_-_1_RU_Generic.simplified.glb \
  CubeSat_-_1_RU_Generic.opt.glb \
  --compress draco \
  --texture-compress webp \
  --texture-size 2048
```

## Batch Command (zsh)

Optimize all relevant models at once:

```bash
for f in AR/Assets/{Antarctica3,earths_interior,tohoku,CubeSat_-_1_RU_Generic}.glb; do
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
ls -lh AR/Assets/CubeSat_-_1_RU_Generic*.glb

# Inspect required extensions
npx @gltf-transform/cli inspect AR/Assets/CubeSat_-_1_RU_Generic.opt.glb \
  | rg 'extensionsRequired'
```

You should see `KHR_draco_mesh_compression` and `EXT_texture_webp`. Modern Safari/Chrome support both; if any target device doesn’t support WebP, rerun with `--texture-compress jpg`.

## Runtime Notes

- Your HTML already references `.opt.glb` via `data-src` with fallback to the original `.glb`, so dropping the optimized files in `AR/Assets` is enough.
- Because the pipeline uses Draco and WebP, no extra JavaScript wiring was necessary—the GLTFLoader bundled with A-Frame 1.5.0 recognizes Draco automatically when using the official decoders.

---

Keep this Markdown file alongside the project for quick reference the next time you need to regenerate optimized assets.
