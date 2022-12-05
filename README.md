# f290_ddm2_todo_list

## Main Widget

Crie o widget principal e configure o tema do App com as cores laranja e e azul

```dart
void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData.light().copyWith(
        colorScheme: const ColorScheme.light(
          primary: Colors.orange,
          secondary: Colors.blueAccent,
        ),
      ),
      home: const HomePage(),
    );
  }
}
```

## HomePage - Parte I

Crie a página principal do App; no arquivo `pages/home/home_page.dart`.

```dart
class HomePage extends StatefulWidget {
  const HomePage({Key? key}) : super(key: key);

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ToDo List')),
      body: Center(child: Text('Body')),
      ),
      floatingActionButton: FloatingActionButton.extended(
          onPressed: () {
            //TODO: 
          },
          icon: const Icon(Icons.add),
          label: const Text('Adicionar')),
    );
  }
}
```

## Service para Persistência de Dados em Arquivo

Para que possamos persistir os dados da Lista de tarefas iremos criar um Service para facilitar o desenvolvimento.

1. Crie o arquivo `services\data_service.dart`e adicione o código abaixo.

```dart
import 'dart:convert';
import 'dart:io';

import 'package:path_provider/path_provider.dart';

class DataService {
  // Obtem o arquivo de dados independente da plataforma; todos os métodos do serviço serão assícronos, então utilizaremos sempre a dupla async e await quando utilizarmos retornos Futures.
  Future<File> getFile() async {
    // Obtém a localização do local de armazenamento dos dispositivos suportados pelo Flutter, voce não precisa se preocupar com a localização em cada plataforma.
    final directory = await getApplicationDocumentsDirectory();
    return File('${directory.path}/data.json');
  }

  Future<File> saveData(List toDoList) async {
    // Convertendo a lista de Maps em JSON
    var data = jsonEncode(toDoList);
    var file = await getFile();

    return file.writeAsString(data);
  }

  Future<String?> readData() async {
    try {
      File file = await getFile();

      // Convertendo o arquivo de dados JSON em uma String.
      return file.readAsString();
    } catch (e) {
      return null;
    }
  }
}
```

2. Crie a model class `ToDo` no arquivo `model\todo_model.dart`.

```dart
import 'package:intl/intl.dart';

class Todo {
  String conteudo;
  bool concluido = false;
  String dataCriacao = DateFormat('d/M/y HH:mm:ss').format(DateTime.now());
  String dataConclusao = '';

  Todo({required this.conteudo});

  // Construtor nomeado para criar objeto Todo a partir de um Mapa
  Todo.fromJson(Map<String, dynamic> map) :
    this.conteudo = map['conteudo'],
    this.concluido = map['concluido'],
    this.dataCriacao = map['dataCriacao'],
    this.dataConclusao = map['dataConclusao'];

  // COnversao de um objeto em Mapa
  Map<String, dynamic> toJson() {
    return {
      'conteudo': conteudo,
      'concluido': concluido,
      'dataCriacao': dataCriacao,
      'dataConclusao': dataConclusao
    };
  }
}
```

3. Crie uma lista para armazenar o estado das tarefas; um controlador para obter o texto de entrada da tarefa e inicialize o `DataService`.

```dart
class _HomePageState extends State<HomePage> {
  List toDoList = [];

  final _controller = TextEditingController();

  final _service = DataService();

  // Restante do código aqui...
```

4. Faça uma chamada ao serviço no `initState`, logo abaixo da inicialização do service; este método é executado quando a aplicação é carregada e ele irá carregar os dados persistidos no arquivo de dados para realizar o preenchimento de dados da lista de tarefas.

```dart
@override
  void initState() {
    super.initState();

    _service.readData().then((data) {
      setState(() {
        log('JSON: $data');
        if (data != null) {
          toDoList = jsonDecode(data);
        }
      });
    });
  }
```



## HomePage - Parte II

Adicione a dependência `intl` com o comando abaixo no shell

```shell
flutter pub add intl
```

1. Substitua o `Center` por uma `Column`.
2. Inclua na `Column` um `TextField`.

```dart
body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16.0),
            child: TextFormField(
              maxLines: 2,
              // Adição do controlador de texto
              controller: _controller,
              decoration: const InputDecoration(
                labelText: 'To Do',
                filled: true,
                prefixIcon: Icon(
                  Icons.text_fields_rounded,
                ),
              ),
            ),
          ),
```

3. Adicione um `Expanded` para ocupar o restante do espaço da tela com um `ListView`, logo abaixo do `Padding`.

