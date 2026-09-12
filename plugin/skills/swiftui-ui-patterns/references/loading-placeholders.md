# Loading & Placeholders

Utilise ceci quand une view a besoin d'un état de loading cohérent (skeletons, redaction, empty state) sans bloquer l'interaction.

## Patterns à privilégier

- **Placeholders redacted** pour le contenu de list/detail, afin de préserver le layout pendant le chargement.
- **ContentUnavailableView** pour les états vides ou d'erreur une fois le chargement terminé.
- **ProgressView** uniquement pour des opérations courtes et globales (à utiliser avec parcimonie sur les écrans riches en contenu).

## Approche recommandée

1. Garde le layout réel, rends des placeholder data, puis applique `.redacted(reason: .placeholder)`.
2. Pour les listes, affiche un nombre fixe de rows de placeholder (évite les spinners infinis).
3. Passe à `ContentUnavailableView` quand le chargement se termine mais que les data sont vides.

## Pièges

- N'anime pas les décalages de layout pendant la redaction ; garde les frames stables.
- Évite d'imbriquer plusieurs spinners ; utilise un seul indicateur de loading par section.
- Garde le nombre de placeholders réduit (3 à 6) pour limiter le jank sur les appareils bas de gamme.

## Usage minimal

```swift
VStack {
  if isLoading {
    ForEach(0..<3, id: \.self) { _ in
      RowView(model: .placeholder())
    }
    .redacted(reason: .placeholder)
  } else if items.isEmpty {
    ContentUnavailableView("No items", systemImage: "tray")
  } else {
    ForEach(items) { item in RowView(model: item) }
  }
}
```
