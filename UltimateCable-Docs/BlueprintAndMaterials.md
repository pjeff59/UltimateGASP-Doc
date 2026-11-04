# Guide Matériaux et Blueprint UltimateCable

Ce guide explique comment créer un matériau par défaut pour le câble (M_CableRope) et un Blueprint d’exemple pour tester rapidement le composant.

## 1) Créer le matériau par défaut M_CableRope

Chemin recommandé (pour l’auto-chargement par l’acteur de démo):
- /UltimateCable/Materials/M_CableRope

Étapes dans l’éditeur:
1. Créez un dossier `UltimateCable/Materials` sous le Content Browser de votre projet.
2. Créez un nouveau Matériau nommé `M_CableRope`.
3. Ouvrez-le et configurez:
   - Shading Model: Default Lit
   - Branches PBR simples:
     - Base Color: utilisez une Texture2D de corde ou un Constant3Vector sombre (0.15–0.25)
     - Roughness: ConstantScalar (0.6–0.9)
     - Normal: optionnel, légère Normal Map pour relief
   - UV: le maillage du câble fournit des UVs où U suit la longueur et V le tour; `UVTiling` côté composant multiplie déjà U. Vous pouvez donc brancher directement `TextureCoordinate` node (Channel 0) dans vos textures.
4. Sauvegardez.

Conseils:
- Pour varier la répétition le long du câble, ajustez `UVTiling` sur le composant (pas besoin d’exposer un paramètre côté matériau).
- Si vous voulez garder la main dans le matériau, vous pouvez multiplier `TextureCoordinate.U` par un `ScalarParameter TilingU` et le calibrer; sachez que le composant multiplie déjà U.

## 2) Utiliser le matériau par défaut automatiquement

L’acteur de démonstration fourni tente de charger `/UltimateCable/Materials/M_CableRope`. Si cet asset existe, il l’applique automatiquement lorsque `OverrideMaterial` est vide.
- Vous pouvez aussi appeler le nœud Blueprint `ApplyDefaultRopeMaterial` avec le composant câble.
- Vous pouvez assigner un matériau dans le slot du composant ou utiliser le nœud Blueprint `SetOverrideMaterial`.

## 3) Créer un Blueprint d’exemple

Objectif: un Actor Blueprint avec un `UltimateCableComponent` configurable.

Étapes:
1. Créez un Blueprint Actor: `BP_UltimateCable_Demo`.
2. Ajoutez un `UltimateCableComponent` (nom: `Cable`).
3. Détails du composant:
   - CableLength: 800
   - NumSegments: 32
   - CableRadius: 2
   - bEnableCollision: coché
   - CCDSubdivisions: 6–10 selon le besoin
   - bUseParallelSolver: coché (pour >16 segments)
   - bUseDistanceLOD: optionnel
4. Construction Script (ou BeginPlay):
   - `SetSimulationPreset(StableXPBD)`
   - `SetNumSegments(32, true)`
   - `SetCableLength(800.0, true)`
   - Optionnel: `PinEndsToComponents` pour attacher les extrémités au Root
   - Matériau: soit assignez `M_CableRope` au slot 0, soit appelez `SetOverrideMaterial(M_CableRope)`
5. Placez `BP_UltimateCable_Demo` dans la scène et lancez Play.

## 4) Astuces
- Pour des plans rapprochés, essayez `CinematicXPBD` avec `Substep` plus petit et plus d’itérations.
- Le coût dépend fortement de `NumSegments` et `SolverIterations`; ciblez 16–32 segments pour gameplay.
- Ajustez `SweepRadius` et `CCDSubdivisions` pour limiter le tunneling.

---

En cas de besoin, vous pouvez ajouter d’autres matériaux (acier, corde épaisse) et les appliquer via `OverrideMaterial` sans changer le code.
