# Haptics

## Intention

Utilise les haptics avec parcimonie pour renforcer les actions utilisateur (sélection de tab, refresh, succès/erreur) et respecte les préférences de l'utilisateur.

## Patterns essentiels

- Centralise les triggers haptiques dans un `HapticManager` ou un utilitaire équivalent.
- Verrouille les haptics derrière les préférences utilisateur et le support matériel.
- Utilise des types distincts pour différents moments UX (selection vs. notification vs. refresh).

## Exemple : haptic manager simple

```swift
@MainActor
final class HapticManager {
  static let shared = HapticManager()

  enum HapticType {
    case buttonPress
    case tabSelection
    case dataRefresh(intensity: CGFloat)
    case notification(UINotificationFeedbackGenerator.FeedbackType)
  }

  private let selectionGenerator = UISelectionFeedbackGenerator()
  private let impactGenerator = UIImpactFeedbackGenerator(style: .heavy)
  private let notificationGenerator = UINotificationFeedbackGenerator()

  private init() { selectionGenerator.prepare() }

  func fire(_ type: HapticType, isEnabled: Bool) {
    guard isEnabled else { return }
    switch type {
    case .buttonPress:
      impactGenerator.impactOccurred()
    case .tabSelection:
      selectionGenerator.selectionChanged()
    case let .dataRefresh(intensity):
      impactGenerator.impactOccurred(intensity: intensity)
    case let .notification(style):
      notificationGenerator.notificationOccurred(style)
    }
  }
}
```

## Exemple : usage

```swift
Button("Save") {
  HapticManager.shared.fire(.notification(.success), isEnabled: preferences.hapticsEnabled)
}

TabView(selection: $selectedTab) { /* tabs */ }
  .onChange(of: selectedTab) { _, _ in
    HapticManager.shared.fire(.tabSelection, isEnabled: preferences.hapticTabSelectionEnabled)
  }
```

## Choix de design à garder

- Les haptics doivent être subtiles et ne pas se déclencher à chaque micro-interaction.
- Respecte les préférences utilisateur (toggle pour désactiver).
- Garde les triggers haptiques proches de l'action utilisateur, pas enfouis dans la couche data.

## Pièges

- Évite de déclencher plusieurs haptics en succession rapide.
- Ne présume pas que les haptics sont disponibles ; vérifie le support.