```dart
    Expanded(
    child: ListView.builder(
        itemCount: toDoList.length,
        itemBuilder: (context, position) {        
        return Dismissible(
            key: Key("$position"),
            direction: DismissDirection.startToEnd,
            background: Container(
            color: Colors.red,
            child: const Align(
                alignment: Alignment.centerLeft,
                child: Icon(
                Icons.delete,
                color: Colors.white,
                ),
            ),
            ),
            onDismissed: (direction) {
                //TODO:
            },
            child: CheckboxListTile(
            secondary: CircleAvatar(
                child: todo.concluido
                    ? const Icon(Icons.check)
                    : const Icon(Icons.warning),
            ),
            value: todo.concluido,
            title: Text(
                todo.conteudo,
                style: TextStyle(
                    decoration: todo.concluido
                        ? TextDecoration.lineThrough
                        : TextDecoration.none),
            ),
            subtitle: Column(
                crossAxisAlignment: CrossAxisAlignment.start,
                children: [
                Text('Criada em ${todo.dataCriacao}'),
                Text(
                    todo.dataConclusao.isEmpty
                        ? ''
                        : 'Finalizada em ${todo.dataConclusao}',
                    style: const TextStyle(fontWeight: FontWeight.bold),
                ),
                ],
            ),
            onChanged: (inChecked) {      
                //TODO:          
            },
            ),
        );
        },
    ),
    ),
```

4. Teste o App.

5. Atualize o método `build` e atente-se ao comentários a cada trecho comentado. Neste trecho os `//TODOS: ` foram preenchidos e comentados para facilitar o entendimento.

```dart
@override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ToDo List')),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16.0),
            child: TextFormField(
              maxLines: 2,
              controller: _controller,
              decoration: const InputDecoration(
                labelText: 'To Do',
                filled: true,
                prefixIcon: Icon(
                  Icons.text_fields_rounded,
                ),
              ),
            ),
          ),
          Expanded(
            child: ListView.builder(
              // 5. Limite de elementos persistidos na lista
              itemCount: toDoList.length,
              itemBuilder: (context, position) {
                // 6. Conversao do item atual, com base no index, para um objeto ToDo
                var todo = Todo.fromJson(toDoList[position]);

                // 7. O Dismissible irá adicionar o comportamento do swipe para podermos fazer a deleção do item
                return Dismissible(
                  key: Key("$position"),
                  direction: DismissDirection.startToEnd,
                  background: Container(
                    color: Colors.red,
                    child: const Align(
                      alignment: Alignment.centerLeft,
                      child: Icon(
                        Icons.delete,
                        color: Colors.white,
                      ),
                    ),
                  ),
                  onDismissed: (direction) {
                    setState(() {
                      // 8. Remoção do item na lista de referencia
                      toDoList.removeAt(position);

                      // 9. Chamada de serviço para persistir a remoção
                      _service.saveData(toDoList);
                    });
                  },
                  child: CheckboxListTile(
                    secondary: CircleAvatar(
                      // 10. Operador ternário para mudar o ícone com base em seu status
                      child: todo.concluido
                          ? const Icon(Icons.check)
                          : const Icon(Icons.warning),
                    ),
                    // 11. Conteúdo da tarefa
                    value: todo.concluido,
                    title: Text(
                      todo.conteudo,
                      style: TextStyle(
                          // 12. Operador ternário para alterar o estilo do texto com base em seu estado.
                          decoration: todo.concluido
                              ? TextDecoration.lineThrough
                              : TextDecoration.none),
                    ),
                    subtitle: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text('Criada em ${todo.dataCriacao}'),
                        Text(
                          todo.dataConclusao.isEmpty
                              ? ''
                              : 'Finalizada em ${todo.dataConclusao}',
                          style: const TextStyle(fontWeight: FontWeight.bold),
                        ),
                      ],
                    ),
                    onChanged: (inChecked) {
                      // 13. Mudança de estado da tarefa
                      setState(() {
                        // Atualização de sintaxe de map JSON para objetos
                        todo.concluido = inChecked!;
                        if (inChecked) {
                          // 14. Formatação de data com intl.
                          todo.dataConclusao = DateFormat('d/M/y HH:mm:ss')
                              .format(DateTime.now());
                        } else {
                          todo.dataConclusao = '';
                        }

                        // 15. Persistencia da alteração de estado da tarefa.
                        toDoList[position] = todo.toJson();
                        _service.saveData(toDoList);
                      });
                    },
                  ),
                );
              },
            ),
          ),
        ],
      ),
      floatingActionButton: FloatingActionButton.extended(
          onPressed: () {
            setState(() {
              final conteudo = _controller.text;

              // Verifica se o Widget já foi criado pelo anteriormente
              if (!mounted) return;

              if (conteudo.isEmpty) {
                // Exibição de mensagem de validação de conteúdo obrigatório para preenchimento
                ScaffoldMessenger.of(context)
                  ..removeCurrentSnackBar()
                  ..showSnackBar(const SnackBar(
                    content: Text('Preencha a tarefa.'),
                  ));
                return;
              }

              var newTodo = Todo(conteudo: conteudo);

              // Atualização para conversao de objeto em mapa para persistencia em JSON
              toDoList.add(newTodo.toJson());
              _controller.text = '';

              _service.saveData(toDoList);
            });
          },
          icon: const Icon(Icons.add),
          label: const Text('Adicionar')),
    );
  }
```

6. Confira o código completo da classe `main.dart`.

