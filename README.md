# lab9
import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';

void main() {
  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'CRUD App',
      home: const HomePage(),
    );
  }
}

class User {
  final int id;
  final String name;
  final String email;

  User({required this.id, required this.name, required this.email});

  factory User.fromJson(Map<String, dynamic> json) {
    return User(id: json['id'], name: json['name'], email: json['email']);
  }
}

class Post {
  String title;
  String body;

  Post({required this.title, required this.body});
}

class HomePage extends StatefulWidget {
  const HomePage({super.key});
  @override
  State<HomePage> createState() => _HomePageState();
}

class _HomePageState extends State<HomePage> {
  List<User> users = [];
  List<Post> posts = [];

  Future<void> fetchUsers() async {
    final response = await http.get(Uri.parse('https://jsonplaceholder.typicode.com/users'));
    if (response.statusCode == 200) {
      final List<dynamic> data = json.decode(response.body);
      setState(() {
        users = data.map((json) => User.fromJson(json)).toList();
      });
    }
  }

  void addPost(String title, String body) {
    setState(() {
      posts.add(Post(title: title, body: body));
    });
  }

  void editPost(int index, String newTitle, String newBody) {
    setState(() {
      posts[index].title = newTitle;
      posts[index].body = newBody;
    });
  }

  void deletePost(int index) {
    setState(() {
      posts.removeAt(index);
    });
  }

  void showPostForm({int? index}) {
    final titleController = TextEditingController();
    final bodyController = TextEditingController();

    if (index != null) {
      titleController.text = posts[index].title;
      bodyController.text = posts[index].body;
    }

    showDialog(
      context: context,
      builder: (_) => AlertDialog(
        title: Text(index == null ? 'Add Post' : 'Edit Post'),
        content: Column(
          mainAxisSize: MainAxisSize.min,
          children: [
            TextField(controller: titleController, decoration: const InputDecoration(labelText: 'Title')),
            TextField(controller: bodyController, decoration: const InputDecoration(labelText: 'Body')),
          ],
        ),
        actions: [
          TextButton(
            onPressed: () {
              final title = titleController.text.trim();
              final body = bodyController.text.trim();
              if (title.isNotEmpty && body.isNotEmpty) {
                if (index == null) {
                  addPost(title, body);
                } else {
                  editPost(index, title, body);
                }
                Navigator.pop(context);
              }
            },
            child: const Text('Save'),
          ),
        ],
      ),
    );
  }

  @override
  void initState() {
    super.initState();
    fetchUsers();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text('Users & Posts')),
      body: SingleChildScrollView(
        child: Column(
          children: [
            const SizedBox(height: 10),
            const Text('Users (From API)', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
            ListView.builder(
              shrinkWrap: true,
              physics: const NeverScrollableScrollPhysics(),
              itemCount: users.length,
              itemBuilder: (context, index) {
                final user = users[index];
                return ListTile(title: Text(user.name), subtitle: Text(user.email));
              },
            ),
            const Divider(),
            const Text('Posts (Local CRUD)', style: TextStyle(fontSize: 18, fontWeight: FontWeight.bold)),
            ListView.builder(
              shrinkWrap: true,
              physics: const NeverScrollableScrollPhysics(),
              itemCount: posts.length,
              itemBuilder: (context, index) {
                final post = posts[index];
                return Card(
                  margin: const EdgeInsets.symmetric(horizontal: 10, vertical: 5),
                  child: ListTile(
                    title: Text(post.title),
                    subtitle: Text(post.body),
                    trailing: Row(
                      mainAxisSize: MainAxisSize.min,
                      children: [
                        IconButton(icon: const Icon(Icons.edit), onPressed: () => showPostForm(index: index)),
                        IconButton(icon: const Icon(Icons.delete), onPressed: () => deletePost(index)),
                      ],
                    ),
                  ),
                );
              },
            ),
          ],
        ),
      ),
      floatingActionButton: FloatingActionButton(
        onPressed: () => showPostForm(),
        child: const Icon(Icons.add),
      ),
    );
  }
}
