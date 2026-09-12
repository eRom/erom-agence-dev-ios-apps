# Gestion du focus et chaînage de champs

## Intention

Utilise `@FocusState` pour contrôler le focus clavier, chaîner les champs, et coordonner le focus dans des forms complexes.

## Patterns essentiels

- Utilise un enum pour représenter les champs focusables.
- Fixe le focus initial dans `onAppear`.
- Utilise `.onSubmit` pour déplacer le focus vers le champ suivant.
- Pour des listes dynamiques de champs, utilise un enum avec des valeurs associées (par ex. `.option(Int)`).

## Exemple : focus sur un seul champ

```swift
struct AddServerView: View {
  @State private var server = ""
  @FocusState private var isServerFieldFocused: Bool

  var body: some View {
    Form {
      TextField("Server", text: $server)
        .focused($isServerFieldFocused)
    }
    .onAppear { isServerFieldFocused = true }
  }
}
```

## Exemple : focus chaîné avec enum

```swift
struct EditTagView: View {
  enum FocusField { case title, symbol, newTag }
  @FocusState private var focusedField: FocusField?

  var body: some View {
    Form {
      TextField("Title", text: $title)
        .focused($focusedField, equals: .title)
        .onSubmit { focusedField = .symbol }

      TextField("Symbol", text: $symbol)
        .focused($focusedField, equals: .symbol)
        .onSubmit { focusedField = .newTag }
    }
    .onAppear { focusedField = .title }
  }
}
```

## Exemple : focus dynamique pour des champs variables

```swift
struct PollView: View {
  enum FocusField: Hashable { case option(Int) }
  @FocusState private var focused: FocusField?
  @State private var options: [String] = ["", ""]
  @State private var currentIndex = 0

  var body: some View {
    ForEach(options.indices, id: \.self) { index in
      TextField("Option \(index + 1)", text: $options[index])
        .focused($focused, equals: .option(index))
        .onSubmit { addOption(at: index) }
    }
    .onAppear { focused = .option(0) }
  }

  private func addOption(at index: Int) {
    options.append("")
    currentIndex = index + 1
    DispatchQueue.main.asyncAfter(deadline: .now() + 0.01) {
      focused = .option(currentIndex)
    }
  }
}
```

## Choix de design à garder

- Garde le state de focus local à la view qui possède les champs.
- Utilise les changements de focus pour piloter l'UX (messages de validation, UI d'aide).
- Associe avec `.scrollDismissesKeyboard(...)` quand tu utilises ScrollView/Form.

## Pièges

- Ne stocke pas le state de focus dans des objets partagés ; il est local à la view.
- Évite les changements de focus agressifs pendant une animation ; retarde-les si besoin.
