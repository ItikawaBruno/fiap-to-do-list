# To-Do List — Android (Kotlin + Jetpack Compose)

Aplicativo de lista de tarefas desenvolvido em Android com Kotlin. O objetivo é permitir que o usuário liste, cadastre, edite, conclua e exclua tarefas, com persistência local e navegação entre a tela de listagem e o formulário de cadastro/edição.

## Tecnologias utilizadas

- **Kotlin**
- **Jetpack Compose** — construção de UI declarativa
- **Room** — persistência local em banco SQLite
- **Coroutines / Flow** — operações assíncronas e observação reativa dos dados
- **ViewModel** — retenção de estado e ponte entre UI e camada de dados
- **Navigation Compose** — navegação entre as telas do app

## Arquitetura

O projeto segue o padrão `UI -> ViewModel -> Repository -> DAO/Room`:

```
Tarefa (Entity)
   -> TarefaDao (Room)
      -> TarefaRepository
         -> TarefaViewModel
            -> ListaTarefasScreen / FormularioTarefaScreen
               -> AppNavigation
                  -> MainActivity
```

### `TarefaRepository`

Responsável por abstrair o acesso aos dados. Expõe `tarefas: Flow<List<Tarefa>>` (obtido diretamente do `TarefaDao.listarTodas()`) e as funções suspensas `inserir`, `atualizar` e `deletar`, que apenas delegam para o DAO. Ela isola a `ViewModel` de detalhes do Room, permitindo trocar a fonte de dados sem alterar as camadas superiores.

### `TarefaViewModel`

Consome o `TarefaRepository` e converte o `Flow` de tarefas em um `StateFlow` (via `stateIn`, com `SharingStarted.WhileSubscribed(5_000)`), que é o estado observado pelas telas Compose. Também expõe as ações `inserir`, `atualizar` e `deletar`, cada uma disparada dentro de `viewModelScope.launch`, mantendo as operações assíncronas fora da UI. A `ViewModel` é criada por uma `Factory` própria (`TarefaViewModel.factory(context)`), que monta o `TarefaDatabase` e o `TarefaRepository` internamente.

### `ListaTarefasScreen`

Observa `viewModel.tarefas` com `collectAsStateWithLifecycle()` e renderiza a lista em uma `LazyColumn`. Cada item (`TarefaItem`) mostra um `Checkbox` para marcar/desmarcar conclusão (disparando `viewModel.atualizar`), um botão de exclusão (`viewModel.deletar`) e é clicável para abrir a edição (`onEditarTarefa`). Um `FloatingActionButton` aciona `onNovaTarefa` para abrir o formulário em modo de cadastro. A tela possui `@Preview`s tanto para a lista com itens quanto para o estado vazio.

### `FormularioTarefaScreen`

Recebe o `tarefaId` da rota de navegação. Quando `tarefaId != 0`, busca a tarefa correspondente na lista observada da `ViewModel` e pré-preenche os campos de título e descrição (modo edição); quando `tarefaId == 0`, os campos iniciam vazios (modo cadastro). Ao salvar, decide entre `viewModel.inserir` (nova tarefa) ou `viewModel.atualizar` (tarefa existente) e retorna à tela anterior via `onVoltar`. Inclui `@Preview`s para os dois modos (novo/edição).

### `AppNavigation`

Configura o `NavHost` com duas rotas:
- `"lista"` — exibe a `ListaTarefasScreen`.
- `"formulario/{tarefaId}"` — exibe a `FormularioTarefaScreen`, extraindo o `tarefaId` dos argumentos da rota.

Para cadastrar, a lista navega para `"formulario/0"`; para editar, navega para `"formulario/{id}"` passando o identificador da tarefa. O `id = 0` é usado como sinalizador de "nova tarefa".

### `MainActivity`

Cria a `TarefaViewModel` usando `viewModel(factory = TarefaViewModel.factory(applicationContext))` e inicia a aplicação chamando `AppNavigation(viewModel = viewModel)` dentro do tema do app, substituindo o conteúdo padrão gerado pelo template do Android Studio.

## Como executar

1. Abra o projeto no Android Studio.
2. Aguarde a sincronização do Gradle.
3. Selecione um emulador ou dispositivo físico com Android configurado.
4. Execute o app (Run 'app').

## Evidências

As evidências de execução (tela de listagem, cadastro, edição, conclusão, exclusão, navegação e build sem erros) estão disponíveis em [`docs/evidencias`](docs/evidencias).

<img width="232" height="525" alt="image" src="https://github.com/user-attachments/assets/56e5081c-6160-4589-a03c-a68d01e81ac0" />
<img width="241" height="531" alt="image" src="https://github.com/user-attachments/assets/b58694f9-f8ac-4b9b-93f1-657b1c409ab0" />
<img width="239" height="523" alt="image" src="https://github.com/user-attachments/assets/7706b240-2243-4f77-be79-04165d704422" />
<img width="227" height="525" alt="image" src="https://github.com/user-attachments/assets/9162cdb2-058c-47af-b531-ce67e5ef0382" />


