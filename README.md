# lesson2

import 'package:flutter/material.dart';
import 'package:http/http.dart' as http;
import 'dart:convert';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: CoffeeOrderScreen(),
    );
  }
}

class CoffeeOrderScreen extends StatefulWidget {
  @override
  _CoffeeOrderScreenState createState() => _CoffeeOrderScreenState();
}

class _CoffeeOrderScreenState extends State<CoffeeOrderScreen> {
  String selectedCoffee = "latte";
  bool addMilk = false;
  int sugar = 0;

  Future<void> submitOrder() async {
    final response = await http.post(
      Uri.parse('http://192.168.0.132:5000/order'),  // Өз серверіңіздегі IP-ді орнатыңыз
      headers: {'Content-Type': 'application/json'},
      body: json.encode({
        'coffee': selectedCoffee,
        'milk': addMilk,
        'sugar': sugar,
      }),
    );

    if (response.statusCode == 200) {
      final result = json.decode(response.body);
      showDialog(
        context: context,
        builder: (context) => AlertDialog(
          title: Text('Тапсырыс нәтижесі'),
          content: Text('Бағасы: ${result['total_price']} тг'),
          actions: [
            TextButton(
              onPressed: () => Navigator.pop(context),
              child: Text('OK'),
            ),
          ],
        ),
      );
    } else {
      // Қате туралы хабарлама көрсету
      showDialog(
        context: context,
        builder: (context) => AlertDialog(
          title: Text('Қате'),
          content: Text('Сіздің тапсырысыңыз қабылданбады.'),
          actions: [
            TextButton(
              onPressed: () => Navigator.pop(context),
              child: Text('OK'),
            ),
          ],
        ),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: Text("☕ Кофе Тапсырыс")),
      body: Padding(
        padding: EdgeInsets.all(16.0),
        child: Column(
          children: [
            DropdownButton<String>(
              value: selectedCoffee,
              items: ["espresso", "americano", "latte", "cappuccino", "mocha"]
                  .map((coffee) => DropdownMenuItem(
                value: coffee,
                child: Text(coffee.toUpperCase()),
              ))
                  .toList(),
              onChanged: (value) {
                setState(() => selectedCoffee = value!);
              },
            ),
            SwitchListTile(
              title: Text("🥛 Сүт қосу (+200 тг)"),
              value: addMilk,
              onChanged: (value) {
                setState(() => addMilk = value);
              },
            ),
            Row(
              mainAxisAlignment: MainAxisAlignment.spaceBetween,
              children: [
                Text("🍬 Қант: $sugar шай қасық"),
                Slider(
                  min: 0,
                  max: 3,
                  divisions: 3,
                  value: sugar.toDouble(),
                  onChanged: (value) {
                    setState(() => sugar = value.toInt());
                  },
                ),
              ],
            ),
            SizedBox(height: 20),
            ElevatedButton(
              onPressed: submitOrder,
              child: Text("✅ Тапсырыс беру"),
            ),
          ],
        ),
      ),
    );
  }
}

# Flutter-and-Pyton-coffee
