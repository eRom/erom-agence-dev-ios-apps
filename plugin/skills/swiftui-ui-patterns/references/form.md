# Form

## Intention

Utilise `Form` pour les settings structurés, les inputs groupés, et les rows d'action. Ce pattern garde le layout, l'espacement, et l'accessibilité cohérents pour les écrans de saisie de data.

## Patterns essentiels

- Enveloppe le form dans un `NavigationStack` uniquement quand il est présenté dans une sheet ou en view autonome sans contexte de navigation existant.
- Groupe les controls liés dans des blocs `Section`.
- Utilise `.scrollContentBackground(.hidden)` plus une couleur de background custom quand tu as besoin des couleurs du design system.
- Applique `.formStyle(.grouped)` pour un style groupé quand c'est pertinent.
- Utilise `@FocusState` pour gérer le focus clavier dans les forms riches en input.

## Exemple : form façon settings

```swift
@MainActor
struct SettingsView: View {
  @Environment(Theme.self) private var theme

  var body: some View {
    NavigationStack {
      Form {
        Section("General") {
          NavigationLink("Display") { DisplaySettingsView() }
          NavigationLink("Haptics") { HapticsSettingsView() }
        }

        Section("Account") {
          Button("Edit profile") { /* open sheet */ }
            .buttonStyle(.plain)
        }
        .listRowBackground(theme.primaryBackgroundColor)
      }
      .navigationTitle("Settings")
      .navigationBarTitleDisplayMode(.inline)
      .scrollContentBackground(.hidden)
      .background(theme.secondaryBackgroundColor)
    }
  }
}
```

## Exemple : form modal avec validation

```swift
@MainActor
struct AddRemoteServerView: View {
  @Environment(\.dismiss) private var dismiss
  @Environment(Theme.self) private var theme

  @State private var server: String = ""
  @State private var isValid = false
  @FocusState private var isServerFieldFocused: Bool

  var body: some View {
    NavigationStack {
      Form {
        TextField("Server URL", text: $server)
          .keyboardType(.URL)
          .textInputAutocapitalization(.never)
          .autocorrectionDisabled()
          .focused($isServerFieldFocused)
          .listRowBackground(theme.primaryBackgroundColor)

        Button("Add") {
          guard isValid else { return }
          dismiss()
        }
        .disabled(!isValid)
        .listRowBackground(theme.primaryBackgroundColor)
      }
      .formStyle(.grouped)
      .navigationTitle("Add Server")
      .navigationBarTitleDisplayMode(.inline)
      .scrollContentBackground(.hidden)
      .background(theme.secondaryBackgroundColor)
      .scrollDismissesKeyboard(.immediately)
      .toolbar { CancelToolbarItem() }
      .onAppear { isServerFieldFocused = true }
    }
  }
}
```

## Choix de design à garder

- Privilégie `Form` aux stacks custom pour les settings et les écrans de saisie.
- Garde les rows tappables en utilisant `.contentShape(Rectangle())` et `.buttonStyle(.plain)` sur les boutons de row.
- Utilise les backgrounds de row de liste pour garder le style de section cohérent avec ton theme.

## Pièges

- Évite les layouts custom lourds à l'intérieur d'un `Form` ; cela peut créer des problèmes d'espacement.
- Si tu as besoin de layouts très custom, privilégie `ScrollView` + `VStack`.
- Ne mélange pas plusieurs stratégies de background ; choisis soit le style par défaut de Form, soit des couleurs custom.
