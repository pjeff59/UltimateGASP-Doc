# Ultimate Dismemberment — Visual Fixes and Stump Cap

This guide covers automatic visual-stability fixes and the optional **Stump Cap Mesh** setting. All setup is performed in Blueprint and Details panels.

## Automatic fixes

The plugin applies the following fixes without requiring changes to existing Blueprints:

- Physics bodies below a severed bone are stopped to help prevent them from intersecting the stump.
- Child-bone collection accounts for the skeleton subtree and explicitly excluded bones.
- An active physics constraint at the cut is handled during severance.
- Registered visual meshes can be refreshed after a mesh change.

These fixes do not require a new asset. The final appearance still depends on the character's skeleton, Physics Asset, materials, and meshes.

## Optional feature: Stump Cap Mesh

A Stump Cap is a Static Mesh spawned at the severance point to cover a visible seam or small opening. Configure it separately for each entry in **Dismemberment Zones**. Leave **Stump Cap Mesh (optional, hides seam)** empty to disable it.

### Setup

1. Create or choose a Static Mesh that fits the stump shape.
2. Open the character Blueprint and select **Ultimate Dismemberment Component**.
3. Expand the relevant zone under **Dismemberment Zones**.
4. Expand **Limb | Stump Cap** and assign **Stump Cap Mesh (optional, hides seam)**.
5. Adjust **Stump Cap Relative Offset**, **Stump Cap Relative Rotation**, and **Stump Cap Scale**.
6. Play the game and inspect the severance from several angles.

The cap attaches to **Stump Bone** when set, otherwise to **Root Bone**. Start with zero offset and rotation, then adjust them to match the orientation of your mesh.

### Verification and troubleshooting

- If the cap is missing, verify the assigned asset, **Stump Cap Mesh**, and attachment bone name.
- If it is offset, adjust the relative offset in the attachment point's local space.
- If it faces the wrong direction, adjust the relative rotation.
- If it is too large or small, adjust **Stump Cap Scale**.
- If the seam remains visible, check cap placement and the mesh skin weights around the cut.

The Stump Cap is optional. The plugin remains usable when this property is empty.
