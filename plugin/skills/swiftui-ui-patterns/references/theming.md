# Theming and dynamic type

## Intention

Fournis une approche de theming propre et scalable qui garde le code de view sémantique et cohérent.

## Patterns essentiels

- Utilise un seul objet `Theme` comme source de vérité (couleurs, fonts, spacing).
- Injecte le theme à la root de l'app et lis-le via `@Environment(Theme.self)` dans les views.
- Privilégie des couleurs sémantiques (`primaryBackground`, `secondaryBackground`, `label`, `tint`) plutôt que des couleurs brutes.
- Garde les controls de theme visibles par l'utilisateur dans un écran de settings dédié.
- Applique le scaling Dynamic Type via des fonts custom ou `.font(.scaled...)`.

## Exemple : objet Theme

```swift
@MainActor
@Observable
final class Theme {
  var tintColor: Color = .blue
  var primaryBackground: Color = .white
  var secondaryBackground: Color = .gray.opacity(0.1)
  var labelColor: Color = .primary
  var fontSizeScale: Double = 1.0
}
```

## Exemple : injection à la root de l'app

```swift
@main
struct MyApp: App {
  @State private var theme = Theme()

  var body: some Scene {
    WindowGroup {
      AppView()
        .environment(theme)
    }
  }
}
```

## Exemple : usage dans une view

```swift
struct ProfileView: View {
  @Environment(Theme.self) private var theme

  var body: some View {
    VStack {
      Text("Profile")
        .foregroundStyle(theme.labelColor)
    }
    .background(theme.primaryBackground)
  }
}
```

## Choix de design à garder

- Garde les valeurs de theme sémantiques et minimales ; évite de dupliquer les couleurs système.
- Stocke les valeurs de theme choisies par l'utilisateur dans un stockage persistant si nécessaire.
- Assure le contraste entre le texte et les backgrounds.

## Pièges

- Évite de disperser des valeurs `Color` brutes dans les views ; ça casse la cohérence.
- Ne lie pas le theme au state local d'une seule view.
- Évite d'utiliser `@Environment(\\.colorScheme)` comme seul control de theme ; il doit compléter ton theme.