```dart
import 'dart:convert';
import 'dart:developer';

import 'package:f290_ddm2_todo_list/model/todo_model.dart';
import 'package:f290_ddm2_todo_list/services/data_service.dart';
import 'package:flutter/material.dart';
import 'package:intl/intl.dart';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      theme: ThemeData.light().copyWith(
        colorScheme: const ColorScheme.light(
          primary: Colors.orange,
          secondary: Colors.blueAccent,
        ),
      ),
      home: const HomePage(),
    );
  }
}

class HomePage extends StatefulWidget {
  const HomePage({Key? key}) : super(key: key);

  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  // 1. Lista para persistencia de estado das tarefas
  List toDoList = [];

  // 2. Controller para obter o texto referente às tarefas adicionadas
  final _controller = TextEditingController();

  // 3. Serviço para persisitir a lista de tarefas
  final _service = DataService();

  @override
  void initState() {
    super.initState();

    // 4. Preenchimento da carga de dados ao iniciar o App.
    _service.readData().then((data) {
      setState(() {
        log('JSON: $data');
        if (data != null) {
          toDoList = jsonDecode(data);
        }
      });
    });
  }

  final GlobalKey<FormFieldState<String>> _formKey = GlobalKey();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('ToDo List')),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16.0),
            child: TextFormField(
              maxLines: 2,
              controller: _controller,
              decoration: const InputDecoration(
                labelText: 'To Do',
                filled: true,
                prefixIcon: Icon(
                  Icons.text_fields_rounded,
                ),
              ),
            ),
          ),
          Expanded(
            child: ListView.builder(
              // 5. Limite de elementos persistidos na lista
              itemCount: toDoList.length,
              itemBuilder: (context, position) {
                // 6. Conversao do item atual, com base no index, para um objeto ToDo
                var todo = Todo.fromJson(toDoList[position]);

                // 7. O Dismissible irá adicionar o comportamento do swipe para podermos fazer a deleção do item
                return Dismissible(
                  key: Key("$position"),
                  direction: DismissDirection.startToEnd,
                  background: Container(
                    color: Colors.red,
                    child: const Align(
                      alignment: Alignment.centerLeft,
                      child: Icon(
                        Icons.delete,
                        color: Colors.white,
                      ),
                    ),
                  ),
                  onDismissed: (direction) {
                    setState(() {
                      // 8. Remoção do item na lista de referencia
                      toDoList.removeAt(position);

                      // 9. Chamada de serviço para persistir a remoção
                      _service.saveData(toDoList);
                    });
                  },
                  child: CheckboxListTile(
                    secondary: CircleAvatar(
                      // 10. Operador ternário para mudar o ícone com base em seu status
                      child: todo.concluido
                          ? const Icon(Icons.check)
                          : const Icon(Icons.warning),
                    ),
                    // 11. Conteúdo da tarefa
                    value: todo.concluido,
                    title: Text(
                      todo.conteudo,
                      style: TextStyle(
                          // 12. Operador ternário para alterar o estilo do texto com base em seu estado.
                          decoration: todo.concluido
                              ? TextDecoration.lineThrough
                              : TextDecoration.none),
                    ),
                    subtitle: Column(
                      crossAxisAlignment: CrossAxisAlignment.start,
                      children: [
                        Text('Criada em ${todo.dataCriacao}'),
                        Text(
                          todo.dataConclusao.isEmpty
                              ? ''
                              : 'Finalizada em ${todo.dataConclusao}',
                          style: const TextStyle(fontWeight: FontWeight.bold),
                        ),
                      ],
                    ),
                    onChanged: (inChecked) {
                      // 13. Mudança de estado da tarefa
                      setState(() {
                        // Atualização de sintaxe de map JSON para objetos
                        todo.concluido = inChecked!;
                        if (inChecked) {
                          // 14. Formatação de data com intl.
                          todo.dataConclusao = DateFormat('d/M/y HH:mm:ss')
                              .format(DateTime.now());
                        } else {
                          todo.dataConclusao = '';
                        }

                        // 15. Persistencia da alteração de estado da tarefa.
                        toDoList[position] = todo.toJson();
                        _service.saveData(toDoList);
                      });
                    },
                  ),
                );
              },
            ),
          ),
        ],
      ),
      floatingActionButton: FloatingActionButton.extended(
          onPressed: () {
            setState(() {
              final conteudo = _controller.text;

              // Verifica se o Widget já foi criado pelo anteriormente
              if (!mounted) return;

              if (conteudo.isEmpty) {
                // Exibição de mensagem de validação de conteúdo obrigatório para preenchimento
                ScaffoldMessenger.of(context)
                  ..removeCurrentSnackBar()
                  ..showSnackBar(const SnackBar(
                    content: Text('Preencha a tarefa.'),
                  ));
                return;
              }

              var newTodo = Todo(conteudo: conteudo);

              // Atualização para conversao de objeto em mapa para persistencia em JSON
              toDoList.add(newTodo.toJson());
              _controller.text = '';

              _service.saveData(toDoList);
            });
          },
          icon: const Icon(Icons.add),
          label: const Text('Adicionar')),
    );
  }
}
```
