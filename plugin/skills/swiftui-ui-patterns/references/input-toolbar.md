# Input toolbar (ancrée en bas)

## Intention

Utilise une barre d'input ancrée en bas pour le chat, le composer, ou des actions rapides sans te battre avec le clavier.

## Patterns essentiels

- Utilise `.safeAreaInset(edge: .bottom)` pour ancrer la toolbar au-dessus du clavier.
- Garde le contenu principal dans un `ScrollView` ou `List`.
- Pilote le focus avec `@FocusState` et fixe le focus initial si besoin.
- Évite d'intégrer la barre d'input dans le contenu du scroll ; garde-la séparée.

## Exemple : scroll view + input en bas

```swift
@MainActor
struct ConversationView: View {
  @FocusState private var isInputFocused: Bool

  var body: some View {
    ScrollViewReader { _ in
      ScrollView {
        LazyVStack {
          ForEach(messages) { message in
            MessageRow(message: message)
          }
        }
        .padding(.horizontal, .layoutPadding)
      }
      .safeAreaInset(edge: .bottom) {
        InputBar(text: $draft)
          .focused($isInputFocused)
      }
      .scrollDismissesKeyboard(.interactively)
      .onAppear { isInputFocused = true }
    }
  }
}
```

## Choix de design à garder

- Garde la barre d'input visuellement séparée du contenu scrollable.
- Utilise `.scrollDismissesKeyboard(.interactively)` pour les écrans façon chat.
- Assure-toi que les actions d'envoi sont atteignables via le return du clavier ou un bouton clair.

## Pièges

- Évite de placer la view d'input dans le stack de scroll ; elle sautera avec le contenu.
- Évite les scroll views imbriquées qui se disputent les gestures de drag.
