# Controls (Toggle, Slider, Picker)

## Intention

Utilise les controls natifs pour les écrans de settings et de configuration, en gardant des labels accessibles et des bindings de state clairs.

## Patterns essentiels

- Bind les controls directement à `@State`, `@Binding`, ou `@AppStorage`.
- Privilégie `Toggle` pour les préférences booléennes.
- Utilise `Slider` pour les plages numériques et affiche la valeur actuelle dans un label.
- Utilise `Picker` pour les choix discrets ; réserve `.pickerStyle(.segmented)` aux cas de 2 à 4 options.
- Garde les labels visibles et descriptifs ; évite d'embarquer des buttons dans les controls.

## Exemple : toggles avec sections

```swift
Form {
  Section("Notifications") {
    Toggle("Mentions", isOn: $preferences.notificationsMentionsEnabled)
    Toggle("Follows", isOn: $preferences.notificationsFollowsEnabled)
    Toggle("Boosts", isOn: $preferences.notificationsBoostsEnabled)
  }
}
```

## Exemple : slider avec texte de valeur

```swift
Section("Font Size") {
  Slider(value: $fontSizeScale, in: 0.5...1.5, step: 0.1)
  Text("Scale: \(String(format: \"%.1f\", fontSizeScale))")
    .font(.scaledBody)
}
```

## Exemple : picker pour un enum

```swift
Picker("Default Visibility", selection: $visibility) {
  ForEach(Visibility.allCases, id: \.self) { option in
    Text(option.title).tag(option)
  }
}
```

## Choix de design à garder

- Groupe les controls liés dans une section de `Form`.
- Utilise `.disabled(...)` pour refléter les settings verrouillés ou hérités.
- Utilise `Label` dans les toggles pour combiner icon + texte quand ça apporte de la clarté.

## Pièges

- Évite `.pickerStyle(.segmented)` pour de grands ensembles ; utilise plutôt les styles menu ou inline.
- Ne cache pas les labels des sliders ; montre toujours le contexte.
- Évite de hardcoder les couleurs des controls ; utilise le tint du theme avec parcimonie.
